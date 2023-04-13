---
title: "OpenAI API 代理"
date: 2023-04-13T11:05:48+08:00
categories: ["技术总结"]
tags: ["OpenAI", "ChatGPT"]
url: openai-api-proxy

################################目录################################
toc: false
autoCollapseToc: false
################################评论################################
comment: false
################################公式渲染################################
mathjax: false

################################基本不动################################
author: "Yaxing"
draft: false
hiddenFromHomePage: false
contentCopyright: true
postMetaInFooter: false
# keywords: []
# description: ""
---

之前部署了一个 ChatGPT 服务后，发现还是需要挂梯子才能访问，想着有没有办法不挂梯子就可以访问。<!--more-->

如果有自己的服务器的话，可以使用服务器来进行反向代理，由于本人没有服务器，所以采用的另一种方法，使用 Cloudflare 的 Workers 来代理 OpenAI 的 API 地址，配合自己的域名即可实现直接访问。以下内容来自 [利用Cloudflare Worker中转api.openai.com](https://github.com/x-dr/chatgptProxyAPI/blob/main/docs/cloudflare_workers.md)。

## 新建 Cloudflare Worker

1、登录到 Cloudflare 的管理界面后，点击侧边栏的 `Workers` 选项，然后点击 `Create a Service` 创建一个 Worker。

2、然后在创建界面中输入 `Service name` 后，`Select a starter` 项选择 `HTTP Handler`，点击 `Create Service` 按钮新建 Worker。

## 修改 Cloudflare Worker 的代码

1、在 Worker 的管理界面，点击右上角的 “Quick Edit” 按钮编辑代码 Worker 的代码。在左侧的代码编辑器中，删除现有的所有代码，然后复制粘贴以下内容到代码编辑器：

```javascript
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    url.host = "api.openai.com";
    // openai is already set all CORS heasders 
    return fetch(url, {
      headers: request.headers,
      method: request.method,
      body: request.body,
      redirect: 'follow'
    });
  }
}
```

2、最后点击编辑器右下角的 `Save and deploy` 按钮部署该代码，在弹出的对话框中继续选择 `Save and deploy` 确认部署。

## 绑定域名

1、在 Cloudflare Workers 的管理界面中，点击 `Triggers` 选项卡，然后点击 `Custom Domians` 中的 `Add Custom Domain` 按钮以绑定域名。

2、输入域名后点击 `Add Custom Domain` (目前只支持 NS 托管在 Cloudflare 上的域名，如果不介意，可以点击 Cloudflare 侧边栏的 “Websites”，然后点击 “Add a Site” 按钮，根据提示将域名的 NS 记录指定到 Cloudflare。)

## 测试

1、对话

```http
curl --location '{custom domain}/v1/chat/completions' \
--header 'Authorization: Bearer sk-xxxxxxxxxxxxxxx' \
--header 'Content-Type: application/json' \
--data '{
   "model": "gpt-3.5-turbo",
  "messages": [{"role": "user", "content": "Hello!"}]
 }'
```

2、响应

```json
{
    "id": "chatcmpl-6rMlZybwjMQIhFAEaiCmWvMP1BXld",
    "object": "chat.completion",
    "created": 1678176917,
    "model": "gpt-3.5-turbo-0301",
    "usage": {
        "prompt_tokens": 9,
        "completion_tokens": 11,
        "total_tokens": 20
    },
    "choices": [
        {
            "message": {
                "role": "assistant",
                "content": "\n\nHello! How can I assist you today?"
            },
            "finish_reason": "stop",
            "index": 0
        }
    ]
}
```
