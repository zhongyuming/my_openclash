# OpenClash + Mihomo 自定义规则

这是一套适合长期维护的 OpenClash + Mihomo 配置模板。个人域名分别保存在 `rules/*.list` 中；配置订阅模式会在每次生成配置时拉取最新规则，完整 YAML 模式则由 Mihomo `rule-providers` 每 86400 秒检查更新。以后通常只需要修改规则文件，无需反复编辑整份配置。

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

当前仍为空的直连和拒绝列表保留了 `DOMAIN-SUFFIX,placeholder.invalid`。这是无害的保留域名占位规则，用来避免某些旧版核心拒绝加载完全空白的规则集；添加真实规则后可以保留，也可以删除。

规则按“拒绝 > 直连 > AI > 代理”的顺序匹配。同一个域名不要重复放入多个文件；如果重复，排在前面的规则生效。所有自定义规则均放在 `ChinaDomain`、`GEOIP` 和 `MATCH` 之前，防止被大范围规则提前命中。

## 公共境外数据库

配置采用“默认直连、命中规则才分流”的黑名单模式：

1. `rules/*.list`：你可以直接维护的高优先级规则；其中代理和 AI 列表预置了一组常用核心域名。
2. Mihomo GEOSITE：使用 `gfw`、`category-scholar-!cn`、`category-social-media-!cn`、`category-browser-!cn`、`category-anticensorship` 和 `category-vpnservices` 等精确分类。
3. 中国域名和中国 IP 明确直连，最终 `MATCH/FINAL` 也是 `DIRECT`。未列入任何规则的境外网站同样直连；若需要代理，将域名加入 `my-proxy.list`。

GEOSITE 分类主要来自 [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)，Mihomo 数据由 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 提供。没有直接引入 blackmatrix7 的完整 Global 列表，因为它规模较大，而且会让大量未明确指定的网站自动走代理；在 R2S 上使用精确分类更符合按需分流模式。

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
7. 首次启动后检查“代理集”和“规则集”页面，确认机场订阅和五个远程规则提供器都已成功加载。

OpenClash 可能接管或改写端口、DNS、控制器地址及 provider 本地路径；最终生效内容应以 OpenClash 运行时配置为准。

## 更新规则

配置订阅模式（推荐）的更新流程：

1. 修改相应的 `rules/*.list`。
2. 提交并推送到 GitHub 的 `main` 分支。
3. 等待 OpenClash 的下一次配置订阅更新，或在“配置订阅”页面点击“更新配置”。
4. 确认更新后的配置仍是当前启用配置；必要时点击“切换”或重启 OpenClash。

`subconverter.ini` 已启用 `update_ruleset_on_request=true`，每次转换请求都会重新读取远程规则，避免使用旧的 subconverter 规则缓存。配置订阅模式会把个人规则展开到生成配置的 `rules` 数组中，因此不应只刷新 Dashboard 的 Rule Provider。

如果使用本地导入的完整 `mihomo.yaml`，四个 Rule Provider 会按 86400 秒周期更新，也可以在 Dashboard 的 Rules/规则提供器页面手动刷新。

## 查看规则是否命中

- 在 OpenClash Dashboard 的 Connections/连接页面查看目标域名、命中的规则和最终策略组。
- 把 OpenClash 日志级别临时调为 `debug`，访问目标网站后搜索域名、`RULE-SET` 名称或策略组名称。
- 配置订阅模式可通过 SSH 搜索源配置和运行时配置：`grep -n "example.com" /etc/openclash/config/my.yaml /etc/openclash/my.yaml`。能搜到说明转换后的配置已经包含该规则。
- 完整 YAML 模式可在 Rules/规则页面确认 `MyProxy`、`MyDirect`、`MyAI`、`MyReject` 已加载且规则数量符合预期。
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
8. `ChinaDomain` 来自 MetaCubeX。若自定义规则正常而它加载失败，重点检查 MetaCubeX URL、MRS 支持和核心版本。

## 隐私与公开仓库提醒

GitHub 公开仓库中的域名列表任何人都可以查看，也可能被搜索引擎、镜像或历史提交长期保存。不要提交私人内网域名、设备名称、公司内部地址、邮箱、Token、密码或任何不希望公开的信息。

一旦订阅地址或 Token 被提交，即使随后删除文件，仍可能留在 Git 历史中。发现泄漏时应立即在服务商处重置订阅链接或 Token，而不是只删除最新提交。

## 依赖与兼容性说明

- 四份个人规则完全由本仓库维护。
- 大范围规则只保留一个 MetaCubeX MRS provider：`ChinaDomain`（中国域名）。不加载 `ProxyLite/geolocation-!cn`，避免所有境外域名自动走代理。
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
