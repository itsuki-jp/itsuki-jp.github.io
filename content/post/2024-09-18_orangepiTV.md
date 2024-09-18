---
title: "orangepi でテレビを見たい"
date: 2024-09-18T20:28:37+09:00
draft: false
---
# はじめに
パソコンとかスマホでテレビ、見たいですよね！！！

ネットで調べてみると、 ラズパイの情報はたくさんありますが、私が持ってる Orange Pi の情報は一切なかったです...(2024年9月現在)

なので、自分でやってみました！

# 使用機材
- [OrangePi 3B 4GB](https://ja.aliexpress.com/item/1005005919873216.html)
- [PX-S1UD V2.0 (テレビチューナー)](https://amzn.asia/d/ivrVLJ1)
- [SCR3310V2.0 (B-CAS リーダー)](https://amzn.asia/d/do9xBhP)
- [(mini B-CAS を B-CAS に変換するアダプター)](https://amzn.asia/d/aOkgRfc)

## 前提
方法は忘れてしまいましたが、 OrangePi は SSD から起動できるようにしています

# 困った事等
参考にしたサイト：[Raspberry Pi 4 と docker-mirakurun-epgstation で録画サーバーを構築する (2023年2月版)](https://blog.ch3cooh.jp/entry/2023/02/25/174845#PX-S1UD-V20-%E3%83%81%E3%83%A5%E3%83%BC%E3%83%8A%E3%83%BC%E3%81%AE%E3%83%89%E3%83%A9%E3%82%A4%E3%83%90%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)

## 詰まったこと1: Docker
### 詰まった
`sudo apt install -y git docker docker-compose`
`docker -v`
ここで、dockerのバージョンが表示されない。docker-composeは表示された
### 解決策
[【簡単な4つの方法】UbuntuにDockerをインストールするには](https://kinsta.com/jp/blog/install-docker-ubuntu/#ubuntudocker-engine)
```shell
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker

sudo reboot
```

## 詰まったこと２：リーダーが上手く動いてない
### 詰まったこと
`curl -X PUT "http://orangepi-ip-address:40772/api/config/channels/scan"`
でスキャンが上手く行かない
```
channel: "24" (12/50) [24%] ...
-> 1 existing config found.
-> {"name":"テレビ朝日","type":"GR","channel":"24"}
# scan has skipped due to the "refresh = false" option because an existing config was found.
```
### 解決策
まずリーダーが認識されてるか確認する
`lsusb`

次に、B-CASが読み取れているか確認
`sudo pcsc_scan`
私は mini B-CAS が読み込まれてなかったのをこれで気づくことが出来ました！（なのでアダプターを購入した）

`curl -X PUT "orangepi-IP:40772/api/config/channels/scan?refresh=true"`

```terminal
channel: "27" (15/50) [30%] ...
-> 2 services found.
-> {"name":"ＮＨＫ総合","type":"GR","channel":"27","isDisabled":false}
```

# 結果
見れるようになりました！（さすがにテレビ画面は権利的に良くない気がするので...）
<img src="/images/2024-09-18_orangepiTV/image.png" width=50%>
<img src="/images/2024-09-18_orangepiTV/image-1.png" width=50%>

# 最後に
結果でも書いたように見れるようになりましたが、OrangePiのスペック的に480pでも偶に止まります...
ただ、録画された番組はサクサク見れるので、そのように運用すると良さそうです

また、エンコードも重いです。結構時間がかかります...

<img src="/images/2024-09-18_orangepiTV/image-2.png" width=50%>