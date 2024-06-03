---
title: "WSLと格闘した話"
date: 2022-12-20
draft: false
summary: はじめてのQiita
---
([Qiitaはじめての記事](https://qiita.com/n_itsuki/items/62d1c69eb368bf2f3862)から)

# はじめに

ちょうど一週間前、WSLが動かなくなって試行錯誤したときの話です。
やったこと、直した方法などを書いていきます。

# 経緯
- Flutterに挑戦しようと思い、環境構築をする
- この際に、コントロールパネル→Windowsの機能→Virtual Machine Platformなどの設定をいじる 
- Flutterの環境は上手く入れることができたものの、VS CodeのWSLターミナルが動かないことに気づく...
って感じでした
![image.png](/images/WSLと格闘した話/img1.png)

上がエラー画面

# 決定打
Virtual Machine PlatformとHyper-VとWSLを無効化して再起動してから再度チェックを入れる（後輩に助けてもらいました、ありがたすぎる!!!）

# 決定打にたどり着くまでにやったこと
- 設定を元に戻し、再起動する
    - 治らない。BIOSで確認しても正しく設定されているのに...


- Windows Subsystem for Linuxのチェックを外し、再起動
    - 治らない


- 何もわからないので、WSL, Ubuntu共に消す
    - 下記のように、謎にUbuntuが2つはいっていたので両方消した
![image.png](/images/WSLと格闘した話/img2.png)


- 再度入れ直すが、エラーが出る
![image.png](/images/WSLと格闘した話/img3.png)


- Ubuntuが悪いのでは？とおもい、Debianを入れるが、同様のエラー
![image.png](/images/WSLと格闘した話/img4.png)

- 「決定打」に書いてある、「Virtual Machine PlatformとHyper-VとWSLを無効化して再起動してから再度チェック」をする
![image.png](/images/WSLと格闘した話/img5.png)
![image.png](/images/WSLと格闘した話/img6.png)
    - 無事インストールが上手くできた！

## 道中のツイート
<!-- https://twitter.com/itsuki_jpnlonvn/status/1599663103708647424 -->
{{< twitter user="itsuki_jpnlonvn" id="1599663103708647424" >}}
{{< twitter user="itsuki_jpnlonvn" id="1599699505884327936" >}}

# やるべきだったこと
- 関係しそうな機能のチェックを全部外し、再起動→再度チェックを入れる
    - 一つ一つ外して、入れて...を繰り返していたため時間かかった

# 入れ直した後にやったこと
- Githubの再設定
    - その際に、lock failed: Operation not permitted とエラーが出た
    - https://alessandrococco.com/2021/01/wsl-how-to-resolve-operation-not-permitted-error-on-cloning-a-git-repository
        - これで解決できた（らしい、記憶にない）

# さいごに?
先日サポーターズさんのハッカソンに参加した際、cmdだと正常に動くのに、WSLだと永遠に動かない自体に遭遇した。なので、最近はWSLをほぼ使っていないです...(費やした時間がもったいない...)

