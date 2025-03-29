---
title: "koyeb * gemini apiで無料でdiscord botを作る"
date: 2025-03-29T09:45:53+09:00
draft: false
summary: 無料枠で作るdiscord bot
categories: [開発]
tags: [discord, bot, koyeb, gemini]
---
# はじめに
なんか作りたいなぁと思ってとりあえずDiscordBotを作りました。

ただ、家に常時稼働のサーバーがあるわけがなく...金欠なので無料で全部動かす！をテーマです。

今の機能はただ「ユーザのメッセージをGemini APIに投げてメッセージとして投稿する」だけですが、今後はもう少しやっていきたいですね

完成したBot

![alt text](/images/2025-03-29_discord_bot/image.png)

# 作り方

## コード関連 

下のようなディレクトリ構成にします  ( `tree -a -I 'venv|.git'`  )
```
.
|-- .env
|-- .gitignore
|-- Dockerfile
|-- app
|   `-- main.py
|-- docker-compose.yml
|-- requirements.txt
```

### `.env`
```.env
GEMINI_API=hogehoge
TOKEN=hugahuga
```
TOKENはDiscordBotのトークンを入れます。{{<linkcard "https://discord.com/developers/applications" >}}　からBotの設定をする必要がありました（うろ覚えですが、めんどくさかった気がします）

### `Dockerfile`
<details>
<summary>Dockerfile</summary>

```dockerfile
FROM python:3.11
WORKDIR /bot

RUN apt-get update && apt-get -y install locales && apt-get -y upgrade && \
    localedef -f UTF-8 -i ja_JP ja_JP.UTF-8
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
ENV TZ Asia/Tokyo
ENV TERM xterm

RUN pip install --upgrade pip

COPY requirements.txt /bot/
RUN pip install -r requirements.txt
COPY . /bot

EXPOSE 8080

CMD ["python", "app/main.py"]
```

</details>

### `main.py`

ここでキモなのは、ダミーサーバーを立てることころです。

> discord.gateway: Shard ID None has connected to Gateway (Session ID: hogehoge).
TCP health check failed on port 8080.
Instance stopped.

どうやら、後述するKoyebは8080ポートが開いていないとHealthCheckに失敗してしまうらしいです。（FastAPIとかで立てればいいじゃん？と思いますが、若干過剰な気がするので...）

<details>
<summary>main.py</summary>

```python
import discord
import os
import base64
import requests
from http.server import BaseHTTPRequestHandler, HTTPServer
import threading

TOKEN = os.environ.get("TOKEN")
GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY")
intents = discord.Intents.default()
intents.message_content = True
client = discord.Client(intents=intents)

class DummyHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"OK")


def run_dummy_server():
    server = HTTPServer(("0.0.0.0", 8080), DummyHandler)
    server.serve_forever()


threading.Thread(target=run_dummy_server, daemon=True).start()


def call_gemini(text_prompt, image_data=None):
    url = f"https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key={GEMINI_API_KEY}"
    contents = [{"parts": [{"text": text_prompt}]}]

    if image_data:
        contents[0]["parts"].append(
            {"inlineData": {"mimeType": "image/png", "data": image_data}}
        )

    payload = {"contents": contents}
    headers = {"Content-Type": "application/json"}

    res = requests.post(url, headers=headers, json=payload)
    if res.status_code == 200:
        return res.json()["candidates"][0]["content"]["parts"][0]["text"]
    else:
        return f"Error: {res.status_code} {res.text}"

@client.event
async def on_ready():
    print(f"We have logged in as {client.user}")


@client.event
async def on_message(message):
    if message.author == client.user:
        return
    prompt = message.content
    image_data = None

    logging.debug("debug")
    if message.attachments:
        image = await message.attachments[0].read()
        image_data = base64.b64encode(image).decode("utf-8")

    await message.channel.send("🤖 Thinking...")
    result = call_gemini(prompt, image_data=image_data)
    await message.channel.send(result)


client.run(TOKEN)
```

</details>

### `docker-compose.yml`
Koyebで完結させるのであれば不要ですが、ローカルでテストしてから動かしたいので...


`docker-compose up -d`で起動, `docker-compose down`で落とせるので便利ですね

<details>
<summary>docker-compose.yml</summary>

```yaml
version: "3"
services:
    discord-bot:
        build:
            context: .
        container_name: discord-bot
        ports:
            - "8080:8080"
        env_file:
            - .env
        restart: unless-stopped
```

</details>

### `requirements.txt`
```txt
discord.py
requests
```

## Koyebにデプロイする
GitHubと連携できるので、さっきのコードを含むレポジトリを連携します。

基本的に適当にやったらよさそうですが、「インスタンス」だけ`CPU Eco` `Free`にする必要があります。


また、`.env`の内容も設定する必要があります。

終わりです！！！と言いたいものの、もう一つ無料ユーザーは設定をする必要があります

## cron
Koyebの無料プランは1時間何もアクションがないとSleepモードに入ります

そのため、定期的にHTTP/1.1 GETリクエストを投げる必要があります。uptimerobotを使おうと思ったのですが、無料ユーザーは`HEAD`しか投げられないので、{{<linkcard "https://console.cron-job.org/" >}} を使いました。

cron-jobはデフォルトの設定でGETできるので、30分に一回確認するように設定したところちゃんと動いている気がします。

{{<linkcard "https://www.koyeb.com/docs/run-and-scale/scale-to-zero">}}

> Your Service will be scaled down to zero if all of the following conditions are met for a given period of time called idle period:
> - No traffic is received from the Internet.
> - No held connection (e.g. websocket or HTTP/2 stream) from the Internet to your Service.
> - No new deployment occurred.

{{<linkcard "https://www.koyeb.com/docs/run-and-scale/scale-to-zero#limitations">}}

> Inbound requests to a sleeping Service may be slower due to a cold start, which typically takes 1 to 5 seconds to create a new dedicated virtual machine
Scale-to-zero works only for Services exposed to the Internet.
HTTP/2 requests cannot be used to wake up a sleeping Service.
You can wake a Service up using a WebSocket connection, but that connection may only live for a few minutes.

# さいごに
2年くらい前にDiscordBotを作ったことがあるものの、記憶がなくなっていました...記憶力が悪いとめんどくさいですが、毎回新鮮な気持ちで取り組めるのでいいですね（？）


＊参考：{{< linkcard "https://zenn.dev/maguro_alterna/articles/65906deef48e2b" >}}