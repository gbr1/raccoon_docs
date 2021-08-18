# Raccoon robot setup

---

> In this section you will guided on how to setup your robot in the following phases:
> 
> 1. OS installation
> 
> 2. Realsense setup
> 
> 3. ROS installation
> 
> 4. OpenVINO installation
> 
> 5. ROS packages setup
> 
> 6. Demo setup

---

## 1.  OS Installation

For this part you can follow: [Ubuntu_20.04 · up-board/up-community Wiki · GitHub](https://github.com/up-board/up-community/wiki/Ubuntu_20.04)

Steps to do are also described here.

- Download Ubuntu 20.04.2 from the official website and put on a pen drive using balena etcher.
  Plug the bootable pendrive into your UP Squared and turn on.

- Follow the instruction by doing a *minimal installation*

*NOTE: if you want to setup like the demo robot use "robot" as username and "demo" as computer name. Password is "demo". Set "log in automatically".*

- After installed Ubuntu, you need to patch the kernel using the UP ppa. Open the terminal and type:

```bash
sudo add-apt-repository ppa:aaeon-cm/5.4-upboard
sudo apt update
```

Remove all the generic installed kernel (select No on the question "Abort Kernel Removal")

```bash
sudo apt-get autoremove --purge 'linux-.*generic'
sudo apt-get install linux-generic-hwe-18.04-5.4-upboard
sudo apt dist-upgrade -y
sudo update-grub
sudo reboot
```

After reboot check the kernel:

```bash
uname -a
```

result must be:

```bash
Linux demo 5.4.0-1-generic #2~upboard2-Ubuntu SMP Thu Jul 25 13:35:27 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

Now we need to install the upboard-extras

```bash
sudo apt install upboard-extras
sudo usermod -a -G gpio ${USER}
sudo usermod -a -G leds ${USER}
sudo usermod -a -G spi ${USER}
sudo usermod -a -G i2c ${USER}
sudo usermod -a -G dialout ${USER}
sudo reboot
```

Now you are ready for step 2.

## 2.  Realsense setup

We need to install linux headers.

```bash
sudo apt install linux-headers-5.4.0-1-generic
sudo ln -s /usr/src/linux-headers-5.4.0-1-generic/ /lib/modules/5.4.0-1-generic/build
```

Now install the librealsense and realsense packages.

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE || sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
sudo add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u
sudo apt-get install librealsense2-dkms
sudo apt-get install librealsense2-utils
sudo apt-get install librealsense2-dev
sudo apt-get install librealsense2-dbg
```

then check if kernel is updated by:

`modinfo uvcvideo | grep "version:"`

result must include `realsense`

Then type `realsense-viewer` and update your RealSense camera if needed.

You can proceed to ROS installation.

## 3.  ROS installation

You can follow this guide: [noetic/Installation/Ubuntu - ROS Wiki](http://wiki.ros.org/noetic/Installation/Ubuntu)[noetic/Installation/Ubuntu - ROS Wiki](http://wiki.ros.org/noetic/Installation/Ubuntu)

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt install curl
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
sudo apt update
sudo apt install ros-noetic-desktop-full
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
sudo apt install python3-rosdep
sudo rosdep init
rosdep update
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin_make
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

You can now proceed to OpenVINO installation

## 4.  OpenVINO installation

Go to OpenVINO [website](https://docs.openvinotoolkit.org/latest/index.html) and download the release 2021.4 LTS.

Extract the folder l_openvino_toolkit_p_2021.4.582 on your Desktop folder.

```bash
cd ~/Desktop/l_openvino_toolkit_p_2021.4.582
sudo ./install_openvino_dependencies.sh
sudo ./install_GUI.sh
cd /opt/intel/openvino_2021/install_dependencies
sudo -E ./install_openvino_dependencies.sh
echo "source /opt/intel/openvino_2021/bin/setupvars.sh" >> ~/.bashrc
source ~/.bashrc
cd /opt/intel/openvino_2021/deployment_tools/model_optimizer/install_prerequisites
sudo ./install_prerequisites_caffe.sh
```

Now you need to install Myriad X support  (NCS2 or Ai Core X)

```bash
sudo usermod -a -G users "$(whoami)"
sudo cp /opt/intel/openvino_2021/inference_engine/external/97.myriad-usbboot.rules /etc/udev/rules.d
sudo udevadm control --reload-rules
sudo udevadm trigger
sudo ldconfig
sudo reboot
```

You have setted everything you need to start working with ROS packages and get your robot working!

## 5. ROS packages setup

You can start by installing dependencies:

```bash
sudo pip3 install tornado
sudo pip3 install simplejpeg
sudo apt install ros-noetic-realsense2-camera
```

Then you can clone repos:

```bash
cd ~/catkin_ws/src
git clone https://github.com/dheera/rosboard
git clone https://github.com/gbr1/moebiustech_stm32_ros
git clone https://github.com/gbr1/raccoon_ros
git clone -b new_core_api https://github.com/gbr1/ros_openvino
```

Now you need to install all the dependencies:

```bash
cd ..
rosdep install --from-paths src --ignore-src -r -y
catkin_make
catkin_make install
```

Your robot is ready to use!

If you want you can also configure the robot in "demo mode".

## 6.  Demo setup

You need to change permission to the startup script.

```bash
cd ~/catkin_ws/src/raccoon_ros/raccoon_demo/scripts/
chmod u+x startup.sh
```

Then in your launchpad search for "Startup application".

Inside set a new command called "demo" with the following command:

`gnome-terminal --command="/home/robot/catkin_ws/src/raccoon_ros/raccoon_demo/scripts/startup.sh"`

This will launch the demo at the startup.

Then you need to open the hotspot.

Go in Wi-Fi settings and "turn on hotspot" from 3 dots button.
Call the network "raccoon-robot" and set the password as "robotdemo".

Then edit to **true** the param **autoconnect** in following file:

`sudo -H gedit /etc/NetworkManager/system-connections/Hotspot.nmconnection`

You can now turn off the robot. When you turn the robot on, the demo will start and you will be able to connect to the robot hotspot.

The setup is ended.

> ***Copyright © 2021 G.Bruno gbr1.github.io under MIT license***
