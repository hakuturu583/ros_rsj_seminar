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

### 何もterminalにログが出力されず、立ち上がらない

roscore接続に失敗している可能性があります.
[roscoreが立ち上がらない](./#roscore)、[roscoreに接続できない](./#roscore_1)の記載を確認し、roscoreの挙動を確認してください.
## catkin_make時にエラーになる
### find_pacakgeに失敗する
以下のような出力が得られる場合、複数の原因が考えられます.

```shell
CMake Error at /opt/ros/indigo/share/catkin/cmake/catkinConfig.cmake:83 (find_package):
  Could not find a package configuration file provided by "roscpp" with any
  of the following names:

    roscppConfig.cmake
    roscpp-config.cmake
```

- 該当パッケージへのパスが通っていない：`source /opt/ros/$ROS_DISTRO/setup.bash`や、
localに存在するworkspace内部のsetup.bashを`source`コマンドで読み込みパッケージをcatkinビルドシステムが探索できるようにする
- rosdepの実行を忘れている：rosdepを実行し忘れている場合、依存が入っていない場合があります.workspaceのsrcディレクトリに移動したあと
`rosdep install -iry --from-paths .`を実行することで依存をまとめてインストールすることができます.
- package.xmlに依存漏れがある:package.xml
- 何らかの必要なパッケージをいれるステップを飛ばしてしまっている:パッケージによっては手動で依存ライブラリをインストールしておく必要があるものがあります.
どうしても原因がわからない場合は、github/gitlabリポジトリを探し、issueを立てるのも良いと思います.

### includeに失敗する

以下のような出力が得られる場合、複数の原因が考えられます.

```shell
fatal error: exploration_msgs/ExploreAction.h: No such file or directory
```

- CMakeLists.txtの`find_pacakge`コマンドに漏れがある:catkinは依存パッケージに存在するヘッダファイルをincludeしたりすることが可能です、
しかし`find_package`コマンドに記載漏れがあると適切にパスを解決してヘッダーを探しに行くことができません.
- catkin_init_workspaceを忘れている:catkinは依存を考慮してパッケージをビルドすることができます.
しかし、catkin_init_workspaceを忘れていると依存が適切に考慮されないケースがあります.
- バージョンアップによりヘッダーがなくなっている/ファイル名が変わっている:ROS packageは様々なものが開発されており、今後もエコシステムは拡張され続けています.
それに伴い、定期的なバージョンアップが行われているパッケージも少なくありません.
そのためパッケージによっては`.h`ファイルを`.hpp`に変更したりしているケースもあり結果として`#include`するヘッダの名称を変更しないといけないケースがあります.

### undefined referenceエラーが発生

undefined referenceとは、ヘッダーファイルの探索には成功したが、関数の実装が見当たらずコンパイルエラーになっているということです.
そのため以下のような原因が考えられます.

- CMakeLists.txtの`find_pacakge`コマンドに漏れがある:`find_package`コマンドに記載漏れがあると適切にパスを解決して関数の実装を探しにくことができません.
- target_link_librariesを忘れている:[こちら](https://github.com/ROBOTIS-GIT/turtlebot3/blob/66681b33749c44e7d9022253ac210ef2da7843a0/turtlebot3_bringup/CMakeLists.txt#L49)のように`${catkin_LIBRARIES}`をlink対象として指定しないと関数の実装を探索することができません.

## gazeboがErrorを出してしまい、起動しない

roslaunchやgazeboコマンド経由でgazeboを立ち上げると以下のようなエラーが出るケースがあります.

```shell
[Err] [REST.cc:205] Error in REST request
```

こちらのメッセージは、gazeboの仕様および設定ファイルの記載ミスによります.
gazeboは初回起動時に基本アセット（3D Model等）をダウンロードしてきます.
その時gazeboのデータをダウンロードしてくるサーバーのURLが`~/.ignition/fuel/config.yaml`に記載されています.

```yaml
# The list of servers.
servers:
  -
    name: osrf
    url: https://api.ignitionfuel.org

  # -
    # name: another_server
    # url: https://myserver

# Where are the assets stored in disk.
# cache:
#   path: /tmp/ignition/fuel
```

おそらく、gazeboをインストールした直後はこのように記載されています.
このうちサーバーのURLである`https://api.ignitionfuel.org`を`https://api.ignitionrobotics.org`に差し替えてください.

```yaml
# The list of servers.
servers:
  -
    name: osrf
    url: https://api.ignitionrobotics.org

  # -
    # name: another_server
    # url: https://myserver

# Where are the assets stored in disk.
# cache:
#   path: /tmp/ignition/fuel
```

編集後、

```shell
gazebo
```

コマンドを実行し、gazeboがエラー無く立ち上がることを確認してください.
[参考リンク](https://github.com/ros-industrial/universal_robot/issues/412)
