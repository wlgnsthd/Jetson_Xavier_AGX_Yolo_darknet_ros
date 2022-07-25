# Jetson Xavier AGX Yolo darknet ROS (Yolov4)
```
sudo apt-get update && sudo apt-get upgrade -y
```
## Check Version
```
git clone https://github.com/jetsonhacks/jetsonUtilities
cd jetsonUtilities
python jetsonInfo.py
pkg-config --modversion opencv
pkg-config --modversion opencv3
pkg-config --modversion opencv4
```
## ROS Melodic install
```
git clone https://github.com/jetsonhacks/installROSXavier
cd installROSXavier

./installROS.sh -p ros-melodic-desktop-full
./setupCatkinWorkspace.sh

sudo apt-get install ros-melodic-rqt*
sudo rosdep init
rosdep update
sudo apt-get install python-rosinstall

source /opt/ros/melodic/setup.bash
cd ~/catkin_ws/src
catkin_init_workspace

cd ~/catkin_ws
catkin_make

source ~/catkin_ws/devel/setup.bash
gedit ~/.bashrc
## 수정하기
# export ROS_MASTER_URI=http://localhost:11311
# export ROS_HOSTNAME=xxx.xxx.xxx.xxx
# alias cw='cd ~/catkin_ws'
# alias cs='cd ~/catkin_ws/src'
# alias cm='cd ~/catkin_ws && catkin_make'

roscore
# ctrl c
```
## Delete OpenCV4
```
sudo apt-get purge libopencv* python-opencv
sudo find /usr/local/ -name "*opencv*" -exec rm {} \;
# 안지워진 폴더 삭제
# sudo rm -rf /usr/local/include/opencv
# sudo rm -rf /usr/local/include/opencv4
# sudo rm -rf /usr/share/opencv
# sudo rm -rf /usr/share/opencv2
# sudo rm -rf /usr/share/opencv4
```
## Install OpenCV3.4.0
### with well known code (recommended)
```
$ git clone https://github.com/jetsonhacks/buildOpenCVXavier.git
$ cd buildOpenCVXavier
$ git checkout v1.0
$ ./buildOpenCV.sh
```
### with source code
___________________________
```
cd
mkdir opencv
cd opencv
wget -O opencv.zip https://github.com/opencv/opencv/archive/3.4.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/3.4.0.zip
unzip opencv
unzip opencv_contrib
cd opencv-3.4.0
mkdir build
cd build
```
```
# Xavier
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_CUDA=ON -D CUDA_ARCH_BIN="7.2" -D CUDA_ARCH_PTX="" -D WITH_CUBLAS=ON -D ENABLE_FAST_MATH=ON -D CUDA_FAST_MATH=ON -D ENABLE_NEON=ON -D WITH_GSTREAMER=ON -D WITH_LIBV4L=ON -D BUILD_opencv_python2=ON -D BUILD_opencv_python3=ON -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_EXAMPLES=OFF -D WITH_QT=ON -D WITH_OPENGL=ON -D CUDA_NVCC_FLAGS="--expt-relaxed-constexpr" -D WITH_TBB=ON ..

```
```
# xavier agx -j8
make -j$(nproc)
```
```
sudo make install
# 환경변수등록 
sudo sh -c 'echo '/usr/local/lib' > /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
```
## Install cv3_bridge 중요
https://github.com/leggedrobotics/darknet_ros/issues/34

```
# do not apt install ros-melodic-cv-bridge
cd catkin_ws/src
git clone https://github.com/superjax/cv3_bridge
#In src/darknet_ros/darknet_ros/CMakeLists.txt:
#find the cv_bridge and replace with cv3_bridge
#find the find_package(OpenCV REQUIRED) and change to find_package(OpenCV 3 REQUIRED)
#find the target_link_libraries(${PROJECT_NAME} ... and add ${OpenCV_LIBRARIES} to the end of the list
#package.xml에서 cv_brige를 cv3_bridge로 바꾸기

```
## Yolo darknet ROS install
```
cd catkin_ws/src
git clone --recursive https://github.com/Tossy0423/yolov4-for-darknet_ros.git
cd ../
# darknet_ros/darknet/Makefile열어서 GPU,CUDNN, OPENCV = 1로, gencode arch=compute_72,code=sm_72\ gencode arch=compute_72,code=[sm_72,compute_72]| 변경
# darknet_ros/darknet_ros/CMakeList.txt에서도 뒷부분 똑같이 수정
catkin_make -DKMAKE_BUILD_TYPE=Release
# 빌드 완료후
# usb 카메라 설치
sudo apt install ros-melodic-uvc-camera 

# darknet_ros/darknet_ros/config/ros.yaml에서 rostopic list에서 image 토픽 확인->/image_raw로 변경
# darknet_ros/darknet_ros/launch/darknet_ros에서 rostopic list에서 image 토픽 확인->/image_raw로 변경, yolo version에 맞게 yolov2,yolov3넣음

# roscore 실행하기
rosrun uvc_camera uvc_camera_node
roslaunch darknet_ros darknet_ros.launch # yolov2, v2 tiny 모델일때 
roslaunch darknet_ros yolo_v3.launch # yolov3 모델일때
# 안될경우 source /home/###/catkin_ws/devel/setup.bash 후 재실시
```
![Screenshot from 2022-07-13 14-26-35](https://user-images.githubusercontent.com/88171531/178666386-5acb53a7-7d59-4e19-8eeb-dc845d8df9db.png)
![Screenshot from 2022-07-13 16-19-20](https://user-images.githubusercontent.com/88171531/178674569-31422ff5-a134-4291-a55c-901f63630688.png)
