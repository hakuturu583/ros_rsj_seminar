# ROSを用いたマップ取得

**SLAM（Simultaneous Localization and Mapping）**は任意の空間の現在位置を推定して地図を描く手法です. SLAMはTurtleBot3の前身からよく知られている機能です.

## TurtleBot3を起動

<span style="color: red; ">roscore、sshはリモートPCで実行をお願いします.</span>

`roscore`を実行します.

```shell
roscore
```

TurtleBot3のアプリケーションを起動するための基本的なパッケージを起動します.

新しいターミナルウィンドウを開き、TurtleBotと接続します.

```shell
ssh ubuntu@192.168.xxx.xxx
```
> **NOTE 1**: The IP `192.168.xxx.xxx` is your Raspberry Pi’s IP or hostname.  
> **NOTE 2**: パスワードは`turtlebot`です.

接続ができましたら下記のコマンドでTurtleBot3を起動します.

<span style="color: red; ">こちらのコマンドはsshしているターミナル上で実行お願いします.</span>

```shell
roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

TurtleBot3のモデルが`burger`の場合は、以下のようなメッセージが表示されます.

```shell
SUMMARY
========

PARAMETERS
 * /rosdistro: noetic
 * /rosversion: 1.12.13
 * /turtlebot3_core/baud: 115200
 * /turtlebot3_core/port: /dev/ttyACM0
 * /turtlebot3_core/tf_prefix:
 * /turtlebot3_lds/frame_id: base_scan
 * /turtlebot3_lds/port: /dev/ttyUSB0

NODES
  /
    turtlebot3_core (rosserial_python/serial_node.py)
    turtlebot3_diagnostics (turtlebot3_bringup/turtlebot3_diagnostics)
    turtlebot3_lds (hls_lfcd_lds_driver/hlds_laser_publisher)

ROS_MASTER_URI=http://192.168.1.2:11311

process[turtlebot3_core-1]: started with pid [14198]
process[turtlebot3_lds-2]: started with pid [14199]
process[turtlebot3_diagnostics-3]: started with pid [14200]
[INFO] [1531306690.947198]: ROS Serial Python Node
[INFO] [1531306691.000143]: Connecting to /dev/ttyACM0 at 115200 baud
[INFO] [1531306693.522019]: Note: publish buffer size is 1024 bytes
[INFO] [1531306693.525615]: Setup publisher on sensor_state [turtlebot3_msgs/SensorState]
[INFO] [1531306693.544159]: Setup publisher on version_info [turtlebot3_msgs/VersionInfo]
[INFO] [1531306693.620722]: Setup publisher on imu [sensor_msgs/Imu]
[INFO] [1531306693.642319]: Setup publisher on cmd_vel_rc100 [geometry_msgs/Twist]
[INFO] [1531306693.687786]: Setup publisher on odom [nav_msgs/Odometry]
[INFO] [1531306693.706260]: Setup publisher on joint_states [sensor_msgs/JointState]
[INFO] [1531306693.722754]: Setup publisher on battery_state [sensor_msgs/BatteryState]
[INFO] [1531306693.759059]: Setup publisher on magnetic_field [sensor_msgs/MagneticField]
[INFO] [1531306695.979057]: Setup publisher on /tf [tf/tfMessage]
[INFO] [1531306696.007135]: Note: subscribe buffer size is 1024 bytes
[INFO] [1531306696.009083]: Setup subscriber on cmd_vel [geometry_msgs/Twist]
[INFO] [1531306696.040047]: Setup subscriber on sound [turtlebot3_msgs/Sound]
[INFO] [1531306696.069571]: Setup subscriber on motor_power [std_msgs/Bool]
[INFO] [1531306696.096364]: Setup subscriber on reset [std_msgs/Empty]
[INFO] [1531306696.390979]: Setup TF on Odometry [odom]
[INFO] [1531306696.394314]: Setup TF on IMU [imu_link]
[INFO] [1531306696.397498]: Setup TF on MagneticField [mag_link]
[INFO] [1531306696.400537]: Setup TF on JointState [base_link]
[INFO] [1531306696.407813]: --------------------------
[INFO] [1531306696.411412]: Connected to OpenCR board!
[INFO] [1531306696.415140]: This core(v1.2.1) is compatible with TB3 Burger
[INFO] [1531306696.418398]: --------------------------
[INFO] [1531306696.421749]: Start Calibration of Gyro
[INFO] [1531306698.953226]: Calibration End
```

## SLAMノードの実行

<span style="color: red; ">
SLAMノードはTB3搭載のボードではなくリモートPCで実行します.
</span>

Turtlebot3関連のパッケージが未インストールの場合は、[こちら](../build_tb3_packages)を参考にインストールしてください.

新しいターミナルを開き、SLAMを実行するlaunchファイルを起動します.

```shell
roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
```

もし、実行時に以下のようなエラーが発生した場合、

```shell
RLException: Invalid <arg> tag: environment variable 'TURTLEBOT3_MODEL' is not set. 

Arg xml is <arg default="$(env TURTLEBOT3_MODEL)" doc="model type [burger, waffle, waffle_pi]" name="model"/>
The traceback for the exception was written to the log file
```

お持ちのTurtlebot3の名称に合わせてTURTLEBOT3_MODEL環境変数を以下の中から選択して設定してください.(burger, waffle, waffle_pi)
```shell
export TURTLEBOT3_MODEL=burger
```

その他にも以下のようなエラーが発生した場合、
```shell
ERROR: cannot launch node of type [gmapping/slam_gmapping]: gmapping
ROS path [0]=/opt/ros/melodic/share/ros
ROS path [1]=/opt/ros/melodic/share
```
このエラーはgmmpingがインストールされていないことで発生しています.
ですので、このエラーは以下のコマンドでgmappingをインストールすることで解決します.

```shell
sudo apt update && sudo apt install gmapping
```

> **ヒント**: 上記のコマンドを実行すると、視覚化ツールRVizも実行されます. RVizを個別に実行する場合は、次のいずれかのコマンドを使用します.
> - ```$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_gmapping.rviz```
> - ```$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_cartographer.rviz```
> - ```$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_hector.rviz```
> - ```$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_karto.rviz```
> - ```$ rviz -d `rospack find turtlebot3_slam`/rviz/turtlebot3_frontier_exploration.rviz```

**注釈**: さまざまなSLAMメソッドをサポートしています
- TurtleBot3は、さまざまなSLAMメソッドの中で、Gmapping、Cartographer、Hector、およびKartoをサポートしています. これを行うには、 `slam_methods：= xxxxx`オプションを変更します.
- `slam_methods`オプションには`gmapping`、 `cartographer`、`hector`、 `karto`、`frontier_exploration`が含まれ、それらの1つを選択できます.
- たとえば、kartoを使用するには、次のようにします:
```shell
roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=karto
```

**注釈**: SLAMパッケージの依存関係パッケージをインストールします
- `Gmapping`の場合: <br>
Gmappingに関連するパッケージは、[事前準備](https://kogakuin-mobility-system-lab.github.io/rsj-seminar-142/linux_and_ros_install.html#ros-依存パッケージのインストール)ページですでにインストールされています.

- `Cartographer`の場合:
```shell
sudo apt-get install ros-melodic-cartographer ros-melodic-cartographer-ros \
  ros-melodic-cartographer-ros-msgs ros-melodic-cartographer-rviz
```

- `Hector Mapping`の場合:
```shell
sudo apt-get install ros-melodic-hector-mapping
```

- `Karto`の場合:
```shell
sudo apt-get install ros-melodic-slam-karto
```

- `Frontier Exploration`の場合  <br>
こちらのパッケージはmelodicにおいてはソースコードからのビルドが必要です.
```shell
mkdir -p catkin_ws/src
cd catkin_ws/src
sudo apt install git googletest
catkin_init_workspace
git clone https://github.com/paulbovbel/frontier_exploration.git
rosdep update
rosdep install -iry --from-paths .
cd ../
catkin_make
```

上のスクリプトの内部で出てきた`rosdep`コマンドはapt等をラップしたrosのコマンドラインツールで
ROSパッケージにあるpackage.xmlをパースしパッケージが依存しているパッケージを列挙、インストールしてくれます.

> **NOTE**: 今回は`Gmapping`を使用します.


## 遠隔操作ノードの実行

<span style="color: red; ">遠隔操作ノードはTB3搭載のボードではなくリモートPCで実行します.</span>

新しいターミナルを開き、[前回の実習](https://kogakuin-mobility-system-lab.github.io/rsj-seminar-142/turtlebot-basics.html#キーボードでロボットを操作-1)で使用した遠隔操作ノードを実行します. 次のコマンドを使用すると、ユーザーはロボットを制御してSLAM操作を手動で実行できます. 速度の変更が速すぎたり、回転が速すぎたりするなどの激しい動きを避けることが重要です. ロボットを使用して地図を作成する場合、ロボットは測定対象の環境の隅々までスキャンする必要があります. きれいな地図を作成するにはある程度の経験が必要なので、SLAMを複数回練習してノウハウを作成しましょう. マッピングプロセスを次の図に示します.

```shell
roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

```shell
  Control Your TurtleBot3!
  ---------------------------
  Moving around:
          w
     a    s    d
          x

  w/x : increase/decrease linear velocity
  a/d : increase/decrease angular velocity
  space key, s : force stop

  CTRL-C to quit
```

![](/images/turtlebot3/slam_running_for_mapping.png)

## チューニングガイド

Gmappingには、さまざまな環境のパフォーマンスを変更するための多くのパラメーターがあります. パラメーター全体に関する情報は、[ROS WiKi](http://wiki.ros.org/gmapping)で入手するか、[ROS Robot Programming](https://community.robotsource.org/t/download-the-ros-robot-programming-book-for-free/51)の第11章を参照してください.

このチューニングガイドでは、重要なパラメーターを設定するためのヒントをいくつか紹介します. 環境に応じてパフォーマンスを変更したい場合は、このヒントが役立つ可能性があり、時間を節約できます.

> **NOTE**: 下記のパラメータのデフォルト値は`$(rospack find turtlebot3_slam)/config/gmapping_params.yaml`のファイルに定義されています.

- _**maxUrange**_ 
  - デフォルト値：3.0
  - このパラメーターは、LIDARセンサーの最大使用可能範囲を設定します.

- _**map_update_interval**_
  - デフォルト値：2.0 
  - マップの更新間の時間（秒単位）. これを低く設定すると、マップがより頻繁に更新されます. ただし、より大きな計算負荷が必要になります. このパラメーターの設定は、環境によって異なります.
  ![](/images/turtlebot3/tuning_map_update_interval.png)

- _**minimumScore**_ 
  - デフォルト値：50 
  - スキャンマッチングの結果を考慮するための最小スコア. このパラメーターにより、ポーズ推定のジャンプを回避できます.
     これが適切に設定されている場合は、（SLAMのノードが起動しているターミナルで）以下の情報を見ることができます.
  
    ```
    Average Scan Matching Score=278.965
    neff= 100
    Registering Scans:Done
    update frame 6
    update ld=2.95935e-05 ad=0.000302522
    Laser Pose= -0.0320253 -5.36882e-06 -3.14142
    ```
  
    この設定が高すぎる場合は、以下の警告が表示されます.
  
    ```
    Scan Matching Failed, using odometry. Likelihood=0
    lp:-0.0306155 5.75314e-06 -3.14151
    op:-0.0306156 5.90277e-06 -3.14151
    ```

- _**linearUpdate**_ 
  - デフォルト値：1.0 
  - ロボットが移動すると、毎回スキャン処理が行われます.

- _**angularUpdate**_ 
  - デフォルト値：0.2
  - ロボットが回転すると、毎回スキャン処理が行われます. これをlinearUpdateよりも小さく設定することを推奨します.

パラメータを編集するために：

- `$(rospack find turtlebot3_slam)/config/gmapping_params.yaml`を直接変更
> NOTE: `turtlebot3_slam.launch`を起動する**前**に実施する必要があります．  
また、[apt経由でTurtlebot3関連パッケージをインストールした](../build_tb3_packages/#apt)場合、sudo権限が必要になる場合があります．  
[ソースコードからビルドしている](../build_tb3_packages/#_1)場合、catkin_makeコマンドの再実行が必要です．

> **ヒント**: どういうパラメータがあるかのとパラメータ名を調べるには：
> ```shell
> rosparam list
> ```

> **ヒント**: パラメータの値を確認したい場合は：
> ```shell
> rosparam get パラメータ名
> ```

## マップの保存

すべての作業が完了したので、`map_saver`ノードを実行してマップファイルを作成します. マップは、ロボットのオドメトリ、tf情報、およびロボットが移動したときのセンサーのスキャン情報に基づいて描画されます.これらのデータは、前のサンプルビデオのRVizで見ることができます.作成されたマップは、`map_saver`が実行されているディレクトリに保存されます.ファイル名を指定しない限り、マップ情報を含む`map.pgm`および`map.yaml`ファイルとして保存されます.

``` bash
rosrun map_server map_saver -f ~/map
```

`-f`オプションは、マップファイルが保存されているフォルダーとファイル名を参照します.`~/map`をオプションとして使用すると、`map.pgm`と `map.yaml`がユーザーのホームフォルダー`~/`（$HOME：`/home/<username>`）のmapフォルダーに保存されます.

## マップ

ROSコミュニティでよく使用されている2次元の `Occupancy Grid Map（OGM）`を使用します. 下の図に示すように、前の[マップの保存](#マップの保存)セクションから取得したマップ.**白色**はロボットが移動可能な空き領域、**黒色**はロボットが動作できない占有領域です.**灰色**は未知の領域です. このマップは[ナビゲーション](./map-navigation-2.md)で使用されます.

![](/images/turtlebot3/map.png)

次の図は、TurtleBot3を使用して大きなマップを作成した結果を示しています. 移動距離が約350メートルの地図を作成するのに約1時間かかりました.

![](/images/turtlebot3/large_map.png)

自分で作成したマップ（map.pgm）をダブルクリックで開けます.

## 全てのプログラムを終了

<!--`SLAM`と`Teleop`のノードを終了させます.<br>
それぞれのターミナルで`Ctrl+c`{: style="border: 1px solid black" }を押します.-->

```shell
rosnode kill -a
```

`roscore`を終了させるのに`roscore`を起動しているターミナルで`Ctrl+c`{: style="border: 1px solid black" }を押します.
下記のような表示があれば`roscore`が終了したことが確認できます.

```shell
^C[rosout-1] killing on exit
[master] killing on exit
shutting down processing monitor ...
... shutting down processing monitor complete
done
username@pcname:~/catkin_ws$
``` 

TurtleBot3のアプリケーションを起動するために開いたターミナルで`exit`を記入し、`Enter`{: style="border: 1px solid black" }キーを押すと接続を切断します.

```shell
turtlebot@turtlebot:~$ exit
username@pc_name:~$
```

## 参考文献

- gmapping
  - [ROS WIKI](http://wiki.ros.org/gmapping), [Github](https://github.com/ros-perception/slam_gmapping)

- cartographer 
  - [ROS WIKI](http://wiki.ros.org/cartographer), [Github](https://github.com/googlecartographer/cartographer)

- hector
  - [ROS WIKI](http://wiki.ros.org/hector_slam), [Github](https://github.com/tu-darmstadt-ros-pkg/hector_slam)

- karto
  - [ROS WIKI](http://wiki.ros.org/slam_karto), [Github](https://github.com/ros-perception/slam_karto)

- frontier_exploration 
  - [ROS WIKI](http://wiki.ros.org/frontier_exploration), [Github](https://github.com/paulbovbel/frontier_exploration)