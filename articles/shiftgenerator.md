---
title: "バイト先のシフト作成業務を改善したい話"
emoji: kissing_face_with_closed_eyes
type: tech
topics: [Python, Django, Webアプリ]
published: false
---

## はじめに
皆さんこんにちは！ayaponzuです！
今、私はスーパーでアルバイトをしています。（現在3年目）

半月の自由シフト制なので半月ごとに店長がシフト作成しなければならないのですが、毎回、店長が「あ～またつくらないと」と言っているのをずっと見てきました。
そこで私は IT の力を使って効率化できないかなと考えました。

今回考慮するのは正社員以外の従業員のシフトとします。<br>
(正社員はほぼ曜日で決まっているためそんなに悩まないから。)


## シフト作成の流れと考慮すべきポイント

### **現在のシフト作成の流れ**
### 例：12月後半（12月16日~31日）のシフトを作る場合

1. **12月10日まで**に従業員にシフト希望を出してもらう（紙媒体）。  
2. **Excelにシフト希望を転記**する（手入力）。  
3. **時間帯やスキルでの上限・下限人数を満たすように調整**する。  
4. **印刷**する（12月15日までには完成させる）。  

---

### **シフト作成における考慮すべきポイント**

### **1. 最大人数・最少人数について**

- 時間帯ごとに人数の上限・下限を考慮します。
- 基本の時間帯区分：  
  **朝（9:00~14:00）** / **昼（14:00~17:00）** / **夜（17:00~20:00）**  

| **平日**   | 朝：最少5人 / 最大10人<br>昼：最少5人 / 最大10人<br>夜：最少3人 / 最大10人 |
|------------|-------------------------------------------------------------------------|
| **土日祝** | 朝：最少6人 / 最大10人<br>昼：最少6人 / 最大10人<br>夜：最少4人 / 最大10人 |

---

### **2. レジ担当の割り振り**

- 基本の交代時間：**朝（9:00~14:00）** / **昼（14:00~17:00）** / **夜（17:00~20:00）**
- **レジ数：4つ**  
  - **1レジ**：土日祝のみ  
  - **2・3レジ**：常時稼働  
  - **1・4レジ**：混雑時のみ（ヘルプ）

| **平日**   | 朝：3人<br>昼：3人<br>夜：3人 |
|------------|-----------------------------|
| **土日祝** | 朝：4人<br>昼：4人<br>夜：4人 |

---

### **3. スキルについて**

- **品出し**：研修中を除き、全員対応可能。  
- **レジ**：レジスキルがある人の中から担当を選定。  
- **冷蔵作業**：夜の時間帯に1人配置（**1レジ・4レジの人が兼任可能**）。  

---

### **4. 休憩時間のルール**

| **勤務時間**      | **休憩時間**   |
|-------------------|---------------|
| **4時間以下**     | 休憩なし      |
| **5時間以上6時間未満** | 15分          |
| **6時間以上7時間未満** | 30分          |
| **7時間以上**     | 45分          |

---

### **5. 連続勤務の制限**

- **連勤は最大3日まで**が理想。（店長の方針）

---

### **プログラムで考慮しておきたいこと**

### **1. 休憩回しの自動配置**

- レジ担当者が休憩に行く際に、**品出し担当者を自動で配置**。  
- 例：休憩に行った時間帯に空いたレジを埋めるよう調整。

---

### **柔軟な対応の方針**

- シフト希望がそもそも必要人数に足りていない場合は、シフト希望を**優先**する。
- **相談なしにシフトを勝手に入れることは避ける**。（当たり前）
- 必要人数が不足している場合、アプリでは**「警告を出す」機能を実装**予定。


## どうやって作っていくか
これは長期インターンに行かせていただいた際に、担当の方に一緒に考えていただきました。（感謝です。）

###  Webアプリかネイティブアプリか

シフト生成アプリを作る際、**Webアプリ**を選んだ理由について書きます。<br>結論としては、Webアプリにしました。



### なぜシフト生成アプリをWebアプリとして選んだのか？

シフト生成アプリを開発する際に、**Webアプリ**を選んだ理由は主に4つあります。

---

### **Webアプリを選んだ理由（メリット）**

#### 1. **どこからでもアクセスできる**
Webアプリの最大の利点は、**インターネットさえあれば、どのデバイスからでもアクセスできる**ことです。これによって、**スタッフが自宅やスマートフォンからシフト希望を簡単に登録**できたり、管理者が外出先からでもシフトを確認したり調整したりできるので、すごく便利だなと感じました。

#### 2. **シフト希望のオンライン登録**
従来の方法（紙やExcelでのシフト提出）だと、スタッフがバイト先に来て希望を出す必要がありましたが、Webアプリなら**24時間いつでもどこでもシフト希望を提出**できます。さらに、**リアルタイムで希望を登録できる**ので、転記ミスなどの人的ミスも防げ、管理者の手間も大幅に削減できます。

#### 3. **インストール不要で使いやすい**
Webアプリは、ユーザーが特別なソフトをインストールする必要がありません。**ブラウザがあればすぐに使える**ので、スタッフ全員が手軽にアクセスできます。しかも、アプリのアップデートもサーバー側で一元的に行えるので、スタッフは常に最新機能を使うことができます。

#### 4. **複数ユーザーが同時に利用可能**
Webアプリでは、複数のスタッフが同時に希望を登録したり、管理者がシフトを調整できるため、**チーム全体でリアルタイムに利用可能**です。みんなが同時にアクセスしても問題なく使えるのは大きなポイントです。

---
### **Webアプリの課題（デメリット）**

#### 1. **インターネット接続が必要**
Webアプリは、インターネット接続が必須です。もしインターネットが利用できない環境だと動作しません。  
$\textcolor{lightblue}{まぁ、インターネットが不安定な時は、紙で頑張ろう！って感じですね（笑）}$

#### 2. **セキュリティ対策が必須**
Webアプリでは、シフトデータやスタッフの個人情報をサーバーに保存するため、**セキュリティ対策**が必要です。暗号化や認証システムを導入すればリスクを軽減できますが、**セキュリティ面に関する知識がないと正直不安**ですよね。  

$\textcolor{lightblue}{ここが一番悩んだところです。でも、やると決めたからには、しっかり学んで対策を講じていこうと思っています。}$

#### 3. **運用コストが発生する**
サーバーを運営するためには、サーバー費用やドメイン費用が定期的に発生します。さらに、**システムに障害が発生した時の対応**なども考慮しないといけません。  
$\textcolor{lightblue}{ただ、PythonAnywhereという無料サービスがあり、3か月に一度ボタンを押すだけでサーバーが無料で使えるので、しばらくは問題なし！本格的に作り込んだ時にまた考えます。}$


#### 4. **動作速度の制限**
通信環境やサーバーの負荷によっては、ローカルアプリに比べて**動作が遅くなる可能性**もあります。データ量が増えてきたときにどうなるかはちょっと心配です。

$\textcolor{lightblue}{今は1店舗のみなので、そこまで気にしていません。もし大規模になったらその時に最適化を考えればいいかなと思っています。}$

---


### Webアプリとローカルアプリで悩んだポイント

実は、最初は**ネイティブアプリ**にする案もありました。理由は単純で、**「セキュリティ面もあまり考慮しなくていいし、インターネットがなくても使える」**という点が魅力的だったからです。

特に、シフト生成のようにデータ量が多くなりそうな場合、「動作が遅くなったらどうしよう？」という不安がありました。でも、ローカルアプリにはいくつかの課題がありました。

---

### **ローカルアプリの課題**
- **デバイスごとにインストールが必要**  
  スタッフ全員のデバイスにアプリをインストールするのは手間がかかりますし、アップデートのたびにまた全員に対応してもらう必要があるのが大変そうでした。

- **データの共有が難しい**  
  シフト希望や生成結果を共有するには、ファイルのやり取りが必要になりそうで、リアルタイム性が失われる点が気になりました。

---

最終的には、**「どこからでもアクセスできて、スタッフ全員が簡単に使える」**というWebアプリの利便性が勝ち、こちらを選ぶことにしました。やっぱり、シフト希望をオンラインで簡単に登録できるのは大きなメリットですよね！



