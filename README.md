# new_auv

```bash

ros2 run rmw_zenoh_cpp rmw_zenohd

./build_base.bash

 ./build_auip.bash


sudo systemctl restart auip_zenoh_router.service


apt search zenoh
mb2167@markus:~/auip/auip/ui$ docker compose up

AUIP
192.168.2.72
osllab
frontier@auip-jetson:~/auip/auip/camera_synchroniser$ docker compose up

```
```bash
ros2 topic pub /auip/pid/dp_joy geometry_msgs/msg/TwistStamped "{header: {stamp: {sec: 0, nanosec: 0}, frame_id: 'auip_base_link'}, twist: {linear: {x: 0.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.0}}}"
```

```bash
# Bridge between ROS 2/DDS and Zenoh
RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-zenoh-bridge-dds \
 && rm -rf /var/lib/apt/lists/*
```


```bash
ros2 run zenoh_bridge_dds zenoh-bridge-dds
```
