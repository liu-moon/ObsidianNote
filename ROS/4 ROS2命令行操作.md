- 显示隐藏文件
	- `ls -A`
- 递归删除
	- `rm -R test/`
`ros2 run turtlesim turtlesim_node`
	运行turtlesim功能包里面的turtlesim_node功能
`ros2 run turtlesim turtle_teleop_key`
	运行turtlesim功能包里面的turtle_teleop_key功能
`ros2 node`
	查看节点相关信息
`ros2 node list`
	查看ros2中正在运行的所有节点
`ros2 node info /turtlesim`
	查看/turtlesim的信息
`ros2 topic`
	查看话题相关信息
`ros2 topic list`
	查看正在发布和订阅的话题
`ros2 topic echo /turtle1/pose`
	打印/turtle1/pose话题信息
`ros2 topic pub —-rate 1 /turtle1/cmd_vel geometry_msgs/msg/Twist "{linear:{x:2.0,y:0.0,z:0.0},angular:{x:0.0,y:0.0,z:1.8}}"`
	发布话题数据
	`—-rate 1`：以1hz的频率发布
`ros2 service call /spawn turtlesim/srv/Spawn "{x:2,y:2,theta:0.2,name:''}"`
	产生新海龟
`ros2 bag`
	录制信息
`ros2 bag record /turtle1/cmd_vel`
	录制/turtle1/cmd_vel的数据
	数据默认保持到当前终端所在的路径
`ros2 bag play rosbag2_2022_05_24-16_21_24/`
	复现数据