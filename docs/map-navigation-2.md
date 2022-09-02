# マップを利用したナビゲージョン操作２

<span style="color:red">**下記の実習を行うために、まず`ROS_MASTER_URI`と`ROS_HOSTNAME`の`localhost`をリモートPCのIPアドレスに変更します。（[参照](./linux_and_ros_install.md#ネットワーク構成)）。**</span>

## 実機でナビゲーションノードを行う

**リモートPCで** roscoreを実行する。

``` bash
$ roscore
```

新しいターミナルウィンドウを開き、TurtleBot3と接続します。

```shell
$ ssh ubuntu@192.168.xxx.xxx
```
> **NOTE 1**: The IP `192.168.xxx.xxx` is your Raspberry Pi’s IP or hostname.  
> **NOTE 2**: パスワードは`turtlebot`です。

下記のような表示があれば接続が成功しました。
```shell
username@pc_name:~$ ssh ubuntu@192.168.10.11
ubuntu@192.168.10.11’s password:

...

Last login: [曜日] [月] [日] XX:XX:XX 2021 from 192.168.XX.XX
ubuntu@192.168.10.11:~$
```

```shell
$ roslaunch turtlebot3_bringup turtlebot3_robot.launch
```

新しいターミナルを開き、ナビゲーションファイルを起動します。

```shell
$ roslaunch turtlebot3_navigation turtlebot3_navigation.launch map_file:=$HOME/map.yaml
```

## 初期ポーズを推定する

まずロボットの初期姿勢推定を実行する必要があります。 RVizのメニューで`2D Pose Estimate`を押すと、非常に大きな緑色の矢印が表示されます。 所定のマップで実際のロボットが配置されているポーズに移動し、マウスの左ボタンを押したまま緑色の矢印をロボットの正面が向いている方向にドラッグし、以下の手順に従います。

- `2D Pose Estimate`{: style="border: 1px solid black" }ボタンを選択します。
- TurtleBot3が配置されているマップ内のおおよそのポイントをクリックし、カーソルをドラッグしてTurtleBot3が向いている方向を示します。

次に、`turtlebot3_teleop_keyboard`ノードなどのツールを使用してロボットを前後に動かし、周囲の環境情報を収集して、ロボットが現在地図上のどこにあるかを調べます。

```shell
$ roslaunch turtlebot3_teleop turtlebot3_teleop_key.launch
```

このプロセスが完了すると、ロボットは緑色の矢印で指定された位置と方向を初期ポーズとして使用して、実際の位置と方向を推定します。 すべての緑色の矢印は、TurtleBot3の予想される位置を表しています。 レーザースキャナーは、地図上に壁のおおよその図を描画します。 図面に図が正しく表示されない場合は、上の`2D Pose Estimate`{: style="border: 1px solid black" }ボタンをクリックして、TurtleBot3のローカライズを繰り返します。

![](/images/turtlebot3/2d_pose_estimate.png)

**注意**: `Estimate InitialPose`に使用される`turtlebot3_teleop_keyboard`ノードは、使用後に終了する必要があります。 そうしない場合はトピックが次のステップのナビゲーションノードの`/cmd_vel`トピックと重複するため、ロボットは奇妙な動作をします。
{: .notice--success}

## ナビゲーション目標の送信

すべての準備ができたらナビゲーションGUIからmoveコマンドを試してみましょう。 RVizのメニューで`2D Nav Goal`{: style="border: 1px solid black" }を押すと、非常に大きな緑色の矢印が表示されます。 この緑色の矢印は、ロボットの宛先を指定できるマーカーです。 矢印のルートはロボットの`x`と`y`の位置であり、矢印が指す方向はロボットの`theta`方向です。 ロボットが移動する位置でこの矢印をクリックし、ドラッグして以下の手順のように方向を設定します。

- `2D Nav Goal`{: style="border: 1px solid black" }ボタンを選択します。
- マップ内の特定のポイントをクリックしてゴール位置を設定し、カーソルをTurtleBotが最後に向いている方向にドラッグします。

ロボットは、地図に基づいて目的地への障害物を回避するためのパスを作成します。 次に、ロボットはパスに沿って移動します。 この時、いきなり障害物を検知しても障害物を避けて目標点に移動します。

![](/images/turtlebot3/2d_nav_goal.png)


ゴール位置へのパスを作成できない場合、ゴール位置の設定に失敗することがあります。 ロボットが目標位置に到達する前に停止したい場合は、TurtleBot3の現在の位置を目標位置として設定します。


## 参考文献

- [基本的なナビゲーションチューニングガイド (ROS Wiki)](http://wiki.ros.org/navigation/Tutorials/Navigation%20Tuning%20Guide)
- [Kaiyu ZhengによるROSナビゲーションチューニングガイド](http://kaiyuzheng.me/documents/navguide.pdf)
- ナビゲーション
  - [ROS WIKI](http://wiki.ros.org/navigation), [Github](https://github.com/ros-planning/navigation)


<button type="button" class="bth btn-primary btn-lg">[
    <span style="color:black">**メインページへ**</span>](index.html)</button>
<button type="button"  class="bth btn-success btn-lg">
    [<span style="color:black">**次の実習へ**</span>](obstacle-detection.html)</button>