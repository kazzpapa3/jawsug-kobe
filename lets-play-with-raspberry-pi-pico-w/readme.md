# raspberry pi pico w を使ってみる

## 想定する対象

多分、超初心者向け

## 想定するゴール

| #      | ゴール                                                       |
| ------ | ------------------------------------------------------------ |
| 第一回 | raspberry pi pico w の設定方法を理解する。<br />Wi-Fi 設定を投入し接続することができる。<br />raspberry pi pico w の内部センサーで取得可能な情報を IoT Core に送信できる。 |

## 準備いただく物

### Raspberry Pi Pico W

Raspberry Pi Pico は Linux OS をインストールできる Raspberry Pi とは異なり RP2040マイコンを搭載した開発基板で、C/C++ SDK、もしくは公式に提供されているMicroPython インタプリタを使った開発が可能です。

無線チップの有無の違いにより、異なる型番が用意されていますが、無線LAN を利用した通信を用いて容易に成功体験を得やすい Raspberry Pi Pico W を利用したいと考えています。

なお、ピンヘッダ実装済みの Raspberry Pi Pico WH と、自身でのピンヘッダの用意やはんだ付けが必要な Raspberry Pi Pico W の2種類が用意されていますが、ご自身での実装ができる方は Raspberry Pi Pico W でも構いません。

#### スイッチサイエンスでの参考価格

表内の下２行が無線チップが実装されている品番

| 商品名                                                       | 価格（税込） | 在庫（2024/8/1時点） |
| ------------------------------------------------------------ | -----------: | -------------------- |
| [Raspberry Pi Pico](https://ssci.to/6900)                    |          836 | 100個以上            |
| [Raspberry Pi Pico（ピンヘッダ実装済）](https://ssci.to/7412) |        1,100 | 販売終了してた       |
| [Raspberry Pi Pico W](https://ssci.to/8171)                  |        1,353 | 100個以上            |
| [Raspberry Pi Pico WH](http://ssci.to/8172)                  |        1,540 | 100個以上            |

### ノートパソコン

Raspberry Pi Pico W と USB を用いた接続をし、MicroPython でのプログラムの作成、送信を行います。

MicroPython の開発 IDE である Thonny をインストール可能な環境、USB ポートが搭載されている必要があります。

#### Thonny について

公式ページ：[thonny.org](https://thonny.org/)

### USB ケーブル

Raspberry Pi Pico W 側は Micro USB 2.0 Type-B となっているため、片側の端は Micro USB 2.0 Type-B である必要があります。

### AWS アカウント

ご自身で自由に使える AWS アカウントをご用意ください。

また、お持ちでなくこれからご用意される方、その他不正利用や高額使用を避けるための推奨設定をご確認になりたい方は [付録](../../appendix/) を参照ください。
