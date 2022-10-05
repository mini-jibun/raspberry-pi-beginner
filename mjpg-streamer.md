# mjpg-streamerを動かす

`mjpg-streamer`はバイナリでの提供はありません｡ソースコードを入手し､自分でビルドする必要があります｡

以下に手順を示します｡

## 前提条件

`Raspberry Pi`にsshでログインし､以下のコマンドを実行します｡

```bash
sudo apt install build-essential cmake libjpeg62-turbo-dev
```

## ソースコードの入手

以下を実行し､GitHubからソースコードを入手します｡

```bash
git clone https://github.com/mini-jibun/mjpg-streamer
```

## ビルド

以下を実行します｡

```bash
cd mjpg-streamer/mjpg-streamer-experimental
make -j$(nproc)
```

`$()`は､コマンドの標準出力を文字列として展開する役割があり､`nproc`コマンドは､CPUのスレッド数(コア数)を表示するコマンドです｡

`make`に渡している`-j`オプションは､並列実行コマンド数を指定します｡すなわち､`make -j$(nproc)`は`make -j4`に展開され､CPUリソースを最大限活用しコンパイルします｡

## インストール

以下を実行します｡`/usr/local`以下に必要なファイル群がコピーされます｡

```bash
sudo make install
```

`mjpg_streamer`を実行するユーザを作成し､`mjpg_streamer@.service`という`systemd`ユニットファイルを`/etc/systemd/system`にコピーします｡

```bash
sudo ./postinstall.sh
```

## 起動

### 使用するカメラのデバイスファイルを調べる

起動にあたって､使用したいカメラのデバイスファイルを知る必要があります｡

`v4l2-ctl`コマンドが実行できない場合､以下で`v4l-utils`をインストールします｡

```bash
sudo apt install v4l-utils
```

以下のコマンドを実行することで､接続されているカメラ(映像デバイス)がすべて表示されます｡また､そのカメラが対応している`Pixel Format`､解像度､フレームレートが表示されます｡

```bash
v4l2-ctl --all --list-formats-ext
```

実際にコマンドを実行し､標準出力された内容を示します:

<details>
<summary>上記コマンドの標準出力</summary>

```text
Driver Info:
        Driver name      : uvcvideo
        Card type        : BUFFALO BSWHD06M USB Camera
:
        Bus info         : usb-0000:01:00.0-1.4
        Driver version   : 5.15.61
        Capabilities     : 0x84a00001
                Video Capture
                Metadata Capture
                Streaming
                Extended Pix Format
                Device Capabilities
        Device Caps      : 0x04200001
                Video Capture
                Streaming
                Extended Pix Format
Media Driver Info:
        Driver name      : uvcvideo
        Model            : BUFFALO BSWHD06M USB Camera
:
        Serial           :
        Bus info         : usb-0000:01:00.0-1.4
        Media version    : 5.15.61
        Hardware revision: 0x00000103 (259)
        Driver version   : 5.15.61
Interface Info:
        ID               : 0x03000002
        Type             : V4L Video
Entity Info:
        ID               : 0x00000001 (1)
        Name             : BUFFALO BSWHD06M USB Camera
:
        Function         : V4L2 I/O
        Flags         : default
        Pad 0x01000007   : 0: Sink
          Link 0x02000010: from remote pad 0x100000a of entity 'Processing 2': Data, Enabled, Immutable
Priority: 2
Video input : 0 (Camera 1: ok)
Format Video Capture:
        Width/Height      : 1280/720
        Pixel Format      : 'MJPG' (Motion-JPEG)
        Field             : None
        Bytes per Line    : 0
        Size Image        : 2764800
        Colorspace        : sRGB
        Transfer Function : Rec. 709
        YCbCr/HSV Encoding: ITU-R 601
        Quantization      : Default (maps to Full Range)
        Flags             :
Crop Capability Video Capture:
        Bounds      : Left 0, Top 0, Width 1280, Height 720
        Default     : Left 0, Top 0, Width 1280, Height 720
        Pixel Aspect: 1/1
Selection Video Capture: crop_default, Left 0, Top 0, Width 1280, Height 720, Flags:
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 1280, Height 720, Flags:
Streaming Parameters Video Capture:
        Capabilities     : timeperframe
        Frames per second: 30.000 (30/1)
        Read buffers     : 0
                     brightness 0x00980900 (int)    : min=-255 max=255 step=1 default=-4 value=-4
                       contrast 0x00980901 (int)    : min=0 max=30 step=1 default=15 value=15
                     saturation 0x00980902 (int)    : min=0 max=127 step=1 default=32 value=32
                            hue 0x00980903 (int)    : min=-16000 max=16000 step=1 default=0 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=20 max=250 step=1 default=84 value=84
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=1 value=1
                                0: Disabled
                                1: 50 Hz
                                2: 60 Hz
      white_balance_temperature 0x0098091a (int)    : min=2500 max=7000 step=1 default=5000 value=5000 flags=inactive
                      sharpness 0x0098091b (int)    : min=0 max=64 step=1 default=0 value=0
         backlight_compensation 0x0098091c (int)    : min=0 max=2 step=1 default=1 value=1
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'YUYV' (YUYV 4:2:2)
                Size: Discrete 1280x720
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 800x600
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 640x480
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 640x360
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 352x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 320x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 176x144
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 160x120
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
        [1]: 'MJPG' (Motion-JPEG, compressed)
                Size: Discrete 1280x960
                        Interval: Discrete 0.083s (12.000 fps)
                Size: Discrete 1280x720
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.042s (24.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.062s (16.000 fps)
                Size: Discrete 800x600
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.042s (24.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.062s (16.000 fps)
                Size: Discrete 640x480
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.042s (24.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.062s (16.000 fps)
                Size: Discrete 640x360
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 352x288
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.042s (24.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 320x240
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 176x144
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.125s (8.000 fps)
                Size: Discrete 160x120
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.042s (24.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.062s (16.000 fps)
```

</details>

ここでは､`BUFFALO BSWHD06M USB Camera`を利用したいので`Video input : 0`ということになります｡すなわち､`/dev/video0`がデバイスファイルです｡

`mjpg-streamer`は名前の通り`MJPG`のみ対応しています｡
その中での最高設定は`1280x720`､`30fps`ですので､そのように設定します｡

(この設定は既に`mjpg_streamer@.service`に書き込んであります｡)

## 起動 & 自動起動化

以下を実行します｡これで､`Raspberry Pi`が起動すると自動的に`mjpg-streamer`が実行されます｡`@`の後ろは上で調べたデバイスファイルの`/dev/`を取り除いたものを指定します｡

```bash
sudo systemctl --now enable mjpg_streamer@video0
```
