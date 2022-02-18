# install ros from source

环境：Ubuntu 16.04 虚拟机

安装ROS kinetic

> 从source安装主要是参考了[ROS官方的tutorial](http://wiki.ros.org/Installation/Source) 。
>
> ### 1、安装bootstrap依赖
> $ sudo apt-get install python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential

#### 问题

遇到报错 

`E: Malformed entry 1 in list file /etc/apt/sources.list.d/ros-latest.list`

解决：

删除该文件：`sudo rm /etc/apt/sources.list.d/ros-latest.list`

> ### 2、初始化rosdep
> $ sudo rosdep init
> $ rosdep update
>     如果前面已经安装过ROS的同学在第一句时会报错，大概就是已经初始化过了，问是否要删除旧的重新初始化，不管它就好了0.0（以后要是出现啥问题就再说吧，呵呵呵）。

#### 问题

`$ sudo rosdep init` 

报错：`ERROR: cannot download default sources list from:
https://raw.github.com/ros/rosdistro/master/rosdep/sources.list.d/20-default.list
Website may be down.`

解决：

```
#打开hosts文件
$ sudo gedit /etc/hosts
#在文件末尾添加
$ 151.101.84.133  raw.githubusercontent.com
#保存后退出再尝试
```

### 添加sources.list

设置你的电脑以从 packages.ros.org 接收软件.

```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

添加公钥

```
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```

用国内镜像会快很多

``` 
sudo sh -c 'echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```

### 安装

首先，确保你的Debian软件包索引是最新的：

```
sudo apt-get update
```

#### 问题

报错：

```
E: Could not get lock /var/lib/apt/lists/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the list directory
```

原因：

>  刚装好的Ubantu系统，内部缺少很多软件源，这时，系统会自动启动软件源更新进程“apt-get”，并且它会一直存活。由于它在运行时，会占用软件源更新时的系统锁（以下称“系统更新锁”，此锁文件在“/var/lib/apt/lists/”目录下），而当有新的apt-get进程生成时，就会因为得不到系统更新锁而出现"E: 无法获得锁 /var/lib/apt/lists/lock - open (11: Resource temporarily unavailable)"错误提示！因此，我们只要将原先的apt-get进程杀死，从新激活新的apt-get进程，就可以让新立德软件管理器正常工作了！

解决：

查看进程

```
ps -e | grep apt
```

杀掉进程

```
sudo kill -9 进程号
```



> ### 3、新建catkin的工作空间
>
> $ mkdir ~/ros_catkin_ws
> $ cd ~/ros_catkin_ws
>     使用wstool安装ROS核心软件包，因为我这里已经安装了ROS Indigo的完全版，而且研究的内容也只是ROS的核心代码部分，用不上它的扩展功能，所以这边只介绍安装ROS_Comm（ROS基础包、构建和通信的库，没有GUI工具。），其余的见官方wiki。
>
> $ rosinstall_generator ros_comm --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall

#### 问题

这一步总是timeout。

解决：换了国内镜像软件源之后解决。



> $ wstool init -j8 src indigo-ros_comm-wet.rosinstall
>     值得注意的是，这里的ros_comm可以不是官方github的，也可以是自己fork出来的（自己修改后的当然也可以啦）。只需在将第一句命令改成如下：
>
> $ rosinstall_generator ros_comm --upstream --rosdistro indigo --deps --wet-only --tar > indigo-ros_comm-wet.rosinstall
>     也就是加上 --upstream 分句，这样在执行完这句命令时会出现indigo-ros_comm-wet.rosinstall这个文件，打开会出现如下：
>
> - tar:
>     local-name: catkin
>     uri: https://github.com/ros/catkin/archive/0.6.19.tar.gz
>     version: catkin-0.6.19
> - tar:
>     local-name: cmake_modules
>     uri: https://github.com/ros/cmake_modules/archive/0.3.3.tar.gz
>     version: cmake_modules-0.3.3
> .....................(省略数行）
> - tar:
>     local-name: ros_comm
>     uri: https://github.com/ros/ros_comm/archive/1.11.21.tar.gz      =》这里可以改成自己的uri
>     version: ros_comm-1.11.21
> - tar:
>     local-name: ros_comm_msgs
>     uri: https://github.com/ros/ros_comm_msgs/archive/1.11.2.tar.gz
>     version: ros_comm_msgs-1.11.2
> ......................(省略数行）
>     将里面的uri改成自己需要的uri保存即可。然后再按照上文所说执行wstool那行命令。
>



> ### 4、解决依赖关系
> $ rosdep install --from-paths src --ignore-src --rosdistro indigo -y
>     这将查看src目录中的所有软件包并查找它们的所有依赖关系。 然后将按照依赖关系依赖关系递归安装。 --from-paths选项表示我们要安装哪个目录的软件包依赖，在本例中为src。 --ignore-src选项指示rosdep不从包管理器的src文件夹中安装任何ROS软件包，因为我们正在自己构建它们。 --rosdistro选项是向rosdep指出正在构建的ROS版本。 最后，-y选项表明rosdep我们不希望受到来自软件包管理器的太多提示的困扰。
>
>     过了一段时间（也许有一些提示输入密码），rosdep将完成安装系统依赖关系。
>     
>     如果您从源代码或pip安装了某些内容，并且不希望rosdep尝试为其安装，请使用--skip-keys选项。 例如，如果从源代码安装了引导工具（如rosdep，rospkg和rosinstall_generator），请添加参数--skip-keys python-rosdep，--skip-keys python-rospkg，--skip-keys python-catkin-pkg 。
>



> ### 5、构建catkin的工作空间
>
> ​    调用catkin_make_isolated:
>
> $ ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release
>     现在应该已将软件包安装到～/ ros_catkin_ws / install_isolated或安装到了使用--install-space参数指定的位置。 如果查看该目录，则会看到已生成setup.bash文件。 



#### 问题

报错：

``` 
c++: internal compiler error: Killed (program cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
See <file:///usr/share/doc/gcc-5/README.Bugs> for instructions.
src/rviz/default_plugin/CMakeFiles/rviz_default_plugin.dir/build.make:926: recipe for target 'src/rviz/default_plugin/CMakeFiles/rviz_default_plugin.dir/point_cloud2_display.cpp.o' failed
make[2]: *** [src/rviz/default_plugin/CMakeFiles/rviz_default_plugin.dir/point_cloud2_display.cpp.o] Error 4
make[2]: *** Waiting for unfinished jobs....
CMakeFiles/Makefile2:2998: recipe for target 'src/rviz/default_plugin/CMakeFiles/rviz_default_plugin.dir/all' failed
make[1]: *** [src/rviz/default_plugin/CMakeFiles/rviz_default_plugin.dir/all] Error 2
Makefile:138: recipe for target 'all' failed
make: *** [all] Error 2
<== Failed to process package 'rviz': 
  Command '['/home/lucifer/ros_catkin_ws/install_isolated/env.sh', 'make', '-j2', '-l2']' returned non-zero exit status 2

Reproduce this error by running:
==> cd /home/lucifer/ros_catkin_ws/build_isolated/rviz && /home/lucifer/ros_catkin_ws/install_isolated/env.sh make -j2 -l2

```

解决：

for rviz,OGRE Not Found error:

```
sudo aptitude install libboost-thread-dev (YES)

sudo apt-get install libogre-1.9-dev ogre-1.9-doc ogre-1.9-tools (YES)
```



> 现在只需要设置环境变量：
>
> $ source ~/ros_catkin_ws/install_isolated/setup.bash
>    可以使用echo $ROS_PACKAGE_PATH查看是否设置成功。这样就可以用新安装的Ros_Comm覆盖之前安装的ROS的相关部分。
>



> ## 更新源代码
>
> 
>
> If we want to keep our source checkout up to date, we will have to periodically update our rosinstall file, download the latest sources, and rebuild our workspace.
>
> 
>
> ### Update the workspace
>
> 
>
> To update your workspace, first move your existing rosinstall file so that it doesn't get overwritten, and generate an updated version. For simplicity, we will cover the *destop-full* variant. For other variants, update the filenames and rosinstall_generator arguments appropriately.
>
> 
>
> ```
> $ mv -i kinetic-desktop-full-wet.rosinstall kinetic-desktop-full-wet.rosinstall.old
> $ rosinstall_generator desktop_full --rosdistro kinetic --deps --wet-only --tar > kinetic-desktop-full-wet.rosinstall
> ```
>
> 
>
> Then, compare the new rosinstall file to the old version to see which packages will be updated:
>
> 
>
> ```
> $ diff -u kinetic-desktop-full-wet.rosinstall kinetic-desktop-full-wet.rosinstall.old
> ```
>
> 
>
> If you're satisfied with these changes, incorporate the new rosinstall file into the workspace and update your workspace:
>
> 
>
> ```
> $ wstool merge -t src kinetic-desktop-full-wet.rosinstall
> $ wstool update -t src
> ```
>
> 
>
> 
>
> ### Rebuild your workspace
>
> 
>
> Now that the workspace is up to date with the latest sources, rebuild it:
>
> 
>
> ```
> $ ./src/catkin/bin/catkin_make_isolated --install
> ```
>
> 
>
> If you specified the `--install-space` option when your workspace initially, you should specify it again when rebuilding your workspace
>
> 
>
> Once your workspace has been rebuilt, you should source the setup files again:
>
> 
>
> ```
> $ source ~/ros_catkin_ws/install_isolated/setup.bash
> ```

