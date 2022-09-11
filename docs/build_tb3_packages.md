# Turtlebot3 関連パッケージのインストール方法

もし、Turtlebot3のパッケージがインストールされていなかった場合、以下の手順でビルドすることが可能です.

## apt経由でインストールする
```
sudo apt install ros-$ROS_DISTRO-turtlebot3*
```

## ソースコードからビルドする

```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
git clone https://github.com/ROBOTIS-GIT/turtlebot3.git
git clone https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git
rosdep update
rosdep install -iry --from-paths .
cd ../
catkin_make
```