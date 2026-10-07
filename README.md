# vs_rdrover_interfaces

ヴイストン製の研究開発用台車ロボットで共通利用するROS 2インターフェースパッケージです。現在はエンコーダー値のメッセージ型を提供します。
PC側のROS 2ワークスペースへ単独で導入できます。通信ノード、モータードライバー、オドメトリの計算処理は各ロボットのパッケージで実装します。

対応・検証済みのROS 2版はJazzyです。

`vs_`はVstone製パッケージの共通接頭辞です。

## PC側への導入

以下はLinuxの例です。ROS 2 Jazzy、`colcon`、初期設定済みの`rosdep`を用意してください。
ビルドには`ament_cmake`、`rosidl_default_generators`、`std_msgs`を使用します。

Arduinoライブラリの配布版に対応タグが指定されている場合は、そのタグを取得してください。以下は`jazzy`ブランチの取得例です。

```bash
source /opt/ros/jazzy/setup.bash
mkdir -p ~/rover_ws/src
cd ~/rover_ws/src
git clone --branch jazzy https://github.com/vstoneofficial/vs_rdrover_interfaces.git
cd ~/rover_ws
rosdep install --from-paths src/vs_rdrover_interfaces --ignore-src --rosdistro jazzy -y
colcon build --packages-select vs_rdrover_interfaces
source install/setup.bash
ros2 interface show vs_rdrover_interfaces/msg/EncoderStamped
```

既にパッケージを取得している場合は、`git clone`の代わりに、そのディレクトリを`~/rover_ws/src/vs_rdrover_interfaces`へコピーできます。
その直下に`package.xml`と`CMakeLists.txt`があることを確認してください。

`ros2 interface show`の出力に`std_msgs/Header header`と`int32[] counts`が表示されれば、メッセージ定義を利用できます。
新しいターミナルでも、ROS 2とワークスペースの環境を読み込みます。

```bash
source /opt/ros/jazzy/setup.bash
source ~/rover_ws/install/setup.bash
```

## EncoderStamped

ROS 2型名は`vs_rdrover_interfaces/msg/EncoderStamped`です。
定義は[msg/EncoderStamped.msg](msg/EncoderStamped.msg)を参照してください。

```text
std_msgs/Header header
int32[] counts
```

| フィールド | 内容 |
| --- | --- |
| `header.stamp` | 1回の取得処理で全エンコーダーの読取りを完了した、ROSと同期した時刻 |
| `header.frame_id` | 初期構成では空文字列 |
| `counts` | 各モーターの生の符号付き32 bitカウント値。単位はcount |

`counts`の要素数がエンコーダー数です。型は可変長配列で、固定長や最大8要素という制限はありません。
現在の2輪メガローバーでは`counts[0]`が左、`counts[1]`が右です。その他の機種では、車輪・モーターとの対応をその機種の実装と説明に従って確認してください。
同じ機種の通常運転中は、配列の順序と要素数を維持します。

値には角度・距離への変換、符号反転、カウンタの周回補正は含まれません。
受信側は32 bitカウンタの周回とモーター再起動時のリセットを考慮し、機種の分解能や車輪径を使って必要な変換を行います。

順次読み取る場合、`header.stamp`は全要素の厳密な同時取得時刻を意味しません。
送信側は全読取りに成功し、ROS時刻と同期できた回だけを送信します。失敗要素の削除や、前回値を今回の値として送ることはしません。
受信側も取得時刻と受信間隔を確認して、データの途絶を検出してください。

## 利用するROS 2パッケージからの依存

利用側の`package.xml`に依存を宣言します。

```xml
<depend>vs_rdrover_interfaces</depend>
```

CMakeを使用する場合は`find_package(vs_rdrover_interfaces REQUIRED)`で取得し、使用するターゲットの依存にも追加します。
Pythonでの型の取得例です。

```python
from vs_rdrover_interfaces.msg import EncoderStamped
```

## Arduinoとの定義の対応

送信側とPC側は、パッケージ名・型名・フィールドの型と順序が一致する定義を使用してください。
ヴイストン版micro-ROS Arduinoライブラリを使う場合は、各配布版で指定される本パッケージのタグまたはコミットに合わせます。
対応版が指定されていない配布物では、その配布物に同梱されたPC用ソースパッケージを使用してください。

Arduinoライブラリの`src/vs_rdrover_interfaces`は生成済みヘッダーです。PC側へ導入するのは、本リポジトリのROS 2ソースパッケージです。
Arduino側の利用方法は、[ヴイストン版micro_ros_arduino](https://github.com/vstoneofficial/micro_ros_arduino)のJazzy用案内とサンプルを参照してください。
同梱のエンコーダー送信サンプルはbest effort / volatileを使用するため、PC側の受信QoSも送信側に合わせてください。

### 開発版の旧パッケージ名からの移行

旧開発版の`vstone_rdrover_interfaces`から名称を変更しました。`EncoderStamped`のフィールドと意味は同じですが、ROS 2上の型名は異なります。
旧パッケージのフォルダを新パッケージへ置き換え、利用側の依存、include、Python import、型指定を`vs_rdrover_interfaces`へ変更して、PC側を新しいビルド出力先でビルドしてください。
Arduino側も新名称で再生成されたライブラリを使用し、スケッチをビルドし直します。旧名称のArduinoライブラリと新名称のPCパッケージを組み合わせないでください。

## ライセンス・お問い合わせ

ライセンスは[Apache License 2.0](LICENSE)です。
製品の利用に関するご相談は、[ヴイストンのお問い合わせ窓口](https://www.vstone.co.jp/contact/index.html)へお寄せください。
