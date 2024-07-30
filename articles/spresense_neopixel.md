---
title: "SpresenseでNeopixelのsetPixelColorを使いたい！"
emoji: kissing_face_with_closed_eyes
type: idea
topics: [Spresense, Neopixel, 電子工作, Arduino]
published: false
---

## きっかけ
こんにちは！ayaponzuです。
今回の記事を書こうと思ったきっかけについてまずは書こうと思います。

今、僕はチームでSonyのSpresenseを使ってハッカソンに出す巡回クンという名前の学校の教室を巡回しながら質問のある子に答えるというロボットを作っています。
そのなかでNeopixelをつかってロボットの状態をLEDで表せれたらいいよね！となったのでNeopixelを配線して光らそう！とAdafruit Neopixelのライブラリを使って簡単なプログラムを書いて光らせようとしたところ、書き込みには成功しますが、まったく光りません。

同じコードを Arduino Uno に書き込んで試したところすんなりレインボーに光りました。
ということはSpresenseはAdafruitのライブラリのsetPixelColor関数は使えないということになります。

めっちゃ困りましたね...

ということでSpresenseでも使えるライブラリを探そう！となり、そこからNeopixelを好きな色、明るさで光らせられるようになるまでのお話です。

## ライブラリの入れ方について
これからいろいろなライブラリを紹介していきますが、そのライブラリをArduinoIDEに入れる方法をここで書いておきます。

### ライブラリの保存場所について
ArduinoIDEのライブラリはスケッチと一緒の場所に保存されています。各OSごとの保存場所は以下の通りです。
ちなみに僕はWindowsです。

| OS | 場所 |
| ---- | ---- |
| Windows | C:￥Users￥ユーザー名￥Documents￥Arduino |
| Mac | /Users/ユーザー名/Documents/Arduino |
| Linux | /home/ユーザー名/Arduino |

このディレクトリの保存先はArduinoIDEの [ファイル -> 基本設定 -> スケッチブックの場所]の参照ボタンを押して任意の場所に変更できます。

![](https://storage.googleapis.com/zenn-user-upload/a87e1590c89b-20240716.png)

## ライブラリを探す
まずは使そうなライブラリを探しました。

一つ目に見つけたのが KotaMeiwa/nepils です。
@[card](https://github.com/KotaMeiwa/nepils)<br>

こちらはかの有名なAdafruit_NeoPixel を継承しているらしく、今までそれを使ってきた人はすんなり使えると思います。が、'setPixelColor' は使えませんでした。なんでだろ？

2つ目に見つけたのは

## ライブラリを見つけた

## サンプルコード

##  