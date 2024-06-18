---
title: "esp32とneopixelを使った7セグ風BigClock"
emoji: kissing_face_with_closed_eyes
type: idea
topics: [esp32, neopixel, M5stack, 電子工作, Arduino]
published: false
---

# はじめに
今回はesp32とneopixelを使用して7セグ風のでっかい時計を作りました。

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

# 私が担当したところの大まかな手順(今回の記事)
1. esp32でntpサーバーに接続して正確な時間を取得する
2. esp32自身でカウントアップする
3. neopixelに色を付ける

# 担当以外の大まかな手順
1. 板にneopixelを貼る
2. はんだ付け
3. neopixelを任意の数字に光らせる（プログラム）
4. M5stackでサーバーを立てる
5. M5stackサーバーにアクセスして数字を見る
6. サーバーの数字に応じて時間を変更するか、ntpサーバーからの正確な時間を表示するか決める

url kairi

url hima

# 今回のすべての手順が入った全体のソースコードを置いておきます
長いのでプルダウンしてご覧ください
wifi設定は別ファイルをインクルードして使用しています。<br>
wifi設定のテンプレートはgitにおいてあるのでそちらを参考に書き換えてこのソースコードのおいてあるファイルの階層に一緒においておくとよいです。


:::details 全体のソースコード

```Cpp
#include <WiFi.h>
#include <Adafruit_NeoPixel.h>
#include "wifi_setting.h"
#include <HTTPClient.h>
#define PIN 27        //INが接続されているピンを指定
#define NUMPIXELS 74  //LEDの数を指定

//プロトタイプ宣言
void ShowTime(int hour, int minute);
void Clock();
void ntpacces();
void ClockOperation();

const int digitSegments[10][18] = {
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0 },  // 0
  { 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0 },  // 1
  { 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1 },  // 2
  { 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 },  // 3
  { 1, 1, 1, 0, 0, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1 },  // 4
  { 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 },  // 5
  { 1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 },  // 6
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0 },  // 7
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1 },  // 8
  { 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 1, 1 }   // 9
};

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);  //800kHzでNeoPixelを駆動

int flag;//:を点滅させるフラグ
int life;
int change;
unsigned long previousTime;
unsigned long ntptime;
unsigned long waitingtime;

struct tm timeInfo;  //時刻を格納するオブジェクト


void setup() {
  Serial.begin(115200);

  WiFi.mode(WIFI_STA);
  WiFi.disconnect();

  if (WiFi.begin(ssid, password) != WL_DISCONNECTED) {
    ESP.restart();
  }

  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
  }

  Serial.println("Connected to the WiFi network!");

  configTime(9 * 3600L, 0, "ntp.nict.jp", "time.google.com", "ntp.jst.mfeed.ad.jp");  //NTPの設定

  //wifi 取得周期
  //  prev = 0;         // 前回実行時刻を初期化
  //  interval = 100;   // 実行周期を設定


  pixels.begin();  //NeoPixelを開始
  int flag = 1;
  int life = 1;
  int change = 1;
  ntpacces();
}



void ShowTime(int hour, int minute) {
  /* ShowTime関数は、構造型(struct)の変数を受け取れるので、ShowTime関数の中に、LED画面に表示するコードを追加する。*/
  int time_4_digits = hour * 100 + minute;
  //Serial.println(time_4_digits);
  int s[4];
  for (int i = 3; i >= 0; i--) {
    s[i] = time_4_digits % 10;
    time_4_digits = (time_4_digits - s[i]) / 10;
  }

  // Serial.print(s[0]);
  // Serial.print(s[1]);
  // Serial.print(s[2]);
  // Serial.println(s[3]);


  pixels.clear();

  //  int digitSegments1 = digitSegments[s[i]][x] - 1;
  for (int i = 0; i < 4; i++) {     //各ケタ
    for (int j = 0; j < 18; j++) {  //1ケタ分のLED（18個）の表示
      int index = i * 18 + j;

      int huestart = 16363;//0;//始まりの色（書き換える）
      int huefin = 32766;//65536;終わりの色（書き換える)
      int hue = ((huefin - huestart) / 37) * index + huestart;//色の範囲を指定している(ここは書き換えない（0~65535))
      int sat = 255;
      int val = 200;
      
   
      if (i == 2) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));//hue(色),色彩、明るさ
      } else if (i == 3) {
        pixels.setPixelColor(index + 2, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
      } else {
        pixels.setPixelColor(index, pixels.ColorHSV(hue, sat, val * digitSegments[s[i]][j]));
      }
    }
  }
  

  pixels.setPixelColor(36, pixels.Color(flag*100, flag*100, flag*100));//1の時[:]点灯
  pixels.setPixelColor(37, pixels.Color(flag*100, flag*100, flag*100));


  pixels.show();  //LEDに色を反映
  
  
}


void loop() {
  if (life == 1){
    if(change == 0){
      ntpacces();
      int change =1;
    }
    Clock();//1秒カウントアップ
    ShowTime(timeInfo.tm_hour, timeInfo.tm_min);
    if(millis() - ntptime > 180*1000){//ntpをとった時間から180秒たったら
      ntpacces();
    }
    ClockOperation();
  }else if(life == 2){
    Clock();
    ShowTime(timeInfo.tm_hour, timeInfo.tm_min);
  }else if(life == 0){
    Clock();
    ClockOperation();
    ShowTime(timeInfo.tm_hour, timeInfo.tm_min);
  }
}


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

void ntpacces(){
  getLocalTime(&timeInfo);  //tmオブジェクトのtimeInfoに現在時刻を入れ込む
  Serial.println("ntpAcces!");
  ntptime = millis();
  Serial.print(timeInfo.tm_hour);
  Serial.print(timeInfo.tm_min);
  Serial.print(timeInfo.tm_sec);
}

void ClockOperation(){
  if (millis() - waitingtime >= 1000) {   //プログラムが経過した時間が1秒経ったら
    waitingtime = millis();   //基準時間に現在時間を代入
    if (WiFi.status() == WL_CONNECTED) {
      HTTPClient http;

      // URLの設定
      http.begin(serverUrl);

      // GETリクエストの送信
      int httpResponseCode = http.GET();

      if (httpResponseCode > 0) {
      //   // HTTPレスポンスコードを表示
        Serial.print("HTTP Response code: ");
        Serial.println(httpResponseCode);

      //   // ペイロードの取得と表示
        String payload = http.getString();
        Serial.println("Received payload:");
        Serial.println(payload);

        int dataInt = payload.toInt();
        int time[2];
        Serial.print(dataInt);
        
        if (dataInt == 9999){//ntpみにいく
          int life = 1;
        }else if(dataInt == 9998){//ntpみにいかない　カウントアップのみ
          int life = 2;
          int change =0;
        }else{
          for (int i = 1; i >= 0; i--){
          time[i] = dataInt % 100;
          dataInt = (dataInt - time[i]) / 100;
          }
          int life =0;
          timeInfo.tm_hour = time[0];
          timeInfo.tm_min = time[1];

          Serial.print(timeInfo.tm_hour);
          Serial.print(timeInfo.tm_min);
        }
        Serial.println("Data content copied to another variable:");
        //Serial.print(dataInt);

      } else {
        Serial.print("Error code: ");
        Serial.println(httpResponseCode);
      }

      // リクエストを終了
      http.end();
    } else {
      Serial.println("WiFi Disconnected");
    }
    // 過剰なリクエストを避けるために遅延を追加  // 必要に応じて遅延を調節
  }
  delay(100);
}

```
:::


# esp32でntpサーバーに接続して正確な時間を取得する
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

# esp32自身でカウントアップする
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

解説していきます。

```Cpp
if (millis() - previousTime >= 1000) {
  previousTime = millis();
```

<p>この部分は１秒に1回だけカウントアップするための条件を設定しています。
`millis()`はESP32に元々用意されている関数で、プログラムが開始されてから経過した時間（ミリ秒）を返します。
`previousTime`は私が作成した変数で、初期値は0です。
この条件文では、現在の時間から前回の時間 `previousTime` を引いた値が1000ミリ秒（1秒）以上であるかをチェックしています。
</p>

# neopixelに色を付ける
色を付けているのはneopixelのledを制御している関数の中で設定しています。

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
      
      // 色相、彩度、明度を設定します。
      int sat = 255;
      int val = 200;
      
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

## ポイントとなる箇所を詳しく解説します
* 取得してきた４桁の数字を一けたにするところ
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

* 

# これからの展望（つぎどんなことしたいか）