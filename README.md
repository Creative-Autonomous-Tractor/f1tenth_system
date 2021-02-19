# RC카 실제 주행을 위한 ROS 패키지
Jetson TX2 위에서 성공적으로 동작하는 ROS package입니다. 아래 사항들을 읽어서 자신의 상황에 맞게 코드를 바꿔주시면 됩니다.

## 사용하는 법
만들어놓은 ROS workspace 내 `src` 폴더에서 `git clone`을 해서 다운을 받고, `catkin_make`를 통해 build를 해줍니다.
그 다음 `roslaunch f1tenth_racecar teleop.launch`를 실행해주면,
컨트롤러의 왼쪽 범퍼를 누르고 있을 때는 직접 주행, 컨트롤러의 오른쪽 범퍼를 누르고 있을 때는 자율 주행 (미리 넣어놓은 주행 알고리즘)이 가능해집니다.
아래 과정들을 통해 원하는 알고리즘을 바꿔 넣을 수 있습니다.

## 주행 알고리즘 파일이 들어갈 위치
이 예시에서는 [ourdrive_node.cpp](f1tenth_racecar/src/ourdrive_node.cpp)가 주행 알고리즘 파일입니다. 원하는대로 바꾸거나 아예 새로운 파일을 넣으면 되는데, 아래 참고하시면 좋을 부분이 있습니다.

```c++
lidar_sub_(node_handle_.subscribe("/scan", 100, &longest_path::scan_callback, this)),
odom_sub_(node_handle_.subscribe("odom", 100, &longest_path::odom_callback, this)),
drive_pub_(node_handle_.advertise<ackermann_msgs::AckermannDriveStamped>("low_level/ackermann_cmd_mux/input/navigation", 100)), // originally "nav"
```
[원본 코드](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/src/ourdrive_node.cpp#L132-L134)

LIDAR 정보가 들어있는 topic은 `"/scan"`이고, odometry 정보가 들어있는 topic은 `"/vesc/odom"`이고, Ackermann 주행 메시지를 담은 topic은 `"/vesc/low_level/ackermann_cmd_mux/input/navigation"`입니다. 

`lidar_sub_`와 같은 변수 이름들은 달라져도 됩니다. 이외에도 다른 topic들을 사용하고 싶다면 `rostopic list` 명령으로 원하는 topic 이름을 찾아내면 됩니다.

컨트롤러의 오른쪽 범퍼를 누르고 있지 않아도 작동하게 하려면 주행 메시지가 `/vesc/low_level/ackermann_cmd_mux/output`으로 publish하도록 바꾸면 됩니다.

## CMakeLists.txt
```CMake
add_executable(노드이름 src/파일이름.cpp)
target_link_libraries(노드이름 ${catkin_LIBRARIES})
```
[원본 코드](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/CMakeLists.txt#L28-L29)

자신의 파일의 node 이름과 파일 자체의 이름을 넣어줍니다.

## launch 파일
```xml
<node pkg="f1tenth_racecar" type=노드이름 name=파일이름 />
```
[원본 코드](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/launch/includes/common/joy_teleop.launch.xml#L16)

위 형식을 맞춰서 추가해주면 됩니다.


## LIDAR 연결
```xml
<param name="ip_address" type="string" value=ip주소/>
```
[원본 코드](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/launch/includes/common/sensors.launch.xml#L5)

Hokuyo 10LX 라이다의 경우 이더넷을 통해 연결하게 되는데, 여기서 라이다의 ip 주소를 각자 알맞게 넣어주면 됩니다. 보통은 `192.168.0.10`입니다.

### 추가적인 정보
[이 코드는 원래 `2`로 설정되어 있었지만 컨트롤러의 설정이 달라서인지 `3`으로 해야 오른쪽 스틱을 활용한 조종이 가능해졌습니다.](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/config/joy_teleop.yaml#L35)

[이 코드는 저희가 사용했던 주행 알고리즘에 C++17에만 있는 함수가 필요해서 따로 넣었는데 필요없으시다면 빼셔도 됩니다.](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/CMakeLists.txt#L4)

[이 코드에는 원래 `-`가 없었지만 주행해보니 차가 반대로 움직여서 넣어주게 되었습니다.](https://github.com/Creative-Autonomous-Tractor/f1tenth_system/blob/6190b45897889e07dd640896fb96aed02ceec86d/f1tenth_racecar/config/vesc.yaml#L3)

