# ChatGPT-Proxy

> OpenAI的官方接口：api.openai.com，这个接口很显然是访问不通的，

ℹ️ 近期因为 OpenAI 的风控有大量API Key或账号被封禁。因为Cloudflare 勉强也算云服务商，而从云服务商请求 API 是再正常不过的操作，所以使用 Cloudfalre Worker 代理地址后，理论上也会降低被封禁的概率。
ℹ️ 事实证明 ChatGPT 是足够火爆的，火爆到什么程度呢，其API一经推出便获得了 GFW 的认证。在 Twitter 上看到很多人都在为解决无法正常访问 OpenAI 的 API 而苦恼，最常见解决方案是使用一台服务器来进行反向代理，但这样又徒增了一些成本。因为之前在公司的业务上遇到过类似问题，当时老板找到了一个还不错的几乎零成本解决方案，试了一下现在仍然可以用来解决 OpenAI 的 API 无法访问的问题，所以在这里推荐给大家。
👉 该方案的主要思路是使用 Cloudflare 的 Workers 来代理 OpenAI 的 API 地址，配合自己的域名即可在境内实现访问。因为 Cloudflare Workers 有每天免费 10 万次的请求额度，也有可以免费注册的域名，所以几乎可以说是零成本。而且该方法理论上支持所有被认证的网站，而不只是 OpenAI。

<img src="https://user-images.githubusercontent.com/2698003/229402093-8e4f55e8-95e5-4adc-92dd-2fb6bfacce42.png" width="800" />


代理请求到 ChatGPT API，代码部署步骤：

1. 注册并登录到 Cloudflare 账户
2. 创建一个新的 Cloudflare Worker
3. 将 [cloudflare-worker.js](./cloudflare-worker.js) 复制并粘贴到 Cloudflare Worker 编辑器中
4. 保存并部署 Cloudflare Worker
5. 在 Worker 详情页 -> Trigger -> Custom Domains 中为这个 Worker 添加一个自定义域名

为啥需要第五步？因为直接使用 Cloudflare 的域名，依然无法访问。

<img src="https://user-images.githubusercontent.com/2698003/229402115-f7463a82-dd03-45e1-820c-1ab29acf1048.png" width="400" />

### 使用说明

ChatGPT 的 API 默认是非流式输出的，如果想让他变成流式输出，需要将 `payload.stream` 设置为 true，大部分的客户端都已经加上了这个参数。

https://github.com/barretlee/cloudflare-proxy/blob/a7cf8ecfd3eed5c4d76f82f4f4387ed4ef39c6f3/cloudflare-worker.js#L36-L50

### License

[MIT](./LICENSE)
