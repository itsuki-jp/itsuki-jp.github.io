---
title: "Hugoの公開"
date: 2024-06-03T19:17:54+09:00
draft: false
summary: Hugo公開までの試行錯誤
---
## クイックスタート出来ない...
[公式のQuick Start](https://gohugo.io/getting-started/quick-start/)通りにやると上手く行かない
```shell
echo "theme = 'ananke'" >> hugo.toml
```
これを実行すると
```shell
WARN **** found no layout file for "HTML" for kind "page": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
```
正しくは
```shell
echo "theme = 'ananke'" >> config.toml
```

## 画像が表示されない...
```html
<img src="/images/path-to-image"><br>
```
上記のように・普段HTMLに画像を埋め込むように書いても、画像が表示されなかった
tomlファイルに以下を追記する
```toml
[markup]
    [markup.goldmark]
        [markup.goldmark.renderer]
            unsafe = true
```

## 記事が公開されない...
- draftの設定を`draft:false` -> `draft: true` に変更する
- `hugo` コマンドを走らせる
  - これで書いた記事がpublicに生成される