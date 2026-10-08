# vs_rdrover_interfaces

ヴイストン製の研究開発用台車ロボットで共通利用するROS 2インターフェースパッケージです。現在はエンコーダー値のメッセージ型を提供します。
PC側のROS 2ワークスペースへ単独で導入できます。通信ノード、モータードライバー、オドメトリの計算処理は各ロボットのパッケージで実装します。

対応・検証済みのROS 2版はJazzyです。

## PC側への導入

以下はLinuxの例です。ROS 2 Jazzy、`colcon`、初期設定済みの`rosdep`を用意してください。
ビルドには`ament_cmake`、`rosidl_default_generators`、`std_msgs`を使用します。

Arduinoライブラリの配布版に対応タグが指定されている場合は、そのタグを取得してください。以下は`jazzy`ブランチの取得例です。
ここではrover_wsを新たに作成していますが、ご利用環境併せて既存のワークスペースを活用するなど調整してください。

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
メガローバーでは`counts[0]`が左、`counts[1]`が右です。その他の機種では、車輪・モーターとの対応をその機種の実装と説明に従って確認してください。

値には角度・距離への変換、符号反転、カウンタの周回補正は含まれません。
受信側は32 bitカウンタの周回とモーター再起動時のリセットを考慮し、機種の分解能や車輪径を使って必要な変換を行ってください。

順次読み取る場合、`header.stamp`は全要素の厳密な同時取得時刻を意味しません。
送信側は全読取りに成功し、ROS時刻と同期できた回だけを送信します。失敗要素の削除や、前回値を今回の値として送ることはしません。


## ライセンス・お問い合わせ

ライセンスは[Apache License 2.0](LICENSE)です。
製品の利用に関するご相談は、[ヴイストンのお問い合わせ窓口](https://www.vstone.co.jp/contact/index.html)へお寄せください。
