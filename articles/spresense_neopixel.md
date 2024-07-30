---
title: "SpresenseでNeopixelのsetPixelColorを使いたい！"
emoji: kissing_face_with_closed_eyes
type: idea
topics: [Spresense, Neopixel, 電子工作, Arduino]
published: True
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

![](https://storage.googleapis.com/zenn-user-upload/56e4b262434a-20240730.jpg)
↑巡回クン

@[card](https://protopedia.net/prototype/5288)

## ライブラリの入れ方について
これからいろいろなライブラリを紹介していきますが、そのライブラリをArduinoIDEに入れる方法をここで書いておきます。

### ライブラリの保存場所について
ArduinoIDEのライブラリはスケッチと一緒の場所に保存されています。標準ではlibrariesというフォルダに保存されています。各OSごとの保存場所は以下の通りです。
ちなみに僕はWindowsです。

| OS | 場所 |
| ---- | ---- |
| Windows | C:￥Users￥ユーザー名￥Documents￥Arduino |
| Mac | /Users/ユーザー名/Documents/Arduino |
| Linux | /home/ユーザー名/Arduino |

このディレクトリの保存先はArduinoIDEの [ファイル -> 基本設定 -> スケッチブックの場所]の参照ボタンを押して任意の場所に変更できます。

![](https://storage.googleapis.com/zenn-user-upload/a87e1590c89b-20240716.png)

このlibrariesフォルダにライブラリを追加することで手動でライブラリを入れることができます。

### ライブラリを追加する方法
まず、zipファイルをダウンロードします。
githubでは緑の $\textcolor{green}{code}$ というボタンをクリックして`Download ZIP` でダウンロードすることができます。

次にダウンロードしたファイルをlibrariesに追加します。
Arduino IDEのツール　-> ライブラリをインクルード　-> .ZIP形式のライブラリをインストール　でファイルが開くのでダウンロードしたzipファイルを選択します。
![](https://storage.googleapis.com/zenn-user-upload/4cbab84ead8f-20240730.png)

これでライブラリを追加することができます。

## ライブラリを探す
まずは使そうなライブラリを探しました。

一つ目に見つけたのが KotaMeiwa/nepils です。
@[card](https://github.com/KotaMeiwa/nepils)<br>

こちらは、かの有名な Adafruit_NeoPixel を継承しているらしく、今までそれを使ってきた人はすんなり使えると思います。が、`setPixelColor` は使えませんでした。なんでだろ？

2つ目に見つけたのは hideakitai/SpresenseNeoPixel です。
@[card](https://github.com/hideakitai/SpresenseNeoPixel)<br>

これも使ってみたのですが、やはり`setPixelColor`が使いたくてやめました。（笑）

## ライブラリを見つけた
いよいよ本題のライブラリに到達します。
先生にこのライブラリ使ってみなーてゆわれて見つけました。
それがこちらの lipoyang/SPI_NeoPixel です。
@[card](https://github.com/lipoyang/SPI_NeoPixel)<br>

## 虹色サンプルコード
試しに虹色に光らせてみます。
今回spresense本体と拡張ボードを使っています。

![](https://storage.googleapis.com/zenn-user-upload/160709457ca1-20240730.png)<br>

Neopixelの信号線は拡張ボードのD11ピン（右の方）に接続しています。
あとは5VとGNDで大丈夫です。
:::details 虹色に光らせるプログラム
```Cpp
#include <SPI.h>
#include <SPI_NeoPixel.h>
//https://github.com/lipoyang/SPI_NeoPixel

const uint16_t NUM_PIXELS = 42; //NeopixelのLED数
//spiの送信のピン　= 11

SPI_NeoPixel neopixel(NUM_PIXELS);

void setup() {

  Serial.begin(115200);
  delay(1000);

  neopixel.begin();

  Serial.println("Serial console start!");
}

void loop() {
  rainbowCycle(10, 128);  // 虹色サイクルを10回繰り返し、最大の明るさを128に設定する
  delay(500);
}

// 特定のLEDを点灯させる関数
void setPixelColor(int pixelIndex, uint32_t color) {
  if (pixelIndex >= 0 && pixelIndex < NUM_PIXELS) {
    neopixel.setPixelColor(pixelIndex, color);
    neopixel.show();
  }
}

// 虹色サイクルを実現する関数
void rainbowCycle(uint8_t wait, uint8_t maxBrightness) {
  uint16_t i, j;

  for (j = 0; j < 256 * 5; j++) {  // 5 cycles of all colors on wheel
    for (i = 0; i < neopixel.numPixels(); i++) {
      neopixel.setPixelColor(i, Wheel(((i * 256 / neopixel.numPixels()) + j) & 255, maxBrightness));
    }
    neopixel.show();
    delay(wait);
  }
}

// 虹色効果を生成する関数
uint32_t Wheel(byte WheelPos, uint8_t maxBrightness) {
  WheelPos = 255 - WheelPos;
  if (WheelPos < 85) {
    return neopixel.Color(((255 - WheelPos * 3) * maxBrightness) / 255, (WheelPos * 3 * maxBrightness) / 255, 0);
  }
  if (WheelPos < 170) {
    WheelPos -= 85;
    return neopixel.Color(0, ((WheelPos * 3 * maxBrightness) / 255), ((255 - WheelPos * 3 * maxBrightness) / 255));
  }
  WheelPos -= 170;
  return neopixel.Color(((WheelPos * 3 * maxBrightness) / 255), 0, ((255 - WheelPos * 3 * maxBrightness) / 255));
}
```
:::

はい、そうです。`setPixelColor`が使えるではありませんか！
やっと使えるところまできました！うれしかったです！
少しこのプログラム解説します。

まず、Neopixelを点灯する部分のプログラムですね。
```Cpp
neopixel.setPixelColor(光らせたいLEDのindex, 色(16進数))
```
これで任意の色で一つのLEDを光らせることができます。
ちなみに色は Adafruit とちがい１６進数になっています。
↓こちらのサイトで任意の色の１６進数が探せるとおもうのでぜひ活用してください。
https://www.colordic.org/#google_vignette

それでは虹色部分を解説していきます。
まず、虹色を生成している`Wheel`関数です。
```Cpp
uint32_t Wheel(byte WheelPos, uint8_t maxBrightness) {
    WheelPos = 255 - WheelPos;
```
引数にはWheelPosという値と最大の明るさを取っています。
WheelPosは0~255までの値で色の変化を示しています。<br>

その次の行で位置を反転させています。これで色の変化の方向を逆にしています。

次に色の計算部分です。
```Cpp
  if (WheelPos < 85) {
    return neopixel.Color(((255 - WheelPos * 3) * maxBrightness) / 255, (WheelPos * 3 * maxBrightness) / 255, 0);
  }
  if (WheelPos < 170) {
    WheelPos -= 85;
    return neopixel.Color(0, ((WheelPos * 3 * maxBrightness) / 255), ((255 - WheelPos * 3 * maxBrightness) / 255));
  }
  WheelPos -= 170;
  return neopixel.Color(((WheelPos * 3 * maxBrightness) / 255), 0, ((255 - WheelPos * 3 * maxBrightness) / 255));
}
```
最初の部分：WheelPosが0から85までの範囲では、赤から緑へ色が変化します。
中間部分：WheelPosが86から170までの範囲では、緑から青へ色が変化します。
最後の部分：WheelPosが171から255までの範囲では、青から赤へ色が変化します。

そこに`maxBrightness`をかけて明るさを調整しています。
:::message alert
明るさを255にするとかなりの電流が流れるので注意
:::

これで虹色が生成できたのでそれを適用する関数`rainbowCycle`について解説します。
```Cpp
void rainbowCycle(uint8_t wait, uint8_t maxBrightness) {
  uint16_t i, j;

  for (j = 0; j < 256 * 5; j++) {  // 5 cycles of all colors on wheel
    for (i = 0; i < neopixel.numPixels(); i++) {
      neopixel.setPixelColor(i, Wheel(((i * 256 / neopixel.numPixels()) + j) & 255, maxBrightness));
    }
    neopixel.show();
    delay(wait);
  }
}
```
引数は各更新の待機時間と最大の明るさです。
* ループ
外側の`forループ`は`j`というカウンタを使って、色の変化を繰り返します。この場合、`256 * 5`回繰り返します。つまり、虹色のエフェクトを5回繰り返すことになります。
内側の`forループ`は`i`というカウンタを使って、各LEDの色を設定します。
* 色の計算
Wheel関数を使って、各LEDの色を計算します。`((i * 256 / neopixel.numPixels()) + j) & 255`は、各LEDの位置に基づいて色を設定するための計算です。
これにより、LEDが連続して虹色に変化するようになります。

* 色の設定と表示
`neopixel.setPixelColor(i, Wheel(...))`で各LEDの色を設定します。
`neopixel.show()`で設定した色を表示します。
* 遅延
`delay(wait)`で指定した時間だけ待機します。短くすればするほど早く虹色が変わっていきます。

##  おわりに
なかなかSpresenseは癖が強いですね、、、（笑）
でも高性能にはちがいないのでこれからも遊んでみようと思います。
巡回君もアップグレードしていきます！
最後まで読んでいただきありがとうございました！