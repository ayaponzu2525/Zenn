---
title: "esp32とneopixelを使った7セグ風BigClock!"
emoji: kissing_face_with_closed_eyes
type: idea
topics: [esp32, neopixel, M5stack, 電子工作, Arduino]
published: false
---

# はじめに
こんにちは！ayaponzuです。
今回はクラスメイトと作業を分担して<br>
esp32とneopixelを使用して7セグ風のでっかい時計を作りました。
![](https://storage.googleapis.com/zenn-user-upload/8bc730651461-20240619.jpg)

# 全体のタスク
* 3分に１回、ntpサーバーから正確な時間を取得
* neopixelをはんだ付け
* neopixelに任意の数字を表示させる
* ntpサーバーから取得した時間からesp32自身でカウントアップ
* M5stackを用いてサーバーを立ててサーバーにある任意の数字の時間に変更できるようにする

# 材料
* esp32
* neopixel(74個のLED)
* 板（今回は百均の石膏ボード）
* M5stack(サーバーとして使用)

# システム構成図
![](https://storage.googleapis.com/zenn-user-upload/68d4b821338e-20240619.jpg)
ハード側の配線は上の画像のようになっていて左から順にバケツリレーのように信号が送られていきます。

![](https://storage.googleapis.com/zenn-user-upload/a63c38fee6d5-20240619.jpg)
こちらは正確な時間を表示するモードの構成図で、esp32は起動時にntpに行った後は、常にカウントアップして３分に一回ntpサーバーにアクセスして正確な時刻を取得しています。
それと同時にM5stackサーバーのほうにも１秒に１回見に行き、任意の時間がかかれていないか見に行っています。
何もない場合は9999が入っているのでそのまま正確な時間モードが続行されます。

![](https://storage.googleapis.com/zenn-user-upload/46f7e739d0d5-20240619.jpg)
こちらは任意の時間に変更してその時刻からカウントアップしていくモードです。
M5stackサーバーに任意の時間の数字が入っていたらその時間で上書きしそこからカウントアップします。
その際、ntpにいってしまうと正確な時間に戻ってしまうので一度任意の時間を確認したらM5stackサーバーは9998を保持することにしてもらい、ntpサーバーにはアクセスしないモードにしています。

:::message
ちなみにesp32の画像は私が3DCADで作成したesp32のモデルです。
:::
# 私が担当したところの大まかな手順(今回の記事)
1. esp32でntpサーバーに接続して正確な時間を取得する
2. esp32自身でカウントアップする
3. neopixelに色を付ける

# 担当以外の大まかな手順
1. 板にneopixelを貼る
2. はんだ付け
3. 3Dプリンターでカバーのデザイン
3. neopixelを任意の数字に光らせる（プログラム）
4. M5stackでサーバーを立てる
5. M5stackサーバーにアクセスして数字を見る
6. サーバーの数字に応じて時間を変更するか、ntpサーバーからの正確な時間を表示するか決める

クラスメイトそれぞれのZennの記事はこちらです。

https://zenn.dev/kai_z/articles/7segu_clock_zenn

https://zenn.dev/high_machine/articles/7segmentsclock3d

# 今回のすべての手順が入った全体のソースコード

@[card](https://github.com/ayaponzu2525/7seg_clock)

Arduino IDEで最初は開発してたのですが、書き込みが遅すぎてVscodeのplatformIOに移行したのでファイルがたくさんあります。<br>
gitのリンクに飛んだあと、src→main.cppがソースコードになります。<br>
その下のwifi_setting.h.templateがwifi設定のテンプレートファイルになります。

wifi設定はwifi_setting.hをインクルードして使用しています。<br>
wifi設定のテンプレートを参考に書き換えてこのソースコードのおいてあるファイルの階層に一緒においておくとよいです。

# 私が担当したところのプログラムを解説していきます
## esp32でntpサーバーに接続して正確な時間を取得する
ntpサーバーにアクセスする関数はこちらです
簡単にコメントで解説していきます。
```Cpp
void ntpaccess(){
  getLocalTime(&timeInfo);  
  //getLocalTimeでntpサーバーにアクセスし、tmオブジェクトのtimeInfoに現在時刻を入れ込む


  Serial.println("ntpaccess!");//シリアルモニター確認用
  ntptime = millis();//3分に１回にするための秒数保存用変数ntpTime

  Serial.print(timeInfo.tm_hour);
  Serial.print(timeInfo.tm_min);
  Serial.print(timeInfo.tm_sec);
  //それぞれhour,min,secに２桁ずつ時間が格納される
}
```

## esp32自身でカウントアップする
カウントアップする関数はこちらです

```Cpp
void Clock(){
  if (millis() - previousTime >= 1000) {   //プログラムが経過した時間が1秒経ったら
    previousTime = millis();   //基準時間に現在時間を代入
    flag = flag ^1; //1秒ごとに点灯
    timeInfo.tm_sec+=1; //1秒カウントアップ
    if(timeInfo.tm_sec == 60){
      timeInfo.tm_sec = 0;
      timeInfo.tm_min += 1;
      if(timeInfo.tm_min == 60){
        timeInfo.tm_min = 0;
        timeInfo.tm_hour += 1;
        if (timeInfo.tm_hour == 24){
          timeInfo.tm_hour = 0;
        }
      }
    }
  }
}
```

ポイントとなる箇所を解説していきます。

```Cpp
if (millis() - previousTime >= 1000) {
  previousTime = millis();
```

この部分は１秒に1回だけカウントアップするための条件を設定しています。
`millis()`はESP32に元々用意されている関数で、プログラムが開始されてから経過した時間（ミリ秒）を返します。
`previousTime`は私が作成した変数で、初期値は0です。
この条件文では、現在の時間から前回の時間 `previousTime` を引いた値が1000ミリ秒（1秒）以上であるかをチェックしています。

## neopixelに色を付ける
色を付けているのはneopixelのLEDを制御している関数の中で設定しています。

:::details neopixelのLEDを制御している関数
```Cpp
void ShowTime(int hour, int minute) {
  // この関数は、時間と分をLEDに表示するためのものです。
  // 入力として時間（hour）と分（minute）を受け取ります。
  
  int time_4_digits = hour * 100 + minute;
  // 時間と分を4桁の整数に変換します。
  // 例: 13時45分の場合、1345になります。

  int s[4];
  for (int i = 3; i >= 0; i--) {
    s[i] = time_4_digits % 10;
    time_4_digits = (time_4_digits - s[i]) / 10;
  }
  // 4桁の数字をそれぞれの桁に分解して配列sに保存します。
  // 例: 1345 -> s = [1, 3, 4, 5]

  pixels.clear();
  // すべてのピクセル（LED）をクリア（消灯）します。

  for (int i = 0; i < 4; i++) {     // 各桁の数字についてループします。
    for (int j = 0; j < 18; j++) {  // 各桁の数字を表示するための18個のLEDをループします。
      int index = i * 18 + j;

      // 色の範囲を指定して設定します。
      int huestart = 16363; // 開始の色
      int huefin = 32766;   // 終了の色
      int hue = ((huefin - huestart) / 37) * index + huestart;
      
      // 色相、彩度、明度を設定
      int sat = 255;
      //彩度を255に設定
      int val = 200;
      //明度を200に設定
      
      if (i == 2) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
        // 3番目の桁のLEDを設定します（オフセット+2）。
      } else if (i == 3) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
        // 4番目の桁のLEDを設定します（オフセット+2）。
      } else {
        pixels.setPixelColor(index, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
        // 1番目と2番目の桁のLEDを設定します。
      }
    }
  }

  // 時刻のコロン（:）を設定します。
  pixels.setPixelColor(36, pixels.Color(flag*100, flag*100, flag*100));
  pixels.setPixelColor(37, pixels.Color(flag*100, flag*100, flag*100));
  

  pixels.show();// 設定した色をLEDに反映させます。
  
}

```
:::
### 詳しく解説します
#### 取得してきた４桁の数字を一けたずつに分解するところ
```Cpp
s[i] = time_4_digits % 10;
```
この部分は`time_4_digits`の一番右の桁を取り出して、配列sの対応する位置に格納します。
`% 10`は、10で割った余りを求める演算子で、これにより一番右の桁を取得できます。

例：`time_4_digits`が1345の場合、最初の`i=3`のときは`1345 % 10 = 5`、すなわち5が取得され、`s[3]`に格納されます。

```Cpp
time_4_digits = (time_4_digits - s[i]) / 10;
```
この部分は、`time_4_digits`から取得した桁`s[i]`を引き、それを10で割って次の桁に進む準備をします。

例：`time_4_digitsが1345`で`s[3]`に5を格納した後、`(1345 - 5) / 10 = 1340 / 10 = 134`となり、次のループでは`time_4_digits`は134となります。

このようにして、`time_4_digits`の4桁の数字がそれぞれ`s[0]`から`s[3]`に分解されて格納されます。これにより、後続の処理で各桁を個別に操作することが可能になります。

#### 色を変えるところ
色を表す書式はいろいろありますが今回はHSVを使って色をグラデーションにしていきたいと思います。
HSVとは下の画像のようなもので色相環をつかって角度で色彩を表すものです。
<!--hueの画像-->
![](https://storage.googleapis.com/zenn-user-upload/140f63a08379-20240619.png)

色相、彩度、明度を指定して使います。
始まりの色の色相は0,終わりの色は65535になっています。

```Cpp
int huestart = 0; // 開始の色
int huefin = 65535;   // 終了の色
int hue = ((huefin - huestart) / 37) * index + huestart; // 色の範囲を指定して設定します
```
この部分で色を指定しています
`huestart`：色の開始点を指定します。
`huefin`：色の終了点を指定します。
これら二つを変えることでどこの部分をつかってグラデーションにするか指定できます。

```Cpp
hue = ((huefin - huestart) / 37) * index + huestart;
```
この部分は`huefin`と`huestart`の差を37(neopixelは全部で74個なのでその半分）で割り、それに`index`を掛けて`huestart`を足すことで、指定された範囲内でインデックスに応じた色相を計算します。
`index`は現在のLEDの位置を示す値で、0から徐々に増加します。
例えば、`index`が0のとき、hueは`huestart`（0）となり、`index`が最大のとき、`hue`は`huefin`（65535）に近い値になります。

```Cpp
int sat = 255;
      
int val = 200;
```
* `sat` 彩度を255に設定しています。彩度は0から255の範囲で指定され、255は最も鮮やかな色を意味します。
* `val` 明度を200に設定しています。明度も0から255の範囲で指定され、200はかなり明るい色を意味します。
:::message
valをマックスにするとかなりの電流が流れるので注意
:::

```Cpp
if (i == 2) {
  pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
} else if (i == 3) {
  pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
} else {
  pixels.setPixelColor(index, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
}
```
* `pixels.setPixelColor`

この関数は、指定されたインデックスのLEDに対して色を設定します。
* `pixels.ColorHSV`

色をHSV（色相、彩度、明度）で指定します。
`hue`：計算した色相。
`sat`：彩度。
* `val * digitSegments[s[i]][j]`：明度。
`digitSegments[s[i][j]]`が1の場合は設定した明度、0の場合は消灯（0）になります。


* `if (i == 2)およびelse if (i == 3)`(３桁目と４桁目)の場合は、インデックスに+2のオフセットを加えています。
これは真ん中のコロンが２個分あるためです。

```Cpp
pixels.setPixelColor(36, pixels.Color(flag*100, flag*100, flag*100));
pixels.setPixelColor(37, pixels.Color(flag*100, flag*100, flag*100));
```
* コロンの設定部分です。
`flag`が1なら点灯し、0なら消灯します。
`flag`は前述したカウントアップする関数の中で１秒ごとに1,0と反転させているのでそれにHSVのすべての値を100にしてかけています。
# これからの展望（つぎどんなことしたいか）
今は時間の表示しか機能として実装していませんが、スピーカーをつけてチャイムを鳴らしたり、ストップウォッチやタイマーなどの機能も実装していきたいと思っています。