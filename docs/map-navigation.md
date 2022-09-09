# マップを利用したナビゲージョン操作１

**ナビゲーション** とは、特定の環境でロボットをある場所から指定された目的地に移動することです。この目的のために、特定の環境の家具、オブジェクト、および壁のジオメトリ情報を含むマップが必要です。 前の[SLAM](./slam-basics.md)セクションで説明したように、マップはセンサーによって取得された距離情報とロボット自体のポーズ情報を使用して作成されました。

ナビゲーションにより、ロボットは、マップ、ロボットのエンコーダー、IMUセンサーおよび距離センサーを使用して現在のポーズからマップ上の指定された目標ポーズに移動できます。 このタスクを実行する手順は次のとおりです。

まずシミュレーションで動作確認します。

<span style="color:red">**下記の実習は実機で行うために、まず`ROS_MASTER_URI`と`ROS_HOSTNAME`を`localhost`に戻します（[参照](https://kogakuin-mobility-system-lab.github.io/rsj-seminar-142/linux_and_ros_install.html#ネットワーク構成)）。**</span>

**シミュレーションを起動します**

```shell
$ roslaunch turtlebot3_gazebo turtlebot3_world.launch
```

下記のようなウィンドウが表示されるまで待ちましょう。

![](/images/turtlebot3/gazebo_turtlebot3_world.png)

**マップを作る**

新しいターミナルを開き、SLAMファイルを起動します。

``` bash
$ roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping
```

新しいターミナルを開き、遠隔操作ノードを実行します。

``` bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

ロボットを動かして下記のようなマップが作成すれば終了です。

![](/images/turtlebot3/map-turtlebot3-world.png)
<!-- <img src="/images/turtlebot3/map-turtlebot3-world.png" alt="map" width="300"/> -->

**マップを保存**

``` bash
$ rosrun map_server map_saver -f ~/map_sim
```

SLAMノードを起動しているターミナルで`Ctrl+c`{: style="border: 1px solid black" }を押してプログラムを終了させます。

## ナビゲーションノードの実行

新しいターミナルを開き、ナビゲーションファイルを起動します。

``` bash
$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map_sim.yaml
```

## 初期ポーズを推定する

まずロボットの初期姿勢推定を実行する必要があります。 RVizのメニューで`2D Pose Estimate`{: style="border: 1px solid black" }を押すと、非常に大きな緑色の矢印が表示されます。 所定のマップで実際のロボットが配置されているポーズに移動し、マウスの左ボタンを押したまま緑色の矢印をロボットの正面が向いている方向にドラッグし、以下の手順に従います。

- `2D Pose Estimate`{: style="border: 1px solid black" }ボタンを選択します。
- TurtleBot3が配置されているマップ内のおおよそのポイントをクリックし、カーソルをドラッグしてTurtleBot3が向いている方向を示します。

次に、`turtlebot3_teleop_keyboard`ノードなどのツールを使用してロボットを前後に動かし、周囲の環境情報を収集して、ロボットが現在地図上のどこにあるかを調べます。

``` bash
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

このプロセスが完了すると、ロボットは緑色の矢印で指定された位置と方向を初期ポーズとして使用して、実際の位置と方向を推定します。 すべての緑色の矢印は、TurtleBot3の予想される位置を表しています。 レーザースキャナーは、地図上に壁のおおよその図を描画します。 図面に図が正しく表示されない場合は、上の`2D Pose Estimate`{: style="border: 1px solid black" }ボタンをクリックして、TurtleBot3のローカライズを繰り返します。

![](/images/turtlebot3/2d_pose_estimate.png)

> **注意**: `2D Pose Estimate`{: style="border: 1px solid black" }に使用される`turtlebot3_teleop_keyboard`ノードは、使用後に終了する必要があります。 そうしない場合はトピックが次のステップのナビゲーションノードの`/cmd_vel`トピックと重複するため、ロボットは奇妙な動作をします。
{: .notice--success}

## ナビゲーション目標の送信

すべての準備ができたらナビゲーションGUIからmoveコマンドを試してみましょう。 RVizのメニューで`2D Nav Goal`{: style="border: 1px solid black" }を押すと、非常に大きな緑色の矢印が表示されます。 この緑色の矢印は、ロボットの宛先を指定できるマーカーです。 矢印のルートはロボットの`x`と`y`の位置であり、矢印が指す方向はロボットの`theta`方向です。 ロボットが移動する位置でこの矢印をクリックし、ドラッグして以下の手順のように方向を設定します。

- `2D Nav Goal`{: style="border: 1px solid black" }ボタンを選択します。
- マップ内の特定のポイントをクリックしてゴール位置を設定し、カーソルをTurtleBot3が最後に向いている方向にドラッグします。

ロボットは、地図に基づいて目的地への障害物を回避するためのパスを作成します。 次に、ロボットはパスに沿って移動します。 この時、いきなり障害物を検知しても障害物を避けて目標点に移動します。

![](/images/turtlebot3/2d_nav_goal.png)


ゴール位置へのパスを作成できない場合、ゴール位置の設定に失敗することがあります。 ロボットが目標位置に到達する前に停止したい場合は、TurtleBot3の現在の位置を目標位置として設定します。

## チューニングガイド

ナビゲーションスタックには、さまざまなロボットのパフォーマンスを変更するための多くのパラメーターがあります。 これに関する情報は、[ROS Wiki](http://wiki.ros.org/navigation)で入手するか、[ROS Robot Programming](https://community.robotsource.org/t/download-the-ros-robot-programming-book-for-free/51)の第11章を参照してください。

このチューニングガイドでは、重要なパラメーターを設定するためのヒントをいくつか紹介します。 環境に応じてパフォーマンスを変更したい場合は、このヒントが役立つ可能性があり、チューニングの時間を節約できます。

下記のパラメータのデフォルト値は`$(rospack find turtlebot3_navigation)/param/costmap_common_param_burger.yaml`のファイルに定義されています。下記のようなコマンドで設定することができます。

_**inflation_radius**_
- デフォルト値：1.0
- このパラメーターは、障害物から膨張領域を作成します。このエリアを越えないようにパスが計画されます。これをロボットの半径よりも大きく設定しても安全です。詳細については、[page of costmap_2d wiki](http://wiki.ros.org/costmap_2d#Inflation)をフォローしてください。

![](/images/turtlebot3/tuning_inflation_radius.png)

_**cost_scaling_factor**_ 
- デフォルト値：3.0
- この係数にコスト値を掛けます。相反する比率であるため、このパラメーターが増加し、コストが削減されます。

![](/images/turtlebot3/tuning_cost_scaling_factor.png)

  最適なパスは、ロボットが障害物間の中心を通過することです。 障害物から遠ざけるために、この係数を小さく設定します。

下記のパラメータのデフォルト値は`$(rospack find turtlebot3_navigation)/param/dwa_local_planner_params_burger.yaml`のファイルに定義されています。

> NOTE: `turtlebot3_navigation.launch`を起動する**前**に実施する必要があります．  
また、yamlファイルの編集に際して  
[apt経由でTurtlebot3関連パッケージをインストールした](../build_tb3_packages/#apt)場合、sudo権限が必要になる場合があります．  
[ソースコードからビルドしている](../build_tb3_packages/#_1)場合、catkin_makeコマンドの再実行が必要です．

<!-- `rosparam set`で変更することができます。 -->

_**max_vel_x**_
- デフォルト値：0.22
- この係数は、並進速度の最大値に設定されます。

_**min_vel_x**_
- デフォルト値：-0.22
- この係数は、並進速度の最小値に設定されます。 これを負に設定すると、ロボットは後方に移動できます。

_**max_vel_trans**_
- デフォルト値：0.22
- 最大並進速度の実際の値。 ロボットはこれより速くなることはできません。

_**min_vel_trans**_
- デフォルト値：0.11
- 最小並進速度の実際の値。 ロボットはこれより遅くなることはできません。

_**max_vel_theta**_
- `デフォルト値：2.75
- 最大回転速度の実際の値。 ロボットはこれより速くなることはできません。

_**min_vel_theta**_
- デフォルト値：1.37
- 最小回転速度の実際の値。 ロボットはこれより遅くなることはできません。

_**acc_lim_x**_
- デフォルト値：2.5
- 並進加速度リミット値の実際の値。

_**acc_lim_theta**_
- デフォルト値：3.2
- 回転加速度リミット値の実際の値。

_**xy_goal_tolerance**_
- デフォルト値：0.05
- ロボットが目標ポーズへの到達時に許可されるx、y距離。

_**yaw_goal_tolerance**_
- デフォルト値：0.17
- ロボットが目標ポーズへの到達時に許容されるヨー角。

_**sim_time**_
- デフォルト値：1.5
- この係数は、シミュレーションを秒単位で転送します。値が小さすぎると狭い領域を通過するのに十分な時間であり、値が大きすぎると急速に回転することはできません。 下の画像で黄色い線の長さの違いを見ることができます。

![](/images/turtlebot3/tuning_sim_time.png)

<!-- `parameter_name`の代わりに変更したいパラメータ名を書き換え、`value`に新しい値に書き換えます。 -->
ナビゲーションを起動します：
```shell
$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map_sim.yaml
```

パラメータを編集するために，２つの方法があります：

1. `$(rospack find turtlebot3_navigation)/param/dwa_local_planner_params_burger.yaml`を直接変更
> NOTE: `turtlebot3_navigation.launch`を起動する**前**に実施する必要があります．  
また、yamlファイルの編集に際して  
[apt経由でTurtlebot3関連パッケージをインストールした](../build_tb3_packages/#apt)場合、sudo権限が必要になる場合があります．  
[ソースコードからビルドしている](../build_tb3_packages/#_1)場合、catkin_makeコマンドの再実行が必要です．

2. `rosrun rqt_reconfigure rqt_reconfigure`による変更　　
> NOTE: `turtlebot3_navigation.launch`を起動した**後**に実施する必要があります　　