# Turtlebot3 関連パッケージのインストール方法

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
```