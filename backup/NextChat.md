

✨ Light and Fast AI Assistant,with Claude, DeepSeek, GPT4 & Gemini Pro support. 


[NextChatAI](https://nextchat.club?utm_source=readme) / [Web App Demo](https://app.nextchat.dev) / [Desktop App](https://github.com/Yidadaa/ChatGPT-Next-Web/releases) / [Discord](https://discord.gg/YCkeafCafC) / [Enterprise Edition](#enterprise-edition) / [Twitter](https://twitter.com/NextChatDev)


[saas-url]: https://nextchat.club?utm_source=readme
[saas-image]: https://img.shields.io/badge/NextChat-Saas-green?logo=microsoftedge
[web-url]: https://app.nextchat.dev/
[download-url]: https://github.com/Yidadaa/ChatGPT-Next-Web/releases
[Web-image]: https://img.shields.io/badge/Web-PWA-orange?logo=microsoftedge
[Windows-image]: https://img.shields.io/badge/-Windows-blue?logo=windows
[MacOS-image]: https://img.shields.io/badge/-MacOS-black?logo=apple
[Linux-image]: https://img.shields.io/badge/-Linux-333?logo=ubuntu

[<img src="https://zeabur.com/button.svg" alt="Deploy on Zeabur" height="30">](https://zeabur.com/templates/ZBUEFA) [<img src="https://vercel.com/button" alt="Deploy on Vercel" height="30">](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2FChatGPTNextWeb%2FChatGPT-Next-Web&env=OPENAI_API_KEY&env=CODE&project-name=nextchat&repository-name=NextChat)  [<img src="https://gitpod.io/button/open-in-gitpod.svg" alt="Open in Gitpod" height="30">](https://gitpod.io/#https://github.com/ChatGPTNextWeb/NextChat) 

[<img src="https://github.com/user-attachments/assets/903482d4-3e87-4134-9af1-f2588fa90659" height="50" width="" >](https://monica.im/?utm=nxcrp)



## Dokploy

Dokploy 是一个免费的、可自托管的平台即服务 （PaaS），可简化应用程序和数据库的部署和管理。

```
curl -sSL https://dokploy.com/install.sh | sh
```

## Development


[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/Yidadaa/ChatGPT-Next-Web)

Before starting development, you must create a new `.env.local` file at project root, and place your api key into it:

```
OPENAI_API_KEY=<your api key here>

# if you are not able to access openai service, use this BASE_URL
BASE_URL=https://chatgpt1.nextweb.fun/api/proxy
```

### Local Development

```shell
# 1. install nodejs and yarn first
# 2. config local env vars in `.env.local`
# 3. run
yarn install
yarn dev
```

## Deployment


### Docker (Recommended)

```shell
docker pull yidadaa/chatgpt-next-web

docker run -d -p 3000:3000 \
   -e OPENAI_API_KEY=sk-xxxx \
   -e CODE=your-password \
   yidadaa/chatgpt-next-web
```

You can start service behind a proxy:

```shell
docker run -d -p 3000:3000 \
   -e OPENAI_API_KEY=sk-xxxx \
   -e CODE=your-password \
   -e PROXY_URL=http://localhost:7890 \
   yidadaa/chatgpt-next-web
```

If your proxy needs password, use:

```shell
-e PROXY_URL="http://127.0.0.1:7890 user pass"
```

If enable MCP, use：

```
docker run -d -p 3000:3000 \
   -e OPENAI_API_KEY=sk-xxxx \
   -e CODE=your-password \
   -e ENABLE_MCP=true \
   yidadaa/chatgpt-next-web
```

### Shell

```shell
bash <(curl -s https://raw.githubusercontent.com/Yidadaa/ChatGPT-Next-Web/main/scripts/setup.sh)
```

## LICENSE

[MIT](https://opensource.org/license/mit/)