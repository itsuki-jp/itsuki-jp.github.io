---
title: Gitで困ったあれやこれ
date: 2024-12-03T21:00:22+09:00
draft: true
summary: Git・GitHubって難しいですよね
---
# はじめに
もう12月ですね。4月に新卒として入社し、半年以上たっているらしいですね。あまりにも早い。

私の会社は、開発エンジニアは9月まで全体研修的な「いろいろな製品を触ってみよう！」「チームで開発してみよう！」があり、夏休みを経て9月半ばから本配属されます。なので、まだ実質社会人3か月みたいなものですね。

まだ始まって間もない社会人人生ですが、Gitで詰まってあーでもないこーでもない、とオロオロする時間はそこそこにあったので、備忘録として残しておきます。このメモを自分で参照する日が来ないことを願っておきます。（一部間違えていそうなので、なおさら参照する日が来ないことを祈る...）

# Gitで困ったあれやこれ
## 改行コードが原因で大量の差分！
先輩に「たくさん変更したね！」と言われ、「50くらいでも褒めてくれる先輩！優しい！」と思ってたら、10,000行とか変更されてました。原因は私の開発環境がWindowsで、適当にGitの設定をしたことっぽかったです。

基本はautocrlfをtrueにしておけば良いはずなのですが、謎に改行コードが違うファイルが転がってるときもあります
```sh
git config --global core.autocrlf false

git config --global -e # エディタを用いて編集もできる
```

## ブランチ切り替えるの忘れて、前作業してたブランチにプッシュしてしまった..
これは、配属されてからも、配属前のチーム開発でもやらかしました。個人開発であれば、正直ブランチなんてあってないようなものですが、チーム開発ましてや製品となると、非常に良くないですね。結構肝を冷やしました...
```sh
git reset --soft HEAD^
git stash -m "hogehoge"
git stash list // 確認
git push -f
```

## ブランチ間違えて生やしちゃった...
ひとつ前の事例と似てますね。ブランチを生やすときは、自分がどこにいるかをちゃんと確かめてから生やしましょう。

```sh
git rebase --onto 正しいベース 間違ったベース 作業ブランチ
```

## なんでこんな処理になってるんだろう？特定の行をこれまで編集した人を知りたい！
このコード意味わからんなぁと頭をひねってもわからないのであれば、自分ではわかりません。実装した人がまだプロジェクトにいれば聞いたほうが良い。
```sh
git blame
```

## Git: You have not concluded your merge (MERGE_HEAD exits).
何度も見たエラー。どうやらマージができないらしい。
```sh
git reset --merge
main に戻り git pull (いらないかも)
作業ブランチに戻り git  commit
git push
```

## いらんコミットしちゃった...
いらんコミットは、消しましょう。ただ、二個以上前のいらんコミットは消すのがめんどくさいらしいです。気を付けましょう（自戒）
```sh
git reset --hard commitID
git push --force
```

## 特定のコミット時点のコードを見たいな
うーん、この時は動いてたと思うんだけどな...の時にめちゃ役立ちます
```sh
git checkout ハッシュ値
```

## PRを出したら、コンフリクトしてた！
後はPR書くだけ！と思ってたのに、コンフリクト...非常に悲しいです。
```sh
git checkout main
git pull
git checkout 作業ブランチ
git merge main
コンフリクトあるよ～とvs code が教えてくれるので、適切に直す
コミッ、プッシュ
```