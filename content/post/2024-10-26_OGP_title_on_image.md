---
title: "HugoのOGPにて画像にタイトルを表示したい！"
date: 2024-10-26T12:14:51+09:00
draft: false
summary: いい感じの背景画像を思いついたら変えたいね
---
# はじめに
[HugoでもOGP(サムネ)を設定したい！](https://itsuki-jp.github.io/post/2024-10-25_ogp/)の最後でも書いたのですが、QiitaのOGPみたいな画像＋タイトルが表示されるやつを作る回です。

![alt text](/images/2024-10-26_OGP_title_on_image/image.png)

こんなやつ

結果としては、こんな感じのが作れました！

![alt text](/images/2024-10-26_OGP_title_on_image/image-1.png)


参考：https://www.kofuk.org/blog/20240815-hugo-ogp-image/


# やったこと
- 好きなフォントを`assets/fonts/`に置く
- 背景画像に使いたい画像を`assets/images/`に置く

<details>
<summary>twitter_cards.htmlに追記・書き換える</summary>

```html
{{ $font := "" }}
{{ $path := "fonts/rounded-mgenplus-1c-bold.ttf" }}
{{ with resources.Get $path }}
  {{ with .Err }}
    {{ errorf "%s" . }}
  {{ else }}
    {{ $font = . }}
  {{ end }}
{{ else }}
  {{ errorf "Unable to get resource %q" $path }}
{{ end }}

{{ $opts := dict
  "color" "#000000"
  "font" $font
  "linespacing" 8
  "size" 60
  "x" 50
  "y" 35
}}

{{ $text := .Title | default "n_itsuki's blog" }} 

{{- $maxLen := 16 }}
{{- $len := strings.RuneCount $text -}}
{{- $titleText := "" -}}
{{- range $i := seq 0 (div $len $maxLen) }}
  {{- $line := substr $text (mul $i $maxLen) $maxLen }}
  {{- $titleText = printf "%s%s\n" $titleText $line }}
{{- end -}}

{{ $filter := images.Text $titleText $opts }}

{{ with resources.Get "images/OGP_image.jpg" }}
  {{ with . | images.Filter $filter }}
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:image" content="{{ .Permalink }}">
  {{ end }}
{{ end }}
```
</details>

<details>
<summary>opengraph.htmlに追記・書き換える</summary>

```html
{{ $font := "" }}
{{ $path := "fonts/rounded-mgenplus-1c-bold.ttf" }}
{{ with resources.Get $path }}
  {{ with .Err }}
    {{ errorf "%s" . }}
  {{ else }}
    {{ $font = . }}
  {{ end }}
{{ else }}
  {{ errorf "Unable to get resource %q" $path }}
{{ end }}

{{ $opts := dict
  "color" "#000000"
  "font" $font
  "linespacing" 8
  "size" 60
  "x" 50
  "y" 35
}}

{{ $text := .Title | default "n_itsuki's blog" }} 

{{- $maxLen := 16 }}
{{- $len := strings.RuneCount $text -}}
{{- $titleText := "" -}}
{{- range $i := seq 0 (div $len $maxLen) }}
  {{- $line := substr $text (mul $i $maxLen) $maxLen }}
  {{- $titleText = printf "%s%s\n" $titleText $line }}
{{- end -}}

{{ $filter := images.Text $titleText $opts }}

{{ with resources.Get "images/OGP_image.jpg" }}
  {{ with . | images.Filter $filter }}
    <meta property="og:image" content="{{ .Permalink }}">
  {{ end }}
{{ end }}
```
</details>

# 詰まったこと
## 画像が読み込まれない...(localhostにて)
永遠に「画像が読み込めないよ～」とエラーが表示され続けました。`static`フォルダに画像を格納していたことが原因でした...

テキストを描写できるかを試す前に、そもそも画像が読み込めるか・加工できるかを確かめましょう！

```html
{{- $img := resources.Get "images/background_image.jpg" }}
{{- if $img }}
  {{ $img := $img.Filter (images.GaussianBlur 100) (images.Pixelate 8) }}
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:image" content="{{ $img.Permalink }}">
{{- else }}
    <meta name="twitter:card" content="summary">
{{- end }}
```
[Directory structure](https://gohugo.io/getting-started/directory-structure/)を読むと、
- assets: グローバルリソースを置く。`assets pipeline`に渡される
- static: 静的ファイルを置く。

##  画像表示されない（GitHub Pages公開後にて）
画像のパスを`Permalink`に変えると絶対パスにしてくれるので表示ます！

## 全然文字が表示されない...
[公式通り](https://gohugo.io/functions/images/text/)にやってるのに、全然文字が表示されませんでした。画像のサイズが大きく、文字が小さく表示されていたのが原因でした...(左上に小さく書いてあります！)

文字の大きさを適当に変更してみましょう！

![alt text](/images/2024-10-26_OGP_title_on_image/image-2.jpg)

# 最後に
`resources/_gen`はgitignoreに書いたほうがいいですね。これが突っ込まれてました
![alt text](/images/2024-10-26_OGP_title_on_image/image-3.png)