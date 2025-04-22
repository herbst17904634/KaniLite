# KaniLite

このアプリケーションは、[Buttplug Sex Device Control Standard](https://buttplug-spec.docs.buttplug.io/) プロトコルを大幅に簡略化したwebsocketを提供します。これにより、より少ないプログラミングでデバイスにコマンドを送ることができます。このため、buttplug.ioプロトコルが困難もしくは不可能な環境での統合が実現可能になります。

## インストールと使用方法

1. 最新のリリースをダウンロードします。
2. KaniLite.exe を実行します。
3. 使用するデバイスのタグを追加します。
4. "設定を適用"を押して、設定を保存して、現在のサーバーに適用します。

## 特徴

- 簡略化されたButtplug Sex Device Control Standard
- 独立したアプリケーションで、他のソフトウェアを必要としません

![GUIのスクリーンショット](https://raw.githubusercontent.com/wiki/runtime-shady-backroom/buttplug-lite/images/buttplug-lite-2.0.0.png)

## 対応デバイス

[buttplug.io](https://iostindex.com/?filter0ButtplugSupport=4) がサポートするすべてのデバイスが使用可能です。これにはLovenseデバイスからXboxコントローラーまでを含みます。

## ソースコードのビルド

1. [Rustのインストール](https://www.rust-lang.org/tools/install)
2. プロジェクトをクローンします
3. `cargo build --release`

## 運用

### Resonite

Resonite用のProtoFluxのリファレンス実装は、以下のパブリックフォルダーにあります:  
`resrec:///U-Lehti/R-78C43B7CC794EE59412305C9A161E9883AA05BF82565325D8C28012018119E00` (ゲーム内でそのリンクを貼り付けて実体化してください)。

以下は、リファレンス実装のスクリーンショットです。

![screenshot of reference implementation](https://raw.githubusercontent.com/wiki/runtime-shady-backroom/buttplug-lite/images/reference-implementation-resonite-1.0.jpg)

この実装は、アバターに配置するように設計されています。ProtoFluxの上半分は、新しいユーザーがアバターに入るとウェブソケット接続をリセットするように設計されています。この部分は、アバターが常に一人のユーザーで使用される場合を除き、省略することができます。ProtoFluxの下半分は、約7Hzでbuttplug-liteサーバーに更新を送信します。7Hzを超えていくと、レイテンシーの問題を引き起こす可能性があります。2つの浮動入力は、0から1（含む）の範囲で、希望するモーターの強度を表します。これらは、最も近いユーザーの手、VirtualHapticPointSampler、または単純なUIスライダーからソースすることができます。

## マニュアル

### コマンドの送信

`ws://127.0.0.1:3031/haptic` にテキストタイプのメッセージを送信します。バイナリタイプのメッセージは現在サポートされていません。コマンドは最大10Hzで送信する必要があります。それ以上送信すると、アプリケーションのパフォーマンスが低下する可能性があります。

#### メッセージフォーマット

メッセージフォーマットは、セミコロン (`;`) で区切られたモーターコマンドのリストです。コマンドには3つの種類があります：スカラー、リニア、回転。すべてのコマンドは、特定のデバイス上の特定のモーターを表すユーザー定義の文字列であるモータータグから始まります。

##### Scalar

`tag:strength`

強度はモーターの強さを制御し、`0.0`から`1.0`の範囲で指定します。

##### Linear

`tag:duration:position`

位置は目標位置を制御し、`0.0`から`1.0`の範囲で指定します。  
持続時間はデバイスが目標位置に移動するまでの時間をミリ秒単位で制御します。持続時間は正の整数である必要があります。

##### Rotation

`tag:speed`

速度は回転の速さを制御し、`-1.0`から`1.0`の範囲で指定します。正の数値は時計回り、負の数値は反時計回りを表します。

##### Contraction (非推奨)

`tag:level`

**バージョン0.5.3から1.1.0までのみサポート**。バージョン2以降では、収縮はスカラーコマンドで処理されます。

ContractionはLovense Maxのポンプ強度を制御します。`0`から`3`までの整数である必要があります。

#### コマンド例

| Tag    | Type     | Strength | Duration | Position | Speed | Contraction |
|--------|----------|---------:|---------:|---------:|------:|------------:|
| foo    | Scalar   |       0% |          |          |       |             |
| bar    | Scalar   |      30% |          |          |       |             |
| baz    | Scalar   |     100% |          |          |       |             |
| gort   | Linear   |          |     20ms |      25% |       |             |
| klaatu | Linear   |          |    400ms |      75% |       |             |
| barada | Rotation |          |          |          | -0.75 |             |
| nikto  | Rotation |          |          |          |  0.26 |             |


```
foo:0;bar:0.3;baz:1;gort:20:0.25;klaatu:400:0.75;barada:-0.75;nikto:0.26
```

すべてのタグ付きモーターを指定する必要はありません。以下の例も有効ですが、もちろん`foo`モーターのみを制御します。
```
foo:0.1
```

#### Motor State

モーターは、次の更新が受信されるまで、最後に指示された振動や回転の速度で動作し続けます。

10秒間コマンドが受信されない場合、buttplug-liteは接続されているすべてのデバイスに停止コマンドを送信します。これを避けるには、希望するモーターの状態が変わらなくても、定期的にコマンドを送信してください。

### アプリケーションバージョンの確認

`http://127.0.0.1:3031/` にHTTP GETリクエストを送信します。200 OKが返され、ボディにはアプリケーション名とバージョンが含まれます。応答例：
```
buttplug-lite 0.7.0
```
バージョン0.7.0以前では、このエンドポイントは404を返します。

### 設定の確認

`http://127.0.0.1:3031/deviceconfig` にHTTP GETリクエストを送信します。200 OKが返され、ボディには機械可読形式の設定済みモーターリストが含まれます。応答例：
```
o;Lovense Edge;scalar
c;Lovense Max;scalar
i;Lovense Edge;scalar
m;Lovense Max;scalar
```

応答は改行（LF）で区切られたモーター設定のリストです。末尾にも改行があります。各モーター設定行は、セミコロン（`;`）で区切られたタグ、デバイス名、モータータイプのリストです。設定されたモーターがない場合、応答本文は空の文字列になります。

利用可能なモータータイプは次のとおりです：`linear`、`rotation`、および`scalar`。

バージョン0.7.0以前では、このエンドポイントは404を返します。

### ステータスの確認

`http://127.0.0.1:3031/hapticstatus` にHTTP GETリクエストを送信します。200 OKが返され、ボディには接続ステータスと接続されたデバイスのプレーンテキストの概要が含まれます。**この応答はデバッグ用であり、解析することを意図していません。**応答の構造は変更される可能性があります。デバイスステータスの解析が必要なユースケースがある場合は、課題を作成してお知らせください。

応答例：
```
device server running=true
  Lovense Edge
    ScalarCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "No description available for feature", _actuator_type: Vibrate, step_count: 20 }
    ScalarCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "No description available for feature", _actuator_type: Vibrate, step_count: 20 }
  Lovense Hush
    ScalarCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "No description available for feature", _actuator_type: Vibrate, step_count: 20 }
  Lovense Max
    ScalarCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "Vibrator", _actuator_type: Vibrate, step_count: 20 }
    ScalarCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "Air Pump", _actuator_type: Constrict, step_count: 5 }
  The Handy
    LinearCmd: ClientGenericDeviceMessageAttributes { feature_descriptor: "No description available for feature", _actuator_type: Position, step_count: 100 }
```

### Checking Battery
Send an HTTP GET to `http://127.0.0.1:3031/batterystatus`. A 200 OK will be returned with body containing a plain text list of devices and battery levels. Devices are delimited by newlines, battery levels are delimited by `:`. If the device has an unknown battery level a `-1` will be returned. Example:
```
Lovense Edge:1
Lovense Max:0.45
```

## Command-Line Arguments

buttplug-lite is intended to be used as a GUI, but for debugging purposes a few command-line arguments are included.

```
Usage: buttplug-lite [OPTIONS]

Options:
  -v, --verbose...               Sets the level of verbosity. Repeating this argument up to four times will apply increasingly verbose log_filter presets
  -c, --stdout                   Log to stdout instead of the default log file
  -f, --log-filter <LOG_FILTER>  Custom logging filter: https://docs.rs/tracing-subscriber/0.3.16/tracing_subscriber/filter/struct.EnvFilter.html. This completely overrides the `--verbose` setting
      --debug-ticks <SECONDS>    Emit periodic ApplicationStatusEvent ticks every <SECONDS> seconds. These "ticks" force the UI to update device state, which for example can be used to poll device battery levels
      --no-panic-handler         Disables the custom panic handler in the log file. Has no effect if used with `--stdout`
      --force-panic-handler      Enables the custom panic handler in stdout logs. Has no effect if file logging is used. Note that file logging is the default without an explicit `--stdout`
  -h, --help                     Print help
  -V, --version                  Print version
```

## Files

Here is where buttplug lite stores its various files on your filesystem:

|                             | Windows                                                    | macOS                                                                                   | *nix                                                                           |
|-----------------------------|------------------------------------------------------------|-----------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| **Configuration Directory** | `%APPDATA%\runtime-shady-backroom\buttplug-lite\config`    | `$HOME/Library/Application Support/io.github.runtime-shady-backroom.buttplug-lite`      | `$XDG_CONFIG_HOME/buttplug-lite` or `$HOME/.config/buttplug-lite`              |
| **Log Directory**           | `%APPDATA%\runtime-shady-backroom\buttplug-lite\data\logs` | `$HOME/Library/Application Support/io.github.runtime-shady-backroom.buttplug-lite/logs` | `$XDG_DATA_HOME/buttplug-lite/logs` or `$HOME/.local/share/buttplug-lite/logs` |

Note that once a maximum of 50 log files are reached, old logs will be rotated out.

## フィードバック

If you have bugs to report or ideas to suggest please let me know by opening an [issue](https://github.com/herbst17904634/KaniLite/issues) or starting a [discussion](https://github.com/herbst17904634/KaniLite/discussions).

## ライセンス

KaniLite は Buttplug Liteのフォーク版です。
KaniLite は [AGPL-3.0 license](LICENSE) で提供されます。
Coopyright of Buttplug Lite 2022-2023 [runtime-shady-backroom](https://github.com/runtime-shady-backroom) and [buttplug-lite contributors](https://github.com/runtime-shady-backroom/buttplug-lite/graphs/contributors).


