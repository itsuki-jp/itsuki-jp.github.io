---
title: "2024 10 26_OGP_title_on_image"
date: 2024-10-26T12:14:51+09:00
draft: true
---

assets配下に置く

下で試す

{{- if $img }}
{{ $img := $img.Filter (images.GaussianBlur 6) (images.Pixelate 8) }}
  
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:image" content="{{ $img.Permalink }}">
{{- else }}
  <meta name="twitter:card" content="summary">
{{- end }}

成功の全体コード

```html
{{- $font := resources.Get "https://github.com/google/fonts/raw/main/ofl/notosansjp/NotoSansJP-Black.otf" }}
{{- $img := resources.Get "images/background_image.jpg" }}

{{- if $img }}
{{ $img := $img.Filter (images.GaussianBlur 100) (images.Pixelate 8) }}
  
  <meta name="twitter:card" content="summary_large_image">
  <meta name="twitter:image" content="{{ $img.Permalink }}">
{{- else }}
  <meta name="twitter:card" content="summary">
{{- end }}

{{- with or .Title site.Title site.Params.title | plainify }}
  <meta name="twitter:title" content="{{ . }}">
{{- end }}

{{- with or .Description .Summary site.Params.description | plainify | htmlUnescape }}
  <meta name="twitter:description" content="{{ trim . "\n\r\t " }}">
{{- end }}

{{- $twitterSite := "" }}
{{- with site.Params.social }}
  {{- if reflect.IsMap . }}
    {{- with .twitter }}
      {{- $content := . }}
      {{- if not (strings.HasPrefix . "@") }}
        {{- $content = printf "@%v" . }}
      {{- end }}
      <meta name="twitter:site" content="{{ $content }}">
    {{- end }}
  {{- end }}
{{- end }}

```