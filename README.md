# Agent 微信公众号阅读

一个面向 Agent 工作流的微信公众号文章读取 Skill。

当消息中出现一个或多个 `https://mp.weixin.qq.com/...` 链接时，支持 Skill 语义路由的 Agent 可以自动选择它，提取公开公众号文章的标题、作者、发布时间、正文结构与懒加载图片，并保存为 Markdown 和结构化元数据。

它不要求固定提示词，也不依赖微信 Cookie、账号、API Key、浏览器登录态或第三方 Reader 服务。

## 为什么做这个项目

公众号文章在人手打开时通常没有问题，但交给 Agent 后，经常只返回验证页面、网页外壳或混杂大量界面文字的内容。手动复制、清理再粘贴可以解决一次，却很难成为稳定的工作流。

这个项目只解决一个问题：让公开公众号文章以干净、可追踪、便于继续处理的形式进入 Agent 上下文。

## 能力

- 自动识别单个或批量 `mp.weixin.qq.com` 链接，无需固定口令
- 优先使用普通浏览器请求，必要时仅进行一次微信 WebView 请求重试
- 定位 `#js_content`，过滤页面按钮、推荐阅读和其他界面噪声
- 提取标题、作者、发布时间、段落、标题、列表、引用、链接和懒加载图片
- 分别保存 Markdown、元数据、原始 HTML 和请求追踪
- 严格校验域名、HTTPS、端口和跨域重定向
- 限制超时时间、响应大小和重试次数
- 单篇失败不阻断同一批次中的其他链接

## 安装

### 使用 Skills CLI

```bash
npx -y skills add Githun1314/agent-wechat-reader --skill wechat-article-reader
```

### 手动安装到 Codex

```bash
git clone https://github.com/Githun1314/agent-wechat-reader.git
mkdir -p ~/.codex/skills
cp -R agent-wechat-reader/skills/wechat-article-reader ~/.codex/skills/
```

安装后重新开始一个 Agent 任务，使 Skill 元数据进入新的上下文。

## 使用

不需要记住固定提示词。直接在任务中发送公众号链接即可：

```text
https://mp.weixin.qq.com/s/...
```

也可以一次发送多篇文章，并附带实际任务：

```text
请比较下面三篇文章的核心观点：

https://mp.weixin.qq.com/s/...
https://mp.weixin.qq.com/s/...
https://mp.weixin.qq.com/s/...
```

如果只发送链接，没有补充要求，Skill 默认让 Agent 返回每篇文章的标题、作者、读取状态和简要摘要。

Skill 的自动选择属于 Agent 的语义路由，不是浏览器层面的域名拦截器。具体产品需要支持 `SKILL.md`，或允许将脚本封装为自定义工具。

## 直接运行脚本

脚本仅依赖 Python 3.9 及以上版本的标准库：

```bash
python3 skills/wechat-article-reader/scripts/read_wechat_article.py \
  "https://mp.weixin.qq.com/s/..." \
  --quiet
```

成功后会打印 `article.md` 和 `article.json` 的本地路径。

## 输出

```text
<temp-dir>/
├── article.md    # 清理后的正文
├── article.json  # 标题、作者、时间、字数、图片数量和文件路径
├── raw.html      # 原始页面，用于排查结构变化
└── trace.json    # 请求档位、耗时、响应大小和验证页状态
```

## 接入 Coze 或其他 Agent 产品

- 支持 `SKILL.md` 的 Agent：直接安装 `skills/wechat-article-reader/`。
- 支持自定义工具或工作流节点的平台：将 `read_wechat_article.py` 封装为只接收 URL 的工具，读取 `article.json` 和 `article.md` 作为返回结果。
- 不支持本地脚本的平台：可以将脚本部署为受控的内部 HTTP 工具，但应继续保留当前域名白名单、大小限制和无凭据边界。

不要把微信 Cookie、登录态或账号凭据加入这个工具。

## 适用边界

只支持公开、免登录的 `https://mp.weixin.qq.com` 文章。

以下内容不会尝试绕过：

- 登录可见或付费文章
- 已删除或不可访问的文章
- 微信安全验证、频率限制或风控页面
- 跳转到其他域名的链接

失败时，建议改用用户提供的正文、截图、PDF 或导出的网页文件。

## 验证

运行不访问网络的确定性自测：

```bash
python3 skills/wechat-article-reader/scripts/read_wechat_article.py --self-test
```

项目的 GitHub Actions 会在 Python 3.9、3.11 和 3.13 上执行语法检查与自测。

## 参考与致谢

这个项目参考并重新组合了两类思路：

- [Claude 和 Codex，终于能读公众号了](https://blog.csdn.net/Niceyun/article/details/162941348)：微信公众号页面请求与正文定位思路
- [code-yeongyu/web-fetch](https://github.com/code-yeongyu/web-fetch)：临时文件、结构化元数据、追踪记录和按需读取设计

当前实现针对公众号文章重新编写，并增加了严格域名校验、跨域重定向限制、正文结构解析、批量链接工作流和明确的失败边界。

## License

[MIT](LICENSE)
