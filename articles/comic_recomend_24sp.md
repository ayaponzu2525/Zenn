---
title: "esp32とneopixelを使った7セグ風BigClock"
emoji: kissing_face_with_closed_eyes
type: idea
topics: [esp32, neopixel, M5stack, 電子工作, Arduino]
published: false
---

# はじめに
今回はesp32とneopixelを使用して7セグ風のでっかい時計を作りました。

# 実装した機能
* 3分に１回、ntpサーバーから正確な時間を取得
* 正確な時間をneopixelに表示させる
* ntpサーバーから取得した時間からesp32自身でカウントアップ
* M5stackを用いてサーバーを立ててサーバーにある任意の数字の時間に変更できる

# 材料
* esp32
* neopixel(74個のLED)
* 板（今回は百均の石膏ボード）
* 

# プログラム編の大まかな手順(私が担当したところ)
1. esp32でntpサーバーに接続して正確な時間を取得する
2. 

# 大まかな手順(ハード編)
1. 板にneopixelを貼る
2. はんだづけ
3. neopixelを任意の数字に光らせる
今回私は基本的にプログラムをしていたのでハード編の詳細な記事はこちら↓

url


# esp32でntpサーバーに接続して正確な時間を取得する


# これからの展望（つぎどんなことしたいか）