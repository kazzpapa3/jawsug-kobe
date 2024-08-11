# Raspberry Pi Pico W を使ってみる：第一回

## 今回のゴール

Raspberry Pi Pico W を Wi-Fi に接続し、AWS IoT Core を利用して AWS にラズパイ内部で収集している温度を送信する。

## ハンズオン手順

### 1. パソコンとラズパイの接続

#### Thonny のインストール

1. 公式サイトへ接続し、お使いの PC の OS、アーキテクチャに合わせたインストーラをダウンロードします。
2. ダウンロードしたインストーラを起動し、ダイアログに沿ってインストールを進めます。

##### CLI （パッケージ管理システム）でのインストール

Mac をお使いの場合は `brew install thonny` で、Windows をお使いの場合は、winget の場合は `winget install -e --id AivarAnnamaa.Thonny` で、chocolatey をお使いの場合は `choco install thonny` でのインストールも可能です。

#### 2. MicroPython ファームウェアのインストール

1. [MicroPython](https://www.raspberrypi.com/documentation/microcontrollers/micropython.html) ページの中ほどの「Download the correct MicroPython UF2 file for your board:」部分にRaspberry Pi Pico W対象の Raspberry Pi Pico ごとのダウンロードリンクがあります。今回のハンズオンでは、Raspberry Pi Pico W(H) をお使いいただいている想定なので、「[Raspberry Pi Pico W](https://micropython.org/download/rp2-pico-w/rp2-pico-w-latest.uf2) with Wi-Fi and Bluetooth LE support」よりダウンロードします。（pr2-pico-xxxxxxxx-vx.xx.x.uf2 というような .uf2 ファイルとしてダウンロードされます）
2. Raspberry Pi Pico W(H) と PC を接続する。  
   Raspberry Pi Pico W(H) の白いボタン（BOOTSEL）を押しながら USB ケーブルで PC と Raspberry Pi Pico W(H) を接続します。
3. PC 側で外付けデバイス RPIｰRP2 として認識されますので、エクスプローラ（Windows）、ファインダー（Mac）で操作し、＜1＞でダウンロードした .uf2 ファイルを RPIｰRP2 にドラッグアンドドロップし コピーします。

