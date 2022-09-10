# トラブルシューティング

このページでは、今回の演習やROSを使っている際によくあるエラー等についてまとめておきます.

## roscoreが立ち上がらない

roscoreを立ち上げたいマシンのターミナルで

```shell
echo $ROS_MASTER_URI
echo $ROS_HOSTNAME
```

を実行してみてください、IPアドレスがlocalhost以外になってはいませんでしょうか？
この2つがlocalhost以外になる際には`roscore`がそのマシン以外で実行されている設定になっています.
以下のコマンドを実行し、localhostで`roscore`を動かす設定に変更してください.

```shell
export ROS_MASTER_URI=http://localhost:11311
```

その後、ROS_HOSTNAMEを適切な値(localhostか`ifconfig`コマンド等で確認したroscoreを実行したいマシンのIPアドレス)に設定し、

```shell
export ROS_HOSTNAME=localhost
```

## roscoreに接続できない

roscoreに接続したいマシンのターミナルで

```shell
echo $ROS_MASTER_URI
echo $ROS_HOSTNAME
```

を実行し、上記の環境変数が適切に設定されているか確認してください.

## roslaunchできない

### ノードが見つからない

`roslaunch`コマンドを実行した時、

```shell
ERROR: cannot launch node of type [gmapping/slam_gmapping]: gmapping
ROS path [0]=/opt/ros/melodic/share/ros
ROS path [1]=/opt/ros/melodic/share
```

上記のような出力が得られている場合、gmappingというパッケージが入っていないというエラーになります.
こういった場合、そのパッケージをlocal環境に存在するworkspaceにcloneしてビルドするか、apt経由でそのパッケージをインストールする必要があります.
