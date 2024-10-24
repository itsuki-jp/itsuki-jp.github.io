---
title: "Hugoの記事にコメントできるようにしたい"
date: 2024-10-24T23:06:51+09:00
draft: false
summary: 中国語の海を彷徨いました
---

このサイトは自分のメモ書きなのでコメント機能は不要なのですが、使ってるテーマの`README.md`にこんなことが書かれてました

```md
## Use [gitalk](https://github.com/gitalk/gitalk) to support comments

add to `config.toml`

[params.gitalk]
    clientID = "Your client ID"
    clientSecret = "Your client secret"
    repo = "repo"
    owner = "Your Github username"
    admin = "Your Github username"
    id = "location.pathname"
    labels = "gitalk"
    perPage = 30
    pagerDirection = "last"
    createIssueManually = true
    distractionFreeMode = false

```

最初は「どうやらgitalkを使うとコメント機能が使えるらしい、ロマンだな。やるか」くらいの軽い気持ちだったのですが、案外詰まったので...

## TL;DR
https://utteranc.es/ でscriptを生成して、適当なところに張り付ける

## gitalk セキュリティゆるゆる問題
GitHub Application を登録して、`README` にも書いてあるように、諸々設定してあげればよいのですが`config.toml`に機密情報を載せるのは流石に良くないです。

[issue](https://github.com/gitalk/gitalk/discussions/483)を見てみると、`uglifyjs`という難読化ツールを使うといいよ！と書いてありました。が、難読化はあくまで「難読化」なので頑張ったら外部の人でも復元できちゃうのでは？と思いgitalkはやめました

（他の中国語のサイトもいろいろと回ったのですが、これ以上は特になかったです。なぜ中国語のが多いかというと、中国からでもアクセスできるかららしい？）

## Isso めんどくさい問題
[静的生成ウェブサイトに Isso でコメントシステム導入](https://qiita.com/tanabe13f/items/b4417191d5369d1aadbb)

他のコメントツールを調べてると、Isso を発見しました。ただ、これは自前でサーバーを用意する必要があるらしく、お金かかるしめんどくさいので止めました。

ちなみに、ロゴ？はソーナノです。かわいい

![alt text](/images/2024-10-24_add_comment/image.png)

## 解決策：utterances 
解決策は[`utterances`](https://utteranc.es/)でした！中国サイトを彷徨ってるときに見つけました

これは、`gitalk`と同様に、GitHubのissueでコメントを管理してくれて、サーバーなども自前で用意する必要がなくめちゃ便利です！！！

> When Utterances loads, the GitHub issue search API is used to find the issue associated with the page based on url, pathname or title. If we cannot find an issue that matches the page, no problem, utterances-bot will automatically create an issue the first time someone comments.
To comment, users must authorize the utterances app to post on their behalf using the GitHub OAuth flow. Alternatively, users can comment on the GitHub issue directly.

Hugoへの導入方法は簡単で、
1. パブリックなレポジトリを用意する（今回の場合だとgithub.ioのレポジトリ）
2. 用意したレポジトリにhttps://github.com/apps/utterances をインストールする
3. Hugoのそれっぽいところにコードを張り付ける
   1. [私が使ってるテーマは`post.html`があったので、それの下らへんに張った](https://github.com/itsuki-jp/github-style/commit/af2d4ab7b673af4be55ced38fcf2208c784cec78)

```html
<script src="https://utteranc.es/client.js"
    repo="itsuki-jp/itsuki-jp.github.io"
    issue-term="title"
    label="comments"
    theme="github-dark"
    crossorigin="anonymous"
    async>
</script>
```

上のコードでは、以下の設定をしてます
- issueのタイトルは記事のタイトル
- issueのラベルは`comments`
- コメントのテーマは`github-dark`

他にも、いろいろと設定があり、クリックするだけでいいので自分好みの設定にしてください！

サイト：https://utteranc.es/

![alt text](/images/2024-10-24_add_comment/image-1.png)

### 結果
こんな感じで、コメントが打てるようになります、やったね🎉🎉🎉

後で気づいたのですが、https://gohugo.io/content-management/comments/ に `Isso`, `Utterances`が載ってました。最初は公式を読むとよさそうですね...

（ロゴがフワライドみたいでかわいい）

![alt text](/images/2024-10-24_add_comment/image-4.png)
![alt text](/images/2024-10-24_add_comment/image-3.png)