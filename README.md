#  👉ChatGPT-Proxy

分享一个ChatGPT代理接口的获取方式，首先说明一下原因：

第一个原因：国内一直都访问不了OpenAI的官方接口，最常见解决方案是使用一台服务器来进行反向代理，但这样又徒增了一些成本，所以我们推荐用Cloudfalre Worker 代理的方式来解决 OpenAI 的 API 无法访问的问题。

另一个原因：近期OpenAI加大了风控，有大量APIKey或账号被封禁。因为Cloudflare也算云服务商，而从云服务商请求 API 是再正常不过的操作，所以使用 Cloudfalre Worker 代理地址后，理论上也会降低被封禁的概率。

👉该方案的主要思路是使用 Cloudflare 的 Workers 来代理 OpenAI 的 API 地址，配合自己的域名即可在境内实现访问。因为 Cloudflare Workers 有每天免费 10 万次的请求额度，也有可以免费注册的域名，所以几乎可以说是零成本。而且该方法理论上支持所有被认证的网站，而不只是 OpenAI。

## 准备工作

使用这个方案需要你有以下东西：

一个域名（可以到 godaddy 注册一个，相信对于大家来说注册域名不是啥大问题）

注册一个 Cloudflare 账号

ℹ️ 请不要直接使用本教程示例中的地址，因为随时会被关闭。也不要使用任何其他人搭建的不受信任的地址，因为有 api key 被盗取的可能。

## 基本步骤

1. 新建一个 Cloudflare Worker
2. 将worker.js中的代码粘贴到 Worker 中并部署
3. 给 Worker 绑定一个没有被 GFW 认证的域名
4. 使用自己的域名代替 api.openai.com
5. 如果遇到问题，可以参考下面的详细版教程。

## 将域名 NS 转到 Cloudflare

如果域名已经托管在 Cloudflare 的忽略这一步即可。
Cloudflare Workers 的域名绑定仅支持托管在 Cloudflare 上的域名，所以得先将域名的 NS 转到 Cloudflare。

没有 Cloudflare 账号的话可以注册一个，具体注册细节就不多说了。

注册或登录到 Cloudflare 的管理界面后，点击侧边栏的 “Websites” ，然后点击 “Add a Site” 按钮准备将域名转到 Cloudflare：
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_1.png"/>

在 “Enter your site (example.com)” 处输入要转入的域名后，点击 “Add Site”：
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_2.png"/>

根据 Cloudflare 的提示，在域名注册商处将 NS 修改到 Cloudflare 指定的地址，等待域名解析成功后，即可进行后续操作。这儿我们参考的是阿里云的DNS服务商修改
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_3.png"/>
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_4.png"/>

## 创建一个 Cloudflare Worker

登录到 Cloudflare 的管理界面后，点击侧边栏的 “Workers” 选项，然后点击 “Create a Service” 创建一个 Worker。

<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_1.png"/>

然后在创建界面中输入 “Service name” 后点击 “Create Service” 按钮新建 Worker。
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_2.png"/>

至此 Cloudflare 的 Worker 便创建好了，下面开始修改 Worker 的代码，使其能代理 OpenAI 的 API。

## 修改 Cloudflare Worker 的代码

在 Worker 的管理界面，点击右上角的 “Quick Edit” 按钮编辑代码 Worker 的代码。
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_3.png"/>

在左侧的代码编辑器中，删除现有的所有代码，然后复制粘贴以下内容到代码编辑器：
<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_4.png"/>

```javascript
const TELEGRAPH_URL = 'https://api.openai.com';

addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url);
  url.host = TELEGRAPH_URL.replace(/^https?:\/\//, '');

  const modifiedRequest = new Request(url.toString(), {
    headers: request.headers,
    method: request.method,
    body: request.body,
    redirect: 'follow'
  });

  const response = await fetch(modifiedRequest);
  const modifiedResponse = new Response(response.body, response);

  // 添加允许跨域访问的响应头
  modifiedResponse.headers.set('Access-Control-Allow-Origin', '*');

  return modifiedResponse;
}
```

最后点击编辑器右上角的 “Save and deploy” 按钮部署该代码，在弹出的对话框中继续选择 “Save and deploy” 确认部署。

至此，便可以使用该 worker 的地址来代替 OpenAI 的 API 地址了。比如想要请求 ChatGPT 的 API 时，把官方文档中的 https://api.openai.com/v1/chat/completions 替换成 https://openaiproxy.jackzhang881026.workers.dev/ 即可（注意这个地址并不存在，是需要换成自己刚刚创建的 Worker 的地址）。

但是你可能会发现，这样做了依然还是没有解决问题，因为 Cloudflare Workers 的 workers.dev 域名也是被 GFW 认证过的🥲。但是好在只是认证了 workers.dev 域名，而 ip 还是幸存的状态，所以我们可以给 Worker 绑定一个自己的域名。

## 绑定域名

在 Cloudflare Workers 的管理界面中，点击 “Triggers” 选项卡，然后点击 “Custom Domians” 中的 “Add Custom Domain” 按钮以绑定域名。

<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_5.png"/>

输入域名后点击 “Add Custom Domain”，根据提示修改域名的 DNS 记录。因为我的域名是托管在 Cloudflare 上的，所以无需手动更改 DNS 记录，如果域名没有托管在 Cloudfalre 上，可以根据相关提示自行配置。 目前只支持 NS 托管在 Cloudflare 上的域名。

<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_6.png"/>

至此便大功告成。等待片刻，应该就可以通过你自己的域名来代替 OpenAI 的 API 地址了。

<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_7.png"/>

<br/>

最后试着访问我们绑定好的域名看看，出现OpenAI 接口调用失败的提示，就大功告成了，快去用这个地址替换https://api.openai.com/吧。比如在本文的例子中，想要请求 ChatGPT 的 API ，即是把官方 API 地址 https://api.openai.com/v1/chat/completions 换为我自己的域名 https://api.gptgo.fun/chat/completions ，其他参数均参照官方示例即可。由于 Cloudflare 有每天免费 10 万次的请求额度，所以轻度使用基本是零成本的。⚠️ 注意请不要使用我这里的 api.gptgo.fun，因为随时可能会被我关闭🤪

<br/>

<img src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/cloudflare_worker_8.png"/>

<br/>

🥰 如果本文对你有帮助，请给我的GitHub点个Star哦。


# ChatGPT相关
有兴趣看我发布的其他两个项目：

苹果Siri智能助手：[ChatGPT-Siri](https://github.com/GoGPTAI/ChatGPT-Siri)

ChatGPT中文指南：[ChatGPT-Prompt](https://github.com/GoGPTAI/ChatGPT-Prompt)

# 联系

- QQ: 471384214
- 微信：jack881026

## 微信群交流
<img width="400" src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/wxq2.jpeg" />

## 微信公众号: AI创新工坊
<img width="400" src="https://raw.githubusercontent.com/GoGPTAI/ChatGPT-Proxy/main/images/qrcode_430.jpg" />


## 👉 GoGPT 官网 - [ChatGPT直接使用](http://gogpt.vip/?channel=git)

### License

[MIT](./LICENSE)
