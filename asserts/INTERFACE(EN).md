# PIPER SDK Interface Documentation

This SDK is used to receive CAN data frames and process them into custom data types.

*Note:* Some details are omitted here. The code includes detailed comments in Chinese. This document is mainly for clarifying structure.

## Interface Analysis

Explanation of initialization parameters and member variables of the `C_PiperInterface` class.

| Variable | Type   | Description                                          |
| -------- | ------ | ---------------------------------------------------- |
| can_name | str    | The name of the CAN route, indicating which CAN bus is used for sending and receiving data. |
| arm_can  | C_STD_CAN | CAN encapsulation class |

### 1. Secondary Encapsulation Classes

| Class                        | Description                                                     |
| ---------------------------- | --------------------------------------------------------------- |
| `ArmStatus`                  | Secondary encapsulation class for arm status, with a timestamp. |
| `ArmEndPose`                 | Secondary encapsulation class for arm end pose, with a timestamp. |
| `ArmJoint`                   | Secondary encapsulation class for arm joint angle feedback, with a timestamp. |
| `ArmGripper`                 | Secondary encapsulation class for arm gripper feedback, with a timestamp. |
| `ArmMotorDriverInfoHighSpd`  | Class for high-speed feedback information of the arm motor driver, with a timestamp. |
| `ArmMotorDriverInfoLowSpd`   | Class for low-speed feedback information of the arm motor driver, with a timestamp. |
| `ArmMotorAngleLimitAndMaxVel`| Feedback information class for current motor angle limits and max speed, with a timestamp. |
| `CurrentEndVelAndAccParam`   | Feedback information class for current end speed and acceleration parameters, with a timestamp. |
| `CrashProtectionLevelFeedback`| Feedback information class for collision protection level setting, with a timestamp. |
| `CurrentMotorMaxAccLimit`    | Feedback information class for the current motor maximum acceleration limit, with a timestamp. |
| `ArmJointCtrl`               | Message class for controlling arm joint angles, with a timestamp. |
| `ArmGripperCtrl`             | Message class for controlling the arm gripper, with a timestamp. |
| `ArmCtrlCode_151`            | Message receiver class for the 0x151 control instruction sent by the master arm, with a timestamp. |
| `ArmTimeStamp`               | Timestamp class for the arm, including timestamps for various feedbacks. |

For detailed explanations, refer to the code comments.

Each arm-related class includes a timestamp. For data requiring multiple CAN frames (like the six joint angles), the timestamps from these frames are collected, and the most recent timestamp is selected as the class’s timestamp.

### 2. Class Member Variables Explained

| Variable   | Type | Description                                                          |
| ---------- | ---- | -------------------------------------------------------------------- |
| `arm_can`  | `C_STD_CAN` | Instance of the class from `can_encapsulation.py`, handling hardware data reception and registering the data processing callback function. |
| `parser` | `C_PiperParserBase` | Instance of the data protocol parser class, currently using the V1 protocol version, a base class for data parsing. |
| `__arm_time_stamp` | `ArmTimeStamp` | Instantiation of the timestamp class, private to `C_PiperInterface`, representing timestamps for combined CAN frames, such as joint angles. |
| `__arm_status` | `ArmMsgStatus` | Instance of the arm status message containing arm status information. |
| `__arm_end_pose` | `ArmEndPose` | Instance of the arm end pose feedback message, containing end pose information. |
| `__arm_joint_msgs` | `ArmJoint` | Instance of arm joint angle feedback message, containing each joint’s status information. |
| `__arm_gripper_msgs` | `ArmGripper` | Instance of the arm gripper feedback message, containing each gripper’s status information. |
| `__arm_motor_info_high_spd` | `ArmMotorDriverInfoHighSpd` | Variable representing the high-speed feedback class for the arm motor. |
| `__arm_motor_info_low_spd` | `ArmMotorDriverInfoLowSpd` | Variable representing the low-speed feedback class for the arm motor. |
| `__feedback_current_motor_angle_limit_max_vel` | `ArmMotorAngleLimitAndMaxVel` | Variable for receiving feedback on current motor angle limits and max speed after actively sending a CAN frame. |
| `__feedback_current_end_vel_acc_param` | `CurrentEndVelAndAccParam` | Variable for receiving feedback on the current end velocity and acceleration parameters after actively sending a CAN frame. |
| `__feedback_crash_protection_level` | `CrashProtectionLevelFeedback` | Variable for receiving feedback on collision protection level settings after actively sending a CAN frame. |
| `__feedback_current_motor_max_acc_limit` | `CurrentMotorMaxAccLimit` | Variable for receiving feedback on the current motor max acceleration limit after actively sending a CAN frame. |
| `__arm_joint_ctrl_msgs` | `ArmJointCtrl` | Variable for receiving joint angle messages sent from the primary arm to the secondary arm. |
| `__arm_gripper_ctrl_msgs` | `ArmGripperCtrl` | Variable for receiving gripper messages sent from the primary arm to the secondary arm. |
| `__arm_ctrl_code_151` | `ArmCtrlCode_151` | Variable for receiving the 0x151 instruction message sent from the primary arm to the secondary arm. |

### 3. Function Analysis

| Function                | Type          | Description                                                                                                                   | Usage                              | Parameters                                   | Feedback                    |
|-------------------------|---------------|-------------------------------------------------------------------------------------------------------------------------------|------------------------------------|----------------------------------------------|-----------------------------|
| `ConnectPort`           | Getter Method | Function to open CAN message reading, calling the read function and creating a read thread to process messages.               | Used to open data processing thread after class instantiation. | None | None |
| `ParseCANFrame`         | Getter Method | CAN protocol parsing function, processes received CAN frames, used as a callback for receiving messages.                     | Creates local variable, calls decoding function in parser class, and updates messages if decoding succeeds. | `rx_message: Optional[can.Message]` | None |
| `GetArmStatus`          | Getter Method | Retrieves the arm's status information. Uses mutex `__arm_status_mtx` to ensure thread safety.                                | `obj.GetArmStatus()`               | None                                         | `self.__arm_status`         |
| `GetArmEndPoseMsgs`     | Getter Method | Retrieves arm end position messages. Uses mutex `__arm_end_pose_mtx` for thread safety.                                      | `obj.GetArmEndPoseMsgs()`          | None                                         | `self.__arm_end_pose`       |
| `GetArmJointMsgs`       | Getter Method | Retrieves arm joint status messages. Ensures thread safety with mutex `__arm_joint_msgs_mtx`.                                | `obj.GetArmJointMsgs()`            | None                                         | `self.__arm_joint_msgs`     |
| `GetArmGripperMsgs`     | Getter Method | Retrieves arm gripper status messages. Mutex `__arm_gripper_msgs_mtx` ensures thread safety.                                 | `obj.GetArmGripperMsgs()`          | None                                         | `self.__arm_gripper_msgs`   |
| `GetArmHighSpdInfoMsgs` | Getter Method | Retrieves arm high-speed motion information messages. Uses `__arm_motor_info_high_spd_mtx` mutex.                           | `obj.GetArmHighSpdInfoMsgs()`      | None                                         | `self.__arm_motor_info_high_spd` |
| `GetArmLowSpdInfoMsgs`  | Getter Method | Retrieves arm low-speed motion information messages. Uses `__arm_motor_info_low_spd_mtx` mutex.                              | `obj.GetArmLowSpdInfoMsgs()`       | None                                         | `self.__arm_motor_info_low_spd` |
| `GetCurrentMotorAngleLimitMaxVel` | Getter Method | Gets the current maximum angle limit for a specified motor. Uses mutex `__feedback_current_motor_angle_limit_max_vel_mtx`. | `obj.GetCurrentMotorAngleLimitMaxVel()` | None                          | `self.__feedback_current_motor_angle_limit_max_vel` |
| `GetCurrentEndVelAndAccParam` | Getter Method | Retrieves feedback on the current end speed and acceleration parameters. Ensures thread safety with `__feedback_current_end_vel_acc_param_mtx`. | `obj.GetCurrentEndVelAndAccParam()` | None | `self.__feedback_current_end_vel_acc_param` |
| `GetCrashProtectionLevelFeedback` | Getter Method | Retrieves collision protection level feedback. Ensures thread safety using `__feedback_crash_protection_level_mtx`.         | `obj.GetCrashProtectionLevelFeedback()` | None | `self.__feedback_crash_protection_level` |
| `GetCurrentMotorMaxAccLimit` | Getter Method | Retrieves feedback on the current maximum acceleration limit of the motor. Uses mutex `__feedback_current_motor_max_acc_limit_mtx`. | `obj.GetCurrentMotorMaxAccLimit()` | None | `self.__feedback_current_motor_max_acc_limit` |
| `GetArmJointCtrl` | Getter Method | Retrieves joint angle control messages sent from the primary arm to the secondary arm. Uses mutex `__arm_joint_ctrl_msgs_mtx` for thread safety. | `obj.GetArmJointCtrl()` | None | `self.__arm_joint_ctrl_msgs` |
| `GetArmGripperCtrl` | Getter Method | Retrieves gripper control messages sent from the primary arm to the secondary arm. Uses mutex `__arm_gripper_ctrl_msgs_mtx` for thread safety. | `obj.GetArmGripperCtrl()` | None | `self.__arm_gripper_ctrl_msgs` |
| `GetArmCtrlCode151` | Getter Method | Retrieves status for control code 151 of the arm. Uses mutex `__arm_ctrl_code_151_mtx` for thread safety. | `obj.GetArmCtrlCode151()` | None | `self.__arm_ctrl_code_151` |
| `UpdateArmStatus` | Getter Method | Updates the arm status. | Internal use | `msg:PiperMessage` | `self.__arm_status` |
| `UpdateArmEndPoseState` | Getter Method | Updates end pose state, with units in 0.001mm. | Internal use | `msg:PiperMessage` | `self.__arm_end_pose` |
| `UpdateArmJointState` | Getter Method | Updates joint state, with units in 0.001 degrees. | Internal use | `msg:PiperMessage` | `self.__arm_joint_msgs` |
| `UpdateArmGripperState` | Getter Method | Updates gripper state, with gripper feedback position in 0.001mm units. | Internal use | `msg:PiperMessage` | `self.__arm_gripper_msgs` |
| `UpdateDriverInfoHighSpdFeedback` | Getter Method | Updates driver information feedback (high-speed). | Internal use | `msg:PiperMessage` | `self.__arm_motor_info_high_spd` |
| `UpdateDriverInfoLowSpdFeedback` | Getter Method | Updates driver information feedback (low-speed). | Internal use | `msg:PiperMessage` | `self.__arm_motor_info_low_spd` |
| `UpdateCurrentMotorAngleLimitMaxVel` | Getter Method | Feedback for current motor angle limit/max speed. | Internal use | `msg:PiperMessage` | `self.__feedback_current_motor_angle_limit_max_vel` |
| `UpdateCurrentMotorMaxAccLimit` | Getter Method | Feedback for current motor max acceleration limit. | Internal use | `msg:PiperMessage` | `self.__feedback_current_motor_max_acc_limit` |
| `UpdateCurrentEndVelAndAccParam` | Getter Method | Feedback for current end speed/acceleration parameters. | Internal use | `msg:PiperMessage` | `self.__feedback_current_end_vel_acc_param` |
| `UpdateCrashProtectionLevelFeedback` | Getter Method | Collision protection level setting feedback instruction. | Internal use | `msg:PiperMessage` | `self.__feedback_crash_protection_level` |
| `UpdateArmJointCtrl` | Getter Method | Updates joint state (for messages sent by primary arm). | Internal use | `msg:PiperMessage` | `self.__arm_joint_ctrl_msgs` |
| `UpdateArmGripperCtrl` | Getter Method | Updates gripper state (for messages sent by primary arm). | Internal use | `msg:PiperMessage` | `self.__arm_gripper_ctrl_msgs` |
| `UpdateArmCtrlCode151` | Getter Method | Updates control instruction 151 from the primary arm. | Internal use | `msg:PiperMessage` | `self.__arm_ctrl_code_151` |
| `MotionCtrl_1` | Ctrl Method | 0x150, controls emergency stop, trajectory transmission, and drag teaching mode. | External use | `emergency_stop (uint8)`, `track_ctrl (uint8)`, `drag_teach_ctrl (uint8)` | None |
| `MotionCtrl_2` | Ctrl Method | 0x151, sets control mode, MOVE mode, and motion speed. Must be sent before any arm movement; otherwise, movement may not occur. | External use | `ctrl_mode (uint8)`, `move_mode (uint8)`, `move_spd_rate_ctrl (uint8)` | None |
| `EndPoseCtrl` | Ctrl Method | 0x152, 0x153, 0x154, controls the arm's end position and pose. Summarizes the next three functions, where x, y, z are in units of 0.001mm, and rx, ry, rz in 0.001 degrees. For instance, to move 0.3m along the x-axis, send 0.3\*100\*1000=30000. | External use | `X (float)`, `Y (float)`, `Z (float)`, `RX (float)`, `RY (float)`, `RZ (float)` | None |
| `__CartesianCtrl_XY` | Ctrl Method | Controls arm motion on the XY-axis in Cartesian coordinates. | Internal use (private) | `X (float)`, `Y (float)` | None |
| `__CartesianCtrl_ZRX` | Ctrl Method | Controls arm motion on the Z-axis and RX in Cartesian coordinates. | Internal use (private) | `Z (float)`, `RX (float)` | None |
| `__CartesianCtrl_RYRZ` | Ctrl Method | Controls arm motion on the RY and RZ axes in Cartesian coordinates. | Internal use (private) | `RY (float)`, `RZ (float)` | None |
| `JointCtrl` | Ctrl Method | 0x155, 0x156, 0x157, controls the six joints of the arm. Units are 0.001 degrees; for 90 degrees (1.57 radians) on joint 1, send 90\*1000; for radians, use 1.57\*1000\*(180/3.14). | External use | `joint_1 (float)`, ..., `joint_6 (float

| `JointCtrl` | Ctrl Method | 0x155, 0x156, 0x157, controls the six arm joints. Units are 0.001 degrees; to move joint 1 by 90 degrees (1.57 radians), send 90\*1000; for radians, use 1.57\*1000\*(180/3.14). | External use | `joint_1 (float)`, `joint_2 (float)`, `joint_3 (float)`, `joint_4 (float)`, `joint_5 (float)`, `joint_6 (float)` | None |
| `__JointCtrl_12` | Ctrl Method | Controls arm joints 1 and 2. | Internal use (private) | `joint_1 (float)`, `joint_2 (float)` | None |
| `__JointCtrl_34` | Ctrl Method | Controls arm joints 3 and 4. | Internal use (private) | `joint_3 (float)`, `joint_4 (float)` | None |
| `__JointCtrl_56` | Ctrl Method | Controls arm joints 5 and 6. | Internal use (private) | `joint_5 (float)`, `joint_6 (float)` | None |
| `GripperCtrl` | Ctrl Method | 0x159 controls arm gripper movement, compatible with AGX end-effector. Units are 0.001mm; to move 50mm, send 50\*1000=50000. | External use | `gripper_angle (int)`, `gripper_effort (int)`, `gripper_code (int)`, `set_zero (int)` | None |
| `MasterSlaveConfig` | Ctrl Method | 0x470, configures master-slave mode for arm following. | External use | `linkage_config (int)`, `feedback_offset (int)`, `ctrl_offset (int)`, `linkage_offset (int)` | None |
| `DisableArm` | Ctrl Method | 0x471, disables motor. If at non-zero position, the arm may lose support. | External use | `motor_num (int)`, `enable_flag (int)` | None |
| `EnableArm` | Ctrl Method | 0x471, enables arm motor. Motor will resist movement when powered. | External use | `motor_num (int)`, `enable_flag (int)` | None |
| `SearchMotorMaxAngleSpdAccLimit` | Ctrl Method | 0x472, queries motor angle, max speed, and max acceleration limits. | External use | `motor_num (int)`, `search_content (int)` | None |
| `MotorAngleLimitMaxSpdSet` | Ctrl Method | Sets motor angle limit and max speed. | External use | `motor_num (int)`, `max_angle_limit (float)`, `min_angle_limit (float)`, `max_joint_spd (float)` | None |
| `JointConfig` | Ctrl Method | Configures joint parameters. | External use | `joint_num (int)`, `set_zero (int)`, `acc_param_is_effective (int)`, `set_acc (int)`, `clear_err (int)` | None |
| `SetInstructionResponse` | Ctrl Method | Sets response to instructions. | External use | `instruction_index (int)`, `zero_config_success_flag (int)` | None |
| `ArmParamEnquiryAndConfig` | Ctrl Method | Queries and sets arm parameters. | External use | `param_enquiry (int)`, `param_setting (int)`, `data_feedback_0x48x (int)`, `end_load_param_setting_effective (int)`, `set_end_load (int)` | None |
| `EndSpdAndAccParamSet` | Ctrl Method | Sets end speed and acceleration parameters for the arm. | External use | `end_max_linear_vel (float)`, `end_max_angular_vel (float)`, `end_max_linear_acc (float)`, `end_max_angular_acc (float)` | None |
| `CrashProtectionConfig` | Ctrl Method | Configures collision protection level for each joint. | External use | `joint_1_protection_level (int)`, ..., `joint_6_protection_level (int)` | None |

**Note:**

All `Update` functions are interfaces for reading CAN data frames and updating parsed data into private variables. Since processed variables are stored as private, use the internal `Get` functions to retrieve values.

- All values are in raw units:
  - Arm end pose feedback: 0.001 mm
  - Joint angle: 0.001 degrees
  - Gripper travel: 0.001 mm
  - Gripper torque: 0.001 N/m

To get real values (e.g., gripper angle), divide the retrieved joint angle by 1000.