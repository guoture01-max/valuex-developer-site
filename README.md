# ValueX Developer Website

GUOTURE 的 ValueX 公开开发者网站。英文页面便于 HSBC Developer Portal 审阅。站点采用静态 HTML/CSS、系统字体和现有 ValueX 图标，无需构建或安装依赖。

## 本地预览

在此目录运行：

```sh
python3 -m http.server 8765 --bind 127.0.0.1 --directory docs
```

打开 `http://127.0.0.1:8765/` 可查看首页。直接打开各 HTML 文件也可以预览内容。这个本机地址仅用于检查网页。

## GitHub Pages

发布需要仓库所有者明确批准。建议在所有者控制的 GitHub 账号中建独立仓库 `valuex-developer-site`。若使用 GitHub Free，网站仓库需公开。

1. 确认公开联系渠道及图标发布权限。本站使用网站仓库的公开 Issues。迁移到其他仓库时，同步更新 `docs/developer.html` 的联系链接。
2. 仅上传本目录中的 `README.md` 和 `docs/`，保留 `docs/.nojekyll`。公开仓库让所有访客看到这些源文件、分支提交记录和提交者署名。请用 GitHub 支持的 noreply 邮箱提交，或从 GitHub 网页提交并保持邮箱隐私。现有 ValueX 工程和数据位于站点目录范围之外。
3. 在仓库 **Settings → Pages** 中，设置 **Deploy from a branch → main → /docs**，然后开启 **Enforce HTTPS**。以 Pages 设置显示的实际站点地址为准，无需自定义域名。
4. 在未登录浏览器检查首页、Privacy Policy、Terms of Service、Developer Information 和 PNG。确认 HTTPS 证书有效、HTTP 会升级到 HTTPS，所有页面和资源均成功返回，导航与联系渠道有效。

[GitHub Pages 发布源说明](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) · [GitHub Pages HTTPS 说明](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)

## HSBC Sandbox 软件声明

选择 Hong Kong SAR 的 **Account Information — HK — Personal**。以 Portal 实际界面为准。公开指南中的步骤编号可能与当前界面不同。

| 字段 | 填写依据 |
| --- | --- |
| Client URL | 已发布、已验证的 HTTPS 站点根地址，保留末尾斜杠 |
| Policy URI | 同一站点下的 `privacy.html` |
| Terms of Service URI | 同一站点下的 `terms.html` |
| Logo URI | 直接访问同一站点下的 `assets/valuex-logo.png` |
| Redirect URLs | 独立 HTTPS 回调部署后，填写由开发者控制的准确 URI，再通过 Sandbox 注册与联调验证；当前待验证 |
| Software roles | 选择 `AISP`，仅用于 Sandbox 的授权账户信息读取 |

若界面还要求 **Application name**，填 `ValueX`。**Application description** 可填 `Private financial analysis app testing read-only balances and transactions in the HSBC Hong Kong Personal Open Banking Sandbox.` **Version** 填用于测试的真实应用版本。

## HSBC OAuth 回调设计

HSBC 的香港个人 Sandbox 指南要求使用登记的回调，并在换取令牌时使用授权请求中的同一个 `redirect_uri`。注册值、授权请求（包括已签名的 Request JWT）和令牌请求使用同一份配置。检查协议、主机、端口、路径、大小写与末尾斜杠。Portal 有多个回调时每行填写一个。HTTPS 页面返回 200 只证明可访问，回调兼容性须通过项目注册及 Sandbox 授权验证。

HSBC 的授权成功示例把 `state`、`id_token` 和 `code` 放在 URL 片段，同一指南的 Postman 流程则把 `code` 称为查询参数。片段留在浏览器内。若实际响应使用查询参数，托管方会在页面脚本运行前收到这些参数。本站只提供公开页面。HSBC 的令牌交换须按注册的方法进行客户端认证，而且需使用测试传输证书。静态网页无法提供安全的私钥保管和服务端交换。

### 独立 HTTPS 后端

1. 在开发者控制的独立 HTTPS 服务上接收 Sandbox 回调。固定一个回调路径并使用受信任证书。确认 Portal 接受该 URI，以及该客户端使用的实际响应模式，然后登记回调。可以选用已有的受控域名或经 Portal 验证、由开发者控制的托管子域名。回调和公开站点地址独立配置。
2. 由可信进程创建短期测试会话、随机一次性的 `state` 和 `nonce`，并签署 HSBC Request JWT。回调与发起授权的会话关联，拒绝过期、重放和不匹配的响应。按 HSBC/OIDC 规则验证 ID token，包括签名、发行者、受众、到期时间、nonce 和适用的 `c_hash`。PKCE 与响应模式按该 Sandbox 客户端实际支持的配置确认。若实际使用片段，由独立后端提供自包含的同源回调页，在内存读取参数、清理地址栏，然后用同源 POST 交给后端处理。
3. 按 Portal 为该项目登记的客户端认证方式在可信进程中换取令牌，并使用 HSBC Sandbox 传输证书和私钥。产品指南涵盖 `tls_client_auth` 和 `private_key_jwt`，香港 DCR 章节列出 `private_key_jwt`，以项目配置为准。测试密钥只在本地 Keychain 或后端专用密钥存储中使用。ValueX 通过经身份验证的测试会话获取必要的只读测试数据。AIS 使用 `openid accounts`，同意对象从 `ReadAccountBalance` 和 `ReadAccountTransaction` 开始，仅在读取流程所需时请求账户状态权限。
4. 在回调、代理、观测、错误处理各层禁用敏感参数记录或在写入前删除。响应设 `Cache-Control: no-store` 与 `Referrer-Policy: no-referrer`，处理后导航到无参数的结果页。页面只显示通用成功或失败状态。避免在浏览器存储、公开源码或支持渠道中保存授权材料。
5. 联调先使用模拟回调验证过期、状态不匹配和重放等场景。经授权后使用 HSBC 测试档案确认 Portal 接受回调、Consent 可完成、令牌交换成功，并验证只读取测试余额及交易流水。确认日志、缓存与结果页均无敏感参数后，记录最终可填写的 URI。

### 本机回调备选

若 HSBC 明确允许该 Sandbox 客户端使用 HTTP loopback，可由 macOS 上的独立测试进程仅绑定 `127.0.0.1` 或 `::1`，在发起浏览器授权时开启监听，收到结果后关闭，并由可信本机进程执行同样的校验和令牌交换。OAuth 原生应用标准允许使用 loopback，但 HSBC 公开指南未确认该项目可使用 loopback、动态端口或自定义 scheme。Portal 接受并完成联调前，任何此类地址都只是设计备选。

[HSBC 香港个人 Account Information Sandbox 指南](https://develop.hsbc.com/ob-api-documentation/account-information-hk-personal) · [HSBC 动态客户端注册说明](https://develop.hsbc.com/ob-api-documentation/dynamic-client-registration-dcr) · [RFC 9700](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1) · [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252.html#section-7.3)
