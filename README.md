# multi-reader by kuromunori
Aliceリスペクトの1記号が1つのコマンドになっている2次元言語です。

言語の仕様は以下に

## 大きな特徴
* 複数の命令ポインタが同時にソースコードを解釈&実行
* 命令ポインタ間には上下関係があり、読む箇所が衝突した場合、負けた命令ポインタは初期位置に戻される
* 入力コマンドは存在せず、命令ポインタが初期位置に戻された際に1文字ずつ読み取られる

## ソースコード
ソースコードは読み込まれたのち、それを包括する最小のN×Mの長方形として内部に格納されます。  
これが命令ポインタが移動できる領域となります。  
領域外へ命令ポインタが移動しようとするとRunTimeErrorとなり処理を停止します。

## 命令ポインタ
### 基本事項
以下の文字が命令ポインタとして扱われます。

Cmd | 説明
--- | -------- 
`0`～`9`|命令ポインタ(入力機能なし) 
`a`～`z`|命令ポインタ  (入力機能あり)

命令ポインタ間の基本の強さは  
0<1<……<9<a<b<……<z  
です。  
各ポインタ間の基本強さは1ずつ異なります。

つまり、ポインタ0の基本強さは0、ポインタaの基本強さは10、ポインタzの基本強さは35です。  
これらの文字が複数ソースコードに存在することは許されません。(コンパイルエラーとなります)  
命令ポインタはソースコード内の初期位置からまず右へと移動していきます。  
移動順は基本の強さの弱い方から1マスずつ進みます。  
各ターンにおいて、まず移動を行い、命令ポインタの衝突があった場合は衝突処理をし、そのあとに移動後のマスのコマンドを実行します。(後述するコマンド`#`のみそのマスから移動する時に反映されます)  
また、各命令ポインタは1つだけ値を格納することが出来ます。  

### 命令ポインタの衝突
命令ポインタが移動した後のマスに他の命令ポインタがいた場合、コマンドを実行する前に衝突処理が発生します。  
衝突処理ではぶつかった命令ポインタのうち、強さが弱い方のポインタが初期位置へと飛ばされます。  
また、以下のコマンドでポインタの強さを増減できます。

Cmd | 説明
--- | -------- 
`U`|ポインタの強さを1増やす。(Up)  
`D`|ポインタの強さを1減らす。(Down)  

この増減した強さは衝突判定の時のみ有効で命令ポインタの実行順は基本強さで不変です。  
例えばポインタbがUを2回通過すると強さは(11+2)で13となります。  
強化後の強さが同じポインタが衝突した場合、基本強さの弱い方のポインタが初期位置へ飛ばされます。  
ex)Uを2回通過したポインタaとポインタcではaが初期位置へ飛ばされる。

初期位置へ飛ばされる際、命令ポインタの移動方向、強化値は保存されます。  
ポインタ0~9については格納された値も保存されます。  
### 命令ポインタに格納される値  
各命令ポインタは1つだけ値を持つことが出来ます。  
これがそのまま変数となります。  
ポインタ0～9の持つ変数の初期値は0です。  
ポインタa～zの持つ変数の初期値は標準入力の1文字の文字コードが順に格納されます。  
ex)ポインタ0,3,a,c,fがおり、標準入力が"Hello"のとき、

|ポインタ| 0 | 3 | a | c | f |
|:------|--:|:-:|:-:|:-:|:-:|
| 初期値 | 0 | 0 | 72|101| 96|

となる
また、残りの入力はポインタa～zが他のポインタと衝突し、初期位置へ飛ばされるたびに飛ばされたポインタに1文字ずつ格納されていきます。
(元々ポインタa～zに格納されていた値は上書きされ破棄されます。)
標準入力が全て格納された後は-1が格納されます。
これ以外に標準入力を受け取る方法はありません。
また、以下のコマンドによって自身の現在の強さを値として格納することが出来ます。

Cmd | 説明
--- | -------- 
`G`|ポインタの現在の強さを値として格納する。(Get)

ex)ポインタaがコマンドDを1度通過したのちコマンドGを通過すると命令ポインタaの持つ変数には9(=10-1)が格納される。

## 移動コマンド
multi-readersには命令ポインタの移動方向を変えるコマンドが多数存在します。
また、縦横方向に移動している時と斜め方向に移動している時で意味の変わるコマンドが存在します。  
### ミラー
縦横と斜めを切り替えるコマンドです。

Cmd | 説明
--- | -------- 
`\` |　命令ポインタの移動方向を縦横と斜めの間で切り替えます
`/` |　命令ポインタの移動方向を縦横と斜めの間で切り替えます

それぞれの移動方向の切り替え方は下図を参照してください。
[![Movement through mirrors][mirrors]][mirrors]  

### 壁

Cmd | 説明
--- | -------- 
`_` | y方向の移動向きを反転させます
`\|` | x方向の移動向きを反転させます

それぞれ、指示のない方向は移動向きを変えません
(斜め移動をしている場合は斜め移動のままです)
### 矢印

Cmd | 説明
--- | -------- 
`^` | 垂直に上へ移動します
`V` | 垂直に下へ移動します
`>` | 水平に右へ移動します
`<` | 水平に左へ移動します

(斜め移動をしていても水平垂直移動へ切り替わります)

## 四則演算コマンド
四則演算コマンドは加減乗除に加え、あまりの計算が用意されています。
これらのコマンドは縦横にとおるか、斜めに通るかで意味が変わります。
これらのコマンドはそれぞれ内部変数を持っています。
内部変数の初期値は0です。

Cmd | 斜め|縦横
--- | ------| ------
`+`|ポインタの格納する変数を内部変数に代入|ポインタの格納する変数+=内部変数
`-`|ポインタの格納する変数を内部変数に代入|ポインタの格納する変数-=内部変数
`*`|ポインタの格納する変数を内部変数に代入|ポインタの格納する変数×=内部変数
`:`|ポインタの格納する変数を内部変数に代入|ポインタの格納する変数÷=内部変数
`%`|ポインタの格納する変数を内部変数に代入|ポインタの格納する変数%=内部変数

## 文字→数値変換コマンド
ポインタの格納する変数が'0'～'9'のいずれかの文字の文字コードであった場合、それを数値へ変換します。
それ以外の文字コードを表す値であった場合は何もしません。

Cmd | 説明
--- | ------
`N`|ポインタの格納する文字'0'~'9'を対応する数字へ変換して格納


##　出力コマンド
標準出力をするためのコマンドを4つ用意しています。

Cmd | 説明
--- | ------
`O`|ポインタの格納する変数を数値として出力
`C`|ポインタの格納する変数を文字として出力
`S`|半角スペースを出力
`E`|改行を出力

## 停止コマンド
このコマンドが命令ポインタのどれか一つに呼ばれたとき、プログラムを終了します。
停止コマンドをふまない限りプログラムが動き続けるので注意してください

Cmd | 説明
--- | -----
`@`|どれか一つの命令ポインタに呼び出されたとき、プログラムを終了する

## 条件分岐コマンド
このコマンドのマスから移動する際、ポインタの格納している値が0以上であれば進行方向のマスを1つ飛ばして移動する。  
0未満であれば通常通り移動する。

Cmd | 説明
--- | -----
`#`|ポインタの格納する値が0以上であればマスを1つ飛ばして移動。

### DEBUGモードについて
ソースコードをコンパイルする際に、

 kusogengo -d hoge.kuso

とすることで、DEBUGモードでプログラムを実行することが出来ます。
このモードでは各命令ポインタが1ステップずつ移動した後の盤面の状態を逐次表示してくれます。
デバッグにお役立てください


  [mirrors]: https://i.stack.imgur.com/YHx0d.png
  
