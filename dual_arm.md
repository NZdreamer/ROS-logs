# dual_arm

## ur_robot_driver

#### `dual_ur_bringup_test.launch`

位置：ur_robot_driver包的launch文件夹下

**left_arm: ** ur5

* robot_ip: 192.168.168.20.128

* reverse_port: 50001

* script_sender_port: 50002

* controller_config_file: $(find ur_robot_driver)/config/ur5_controllers.yaml

* robot_description_file: $(find ur_description)/launch/ur5_upload.launch

* kinematics_config: $(find ur_description)/config/my_left_arm_calibration.yaml

* include file: $(find ur_robot_driver)/launch/ur5_bringup.launch



**right_arm: ** ur5

* robot_ip: 192.168.168.20.127

* reverse_port: 50003

* script_sender_port: 50004

* controller_config_file: $(find ur_robot_driver)/config/ur5_controllers.yaml

* robot_description_file: $(find ur_description)/launch/ur5_upload.launch

* kinematics_config: $(find ur_description)/config/my_right_arm_calibration.yaml

* include file: $(find ur_robot_driver)/launch/ur5_bringup.launch



## moveit



#### `dual_arm_moveit_planning_execution.launch`

位置：dual_ur_moveit_config包的launch文件夹下

**left_arm:** $(find ur5_moveit_config)/launch/move_group.launch

**right_arm: **$(find ur5_e_moveit_config)/launch/move_group.launch