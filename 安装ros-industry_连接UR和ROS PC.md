### 了解关于ros-industrial（用ros控制UR机器人）

[http://wiki.ros.org/universal_robot/Tutorials/Getting%20Started%20with%20a%20Universal%20Robot%20and%20ROS-Industrial](http://wiki.ros.org/universal_robot/Tutorials/Getting Started with a Universal Robot and ROS-Industrial)

*ROS-Industrial support for Universal Robots manipulators (metapackage) 有已知的bug，API很不稳定，不推荐在生产系统中使用*

#### 安装ros-industrial

* 一步到位

```csharp
apt-get install ros-kinetic-universal-robot
```

* 从源码安装

从github clone文件很慢

#### 安装UR驱动包

https://github.com/UniversalRobots/Universal_Robots_ROS_Driver

* 新版驱动包特性：

> * Works for all **CB3 (with software version >= 3.7) and e-Series (software >= 5.1)** robots and uses the RTDE interface for communication, whenever possible.
> * **Factory calibration** of the robot inside ROS to reach Cartesian targets precisely.
> * **Realtime-enabled** communication structure to robustly cope with the 2ms cycle time of the e-Series. To use this, compile and run it on a kernel with the `PREEMPT_RT` patch enabled. (See the [Real-time setup guide](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/real_time.md) on how to achieve this)
> * Transparent **integration of the teach-pendant**. 
> * Use the robot's **speed-scaling**. 
> * **ROS-Service-based replacement of most every-day TP-interactions** offer using UR robots without interacting with the teach pendant at all, if desired. The robot can be started, stopped and even recovery from safety events can be done using ROS service- and action calls. See the driver's [dashboard services](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/ROS_INTERFACE.md#ur_robot_driver_node) and the [robot_state_helper node](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/ROS_INTERFACE.md#robot_state_helper) for details.



* 安装环境要求： 推荐 **Ubuntu 18.04 with ROS melodic**，Ubuntu 16.04 with ROS kinetic应该也可以。

为了减小延迟，推荐使用 real-time kernel。 [real-time setup guide](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/real_time.md)

* 安装步骤见github的readme

#### 安装UR机器人描述文件

描述文件用于记录机器人的模型参数

```
sudo apt install ros-kinetic-ur-description
```

#### 安装Moveit!：一个规划控制包

```
sudo apt-get install ros-kinetic-moveit
```

#### 设置机器人

这里UR机器人使用的是UR虚拟机

For using the *ur_robot_driver* with a real robot you need to install the **externalcontrol-1.0.2.urcap** which can be found inside the **resources** folder of this driver.

For installing the necessary URCap and creating a program, please see the individual tutorials on how to [setup a CB3 robot](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/install_urcap_cb3.md) or how to [setup an e-Series robot](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/install_urcap_e_series.md).

To setup the tool communication on an e-Series robot, please consider the [tool communication setup guide](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/setup_tool_communication.md).

#### e系列UR机器人安装URCap

[参考 Installing a URCap on a s-Series robot](https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_robot_driver/doc/install_urcap_e_series.md)

* 下载文件**externalcontrol-1.0.2.urcap**，which can be found inside the **resources** folder of this driver.

* 在系统设置中添加URCap，选择下载的文件
* 在安装设置中设置URCaps的外部控制，设置host's IP为ROS主机ip（ROS主机与UR机器人在同一网络下），自定义端口暂时不改。

 URCaps用法：

创建一个新程序，把External Control插入到程序树中

#### ROS PC连接机器人

* 查看机器人ip地址
* 设置环境变量

```
cd ~/ros_catkin_ws/src
source devel/setup.bash
```

* 提取配置信息

```
$ roslaunch ur_calibration calibration_correction.launch \
  robot_ip:=<robot_ip> target_filename:="${HOME}/my_robot_calibration.yaml"
```

此处<robot_ip>改为机器人ip地址，target_filename设置保存路径和名称

* 启动UR驱动

```
 roslaunch ur_robot_driver <robot_type>_bringup.launch robot_ip:=192.168.56.101
```

where **<robot_type>** is one of *ur3, ur5, ur10, ur3e, ur5e, ur10e*. 

*这时及以后就应该手在急停按钮附近了*

如果之前标定过机器人，把标定配置作为参数传入 launch file:

```
$ roslaunch ur_robot_driver <robot_type>_bringup.launch robot_ip:=192.168.56.101 \
 kinematics_config:=$(rospack find ur_calibration)/etc/ur10_example_calibration.yaml
```

If the parameters in that file don't match the ones reported from the robot, the driver will output an error during startup, but will remain usable.

* robot drive启动后，在UR机器人上运行之前的外部控制程序

Inside the ROS terminal running the driver you should see the output `Robot ready to receive control commands.`

> ## ROS端配置
>
> 由于最新的 ur_robot_driver 驱动已经将原驱动的 **/follow_joint_trajectory** 更改为 **/scaled_pos_traj_controller/follow_joint_trajectory** ，因此需要对个别文件进行修改才能正常使用。
>
> ### 修改moveit配置文件
>
> 以UR3为例，其他机型只需修改对应前缀即可：
> 找到该路径中的文件
>
> ```
> catkin_ws/src/fmauch_universal_robot/ur3_moveit_config/config/controllers.yaml
> 1
> ```

我的目录是：ros_catkin_ws/src/src/fmauch_universal_robot/ur5_e_moveit_config/config/controllers.yaml

> 将以下内容
>
> ```yaml
> action_ns: follow_joint_trajectory
> 1
> ```
>
> 修改为
>
> ```yaml
> action_ns: /scaled_pos_traj_controller/follow_joint_trajectory
> 1
> ```
>
> ### 修改默认测试脚本（可选）
>
> 如不修改，在运行脚本后将始终处于 wating 状态，即找不到有关的控制服务
> 找到该路径中的文件
>
> ```
> catkin_ws/src/fmauch_universal_robot/ur_driver/test_move.py
> 1
> ```
>
> 将第94行内容从
>
> ```python
> client = actionlib.SimpleActionClient('follow_joint_trajectory', FollowJointTrajectoryAction)
> 1
> ```
>
> 修改为
>
> ```python
> client = actionlib.SimpleActionClient('scaled_pos_traj_controller/follow_joint_trajectory', FollowJointTrajectoryAction)
> 1
> ```
>
> ## 虚拟机网络配置
>
> 使用 ROS 和 URSIM 各自的终端，输入ifconfig查看各自虚拟机的ip地址，测试能否ping通。有关教程网上很多，不再赘述。
> 如果未达到预期效果，请在虚拟机的网络设置中，把联网模式改为桥接模式。
>
> ## 总结
>
> 总体而言，相关教程在github页面已经说明得非常仔细，此处需要注意的就是 follow_joint_trajectory 命名的更改，至少在本人这里使用默认的文件设置是没有效果的。
> 在一切设置正常的情况下，运行测试脚本后，另一个虚拟机中的URSIM机械臂将开始移动，这意味着调试成功，运行指令为
>
> ```
> rosrun ur_driver test_move.py
> ```



#### 使用Moveit！控制机器人

* 启动moveit!的节点，并导入ur5的moveit!配置文件：

```
roslaunch ur5_e_moveit_config ur5_e_moveit_planning_execution.launch
```

成功的话输出：You can start planning now!

##### 遇到问题

问题：

```
[ERROR] [1595989337.293180061]: Robot model parameter not found! Did you remap 'robot_description'?
[ERROR] [1595989337.321633013]: Robot model not loaded
[ERROR] [1595989337.339022349]: Planning scene not configured
```

解决：





*  打开带有运动规划插件的Rviz：

  ```
  roslaunch ur5_e_moveit_config moveit_rviz.launch config:=true
  ```

* 由于MoveIt!在[−π,π][−π,π]范围内进行运动规划有困难，所以将关节约束在[−π2,π2][−π2,π2]范围内，使用关节约束的版本的命令：

```
roslaunch ur_robot_driver ur5_bringup.launch limited:=true robot_ip:=IP_OF_THE_ROBOT [reverse_port:=REVERSE_PORT]

roslaunch ur5_moveit_config ur5_moveit_planning_execution.launch limited:=true

roslaunch ur5_moveit_config moveit_rviz.launch config:=true 
```



#### 通信测试

> 如果一切正常， 你就会看到RVIZ中的UR5机械臂和实物的状态是一致的，拖动UR5机械臂实物，在RVIZ里的机械臂也会跟着运动。此时，你可以在RVIZ中用鼠标拖动机械臂到达目标位置，在planing下点击plan,如果路径规划成功即可点击execute,你就会看到UR5实物也会跟着运动到目标点。界面中可以调节机械臂运行速度。
> 这部分内容参考链接[UR机器人通讯与控制](https://programtip.com/en/art-134423)

##### 遇到问题

问题：

在运行

```
roslaunch ur5_moveit_config ur5_moveit_planning_execution.launch limited:=true
```

的时候，出现如下错误

```
[ERROR] [1595918090.847949113]: Unable to identify any set of controllers that can actuate the specified joints: [ elbow_joint shoulder_lift_joint shoulder_pan_joint wrist_1_joint wrist_2_joint wrist_3_joint ]
[ERROR] [1595918090.853504904]: Known controllers and their joints:
```

rviz中的机械臂控制可以动，而ur的机械臂无法运动

解决：

https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/blob/master/ur_calibration/README.md



矫正机器人的标定信息

```
$ roslaunch ur_calibration calibration_correction.launch \
robot_ip:=<robot_ip> target_filename:="${HOME}/my_robot_calibration.yaml"
```

把标定配置作为参数传入 launch file:

```
$ roslaunch ur_robot_driver ur5e_bringup.launch robot_ip:=192.168.121.130 \
  kinematics_config:=$(rospack find ur_calibration)/my_calibration/my_robot_calibration.yaml
```

https://github.com/UniversalRobots/Universal_Robots_ROS_Driver/issues/55#issuecomment-562215033
将ros_catkin_ws/src/src/fmauch_universal_robot/ur5_e_moveit_config/config/controllers.yaml中的`action_ns`参数改为`scaled_pos_traj_controller/follow_joint_trajectory`



问题：

```
[calibration_correction.launch] is neither a launch file in package [ur_calibration] nor is [ur_calibration] a launch file name The traceback for the exception was written to the log file
```

原因：

没有找到package

解决：

配置环境变量

```
cd ~/ros_catkin_ws/src
source devel/setup.bash
```

为了工作空间之间不相互干扰，推荐每次手动`source setup.bash`，如果只有一个工作空间，也可以直接写在`bashrc`里面。

source devel/setup.bash只在当前终端生效，每次打开其他终端时都要重新source，这样比较麻烦。 解决方法：`gedit ~/.bashrc`，打开.bashrc文件，在文件底部添加`source ~/ros_catkin_ws/src/devel/setup.bash`，保存退出即可。



#### 在Gazebo仿真环境下，模拟机器人运动

> ```go
> roslaunch ur_gazebo ur5.launch limited:=true
> 
> roslaunch ur5_moveit_config ur5_moveit_planning_execution.launch sim:=true limited:=true
> 
> roslaunch ur5_moveit_config moveit_rviz.launch config:=true
> ```

*Gazebo是一款3D动态模拟器*