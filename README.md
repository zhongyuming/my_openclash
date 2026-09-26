# OpenClash + Mihomo 自定义规则

这是一套适合长期维护的 OpenClash + Mihomo 配置模板。个人域名分别保存在 `rules/*.list` 中；公开到 GitHub 后，Mihomo 会通过 `rule-providers` 每 86400 秒（24 小时）检查一次更新。以后通常只需要修改规则文件，无需反复编辑整份配置。

## 目录结构

```text
custom-openclash-rules/
├─ README.md
├─ config/
│  ├─ mihomo.yaml
│  ├─ base.yaml
│  └─ subconverter.ini
└─ rules/
   ├─ my-proxy.list
   ├─ my-direct.list
   ├─ my-ai.list
   └─ my-reject.list
```

文件用途：

- `config/mihomo.yaml`：OpenClash/Mihomo 主配置模板，包含订阅提供器、策略组、远程规则提供器和规则顺序。
- `config/subconverter.ini`：OpenClash“配置订阅”页面使用的远程自定义转换模板。
- `config/base.yaml`：subconverter 生成 Mihomo 配置时使用的基础配置。
- `rules/my-proxy.list`：交给“我的代理网站”策略组。
- `rules/my-direct.list`：交给“我的直连网站”策略组。
- `rules/my-ai.list`：交给“AI”策略组。
- `rules/my-reject.list`：交给“我的拒绝网站”策略组，默认拒绝连接。

## 首次使用前替换占位符

在 `config/mihomo.yaml` 中替换：

```text
YOUR_GITHUB_USERNAME   -> 你的 GitHub 用户名
YOUR_GITHUB_REPOSITORY -> 你的仓库名
```

例如仓库地址是 `https://github.com/alice/openclash-rules`，对应规则地址应为：

```text
https://raw.githubusercontent.com/alice/openclash-rules/main/rules/my-proxy.list
```

`YOUR_SUBSCRIPTION_URL` 只应在路由器上的本地配置副本中替换。不要把真实机场订阅地址、Token、密码、节点或其他凭据提交到 GitHub。建议先把模板上传，再下载/复制到路由器，最后只在路由器上填写订阅地址。

订阅地址必须返回 Clash/Mihomo 可识别的代理节点提供器内容。如果机场只提供“完整配置订阅”而不是节点订阅，需要先使用机场提供的 Clash 订阅地址或可信的本地转换方式。

## 添加自定义规则

四个规则文件均使用 Mihomo `classical + text` 格式，每行一条规则，不需要 `payload:`：

```text
DOMAIN-SUFFIX,example.com
DOMAIN,login.example.com
DOMAIN-KEYWORD,example
```

按用途编辑对应文件：

- 需要代理：`rules/my-proxy.list`
- 需要直连：`rules/my-direct.list`
- AI 服务：`rules/my-ai.list`
- 需要拒绝：`rules/my-reject.list`

文件中的 `DOMAIN-SUFFIX,placeholder.invalid` 是无害的保留域名占位规则，用来避免某些旧版核心拒绝加载完全空白的规则集。添加真实规则后可以保留，也可以删除。

规则按“拒绝 > 直连 > AI > 代理”的顺序匹配。同一个域名不要重复放入多个文件；如果重复，排在前面的规则生效。所有自定义规则均放在 `ChinaDomain`、`ProxyLite`、`GEOIP` 和 `MATCH` 之前，防止被大范围规则提前命中。

## 导入 OpenClash

### 方式一：配置订阅自动更新（推荐）

在 OpenClash 的“配置订阅”页面添加或编辑订阅：

1. “订阅地址”填写机场订阅地址；不要把机场地址写入 GitHub。
2. “订阅转换模板”选择“自定义模板”。
3. 自定义模板地址填写：

   ```text
   https://raw.githubusercontent.com/zhongyuming/my_openclash/main/config/subconverter.ini
   ```

4. 配置文件名可填写 `my`，目标格式选择 Clash/Mihomo（Meta）。
5. 建议把自动更新设为“每天”，选择低使用时段。
6. 保存后点击“更新配置”，完成后切换到生成的配置并启动 OpenClash。

每次配置订阅更新时，subconverter 会读取最新的 `subconverter.ini`、`base.yaml` 和四份 `rules/*.list`，重新生成完整配置。因此提交 GitHub 后，最迟会在下一次 OpenClash 配置订阅更新时生效；也可以点击“更新配置”立即拉取。

> 配置订阅模式会把个人规则展开到生成配置的 `rules` 中，不依赖运行时 `rule-providers` 更新。自动更新时间由 OpenClash“配置订阅”页面控制。

### 方式二：本地导入完整 Mihomo YAML

1. 先把本仓库设为公开仓库，并确认四个 `raw.githubusercontent.com` 地址能直接打开纯文本内容。
2. 下载 `config/mihomo.yaml`，替换 GitHub 用户名、仓库名。
3. 在这份本地副本中替换 `YOUR_SUBSCRIPTION_URL`，不要把替换后的文件重新提交到公开仓库。
4. 在 OpenClash 的“配置文件订阅/配置文件管理”页面上传本地 YAML；不同 OpenClash 版本菜单名称可能略有差异。
5. 也可以通过 SSH 把文件放到 `/etc/openclash/config/mihomo.yaml`，然后在 OpenClash 中选择并启用该配置。
6. 运行模式选择规则模式，核心选择 Mihomo/Meta 内核，保存并启动 OpenClash。
7. 首次启动后检查“代理集”和“规则集”页面，确认机场订阅和六个远程规则提供器都已成功加载。

OpenClash 可能接管或改写端口、DNS、控制器地址及 provider 本地路径；最终生效内容应以 OpenClash 运行时配置为准。

## 更新规则

正常自动更新流程：

1. 修改相应的 `rules/*.list`。
2. 提交并推送到 GitHub 的 `main` 分支。
3. 等待 Mihomo 的 86400 秒更新周期，或在 OpenClash 面板中手动刷新规则提供器。

手动更新通常可在 OpenClash Dashboard 的 Rules/规则提供器页面点击对应 provider 的刷新按钮。若当前版本没有刷新入口，可重启 OpenClash触发加载。仍使用旧缓存时，先备份配置，再从 OpenClash 的规则集文件管理页面删除对应缓存文件并重启；不要误删整个 `/etc/openclash` 目录。

## 查看规则是否命中

- 在 OpenClash Dashboard 的 Connections/连接页面查看目标域名、命中的规则和最终策略组。
- 把 OpenClash 日志级别临时调为 `debug`，访问目标网站后搜索域名、`RULE-SET` 名称或策略组名称。
- 在 Rules/规则页面确认 `MyProxy`、`MyDirect`、`MyAI`、`MyReject` 已加载且规则数量符合预期。
- 测试结束后把日志级别改回 `info`，减少 R2S 的日志和存储开销。

## 外部规则下载失败排查

按以下顺序检查：

1. 仓库是否公开，默认分支是否确实叫 `main`。
2. `YOUR_GITHUB_USERNAME` 和 `YOUR_GITHUB_REPOSITORY` 是否已完全替换，大小写及路径是否正确。
3. 在浏览器中直接打开 raw URL，确认 HTTP 状态为 200，内容是纯文本而不是 GitHub 登录页或 404 页面。
4. 路由器的系统时间、DNS、默认网关和 HTTPS 证书是否正常。
5. OpenClash 日志中是否出现超时、TLS、路径权限、`format: text` 或规则解析错误。
6. Mihomo 核心是否足够新；`classical + text` 必须显式配置 `format: text`。
7. 如果只有 `raw.githubusercontent.com` 无法访问，可检查 OpenClash 的 GitHub 地址修改/CDN 设置；修改后再次刷新 provider。
8. `ChinaDomain` 和 `ProxyLite` 来自 MetaCubeX。若自定义规则正常而这两个 provider 失败，重点检查 MetaCubeX URL、MRS 支持和核心版本。

## 隐私与公开仓库提醒

GitHub 公开仓库中的域名列表任何人都可以查看，也可能被搜索引擎、镜像或历史提交长期保存。不要提交私人内网域名、设备名称、公司内部地址、邮箱、Token、密码或任何不希望公开的信息。

一旦订阅地址或 Token 被提交，即使随后删除文件，仍可能留在 Git 历史中。发现泄漏时应立即在服务商处重置订阅链接或 Token，而不是只删除最新提交。

## 依赖与兼容性说明

- 四份个人规则完全由本仓库维护。
- 大范围规则只保留两个 MetaCubeX MRS provider：`ChinaDomain`（中国域名）和 `ProxyLite`（非中国域名），以减少第三方远程依赖。
- Microsoft、Netflix、Telegram、AI 和游戏使用 Mihomo 内置 `GEOSITE` 数据；`GEOIP` 只用于 `private` 和 `CN`。
- `category-ai-!cn`、`category-games@cn` 等 GEOSITE 标签依赖当前 Mihomo geodata。若所用 OpenClash/Mihomo 版本的数据集不包含某标签，需升级 geodata/核心，或删除对应规则后改用自己的 classical 规则。
- 部分机场节点名称不含国家/地区关键词，会被地区正则漏掉；这属于节点命名兼容性问题，可按机场实际节点名调整 `filter`。
- 如果某地区没有匹配节点，相关自动/手动组可能为空。可修改正则，或从上层策略组中暂时移除该地区组。
- DNS、TProxy、redir-port 是否由 OpenClash 接管取决于插件版本及覆写设置。R2S 上应以 OpenClash 生成的运行时配置和启动日志为准。

## 上传到 GitHub

配置中的 raw URL 假定 `rules/` 位于 GitHub 仓库根目录。当前项目已经按这一结构组织；如果复制到其他仓库，也不要把整个项目再嵌套一层。

目标仓库根目录应当看到：

```text
README.md
config/mihomo.yaml
config/base.yaml
config/subconverter.ini
rules/my-proxy.list
rules/my-direct.list
rules/my-ai.list
rules/my-reject.list
```

确认文件中仍然没有真实订阅地址或凭据后，在目标仓库根目录执行：

```bash
git status
git add README.md config rules
git diff --cached
git commit -m "Add maintainable OpenClash rules"
git push origin main
```

如果选择把本目录原样放进另一个仓库的 `custom-openclash-rules/` 子目录，四个 provider URL 必须相应改成 `.../main/custom-openclash-rules/rules/文件名`；否则 GitHub 会返回 404。为了保持模板中约定的统一 URL，推荐把本目录内容作为仓库根目录发布。

执行 `git diff --cached` 时再次搜索 `YOUR_SUBSCRIPTION_URL` 的替换位置，确认没有意外提交真实链接。
