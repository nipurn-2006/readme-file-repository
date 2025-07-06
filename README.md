
# FULL AUTO STEERING 

### shebang-->
ensure that the file operates in python 3

### imports statements -->
- ROS imports -> For node, topics and custom messages 
queue->For taking average of velocity and omega commands explained later 
copy->For making deep and shallow copy of lists 
operator.add->For element wise addition 

### Initialisation of class named as Drive ->
- self.init_dir-> An list initialised by 1 which can be changed to -1 if that wheel is required to move in opposite direction 
- self.max_steer_pwm->A maximum pwm value is set which is later utilised in pid controller in steer function 
- An node named as drive arc is created 

### Subscriptions -->
- this node is subscribing to the topics joy,enc_auto(This is required for manual mode )
- Further it also subscribes to /motion and rot (For autonomous motion )

### Publishers -->
- The node is publsihing to motor_pwm and state 
- Here state is a boolean value which is 0 for manual mode and 1 for autonomous mode 

### INT32MultiArray
- INT32MultiArray is a message type which consists of data as well as a layout 
- layout basically takes care of the format of the array of data including the label,data offset and size and stride.
- Layout is not considered necessary but its advised to use it 

### joy data format-->
It returns two arrays 
1-buttons is a int array which consisits of either 0 or 1
2-axes is a float array which consists of values ranging from -1 to +1

Initialisations and designation of various buttons and axes to their rrespective indexes -->
here the button and axes are then designated to various indexes to be returned from the joy data 
like forward button is named as 4 which means its data can be read from 4 index of the buttons array of the message received from joy callback.(IF ITS 1 ,IT MEANS THAT THE BUTTON IS BEING PRESSED AND IF ITS 0 IT MEANS IT IS NOT BEING PRESSED )

There are two important modes which control the entire logic of the code 
one is steer_isLocked and the other is full_potential_isLocked 
Depending on whether botha ere true or which one of them is true or false the axes definition changes which can be seen later in the code 

Initially the state is initialised as False  for manual mode 
Moroever both the steering_isLocked and full_potential_isLocked are initailised to true and rotinplace is initialised as false


Joycallback --->>
the modes available for the rover are 0,1,2,3,4.
On pressing mode up button the mode rises by 1 and on pressing mode down button the mode decreases by 1 
The higher the mode Higher is the speed of the rover 

## Steering and Drive Modes

| Mode # | `steer_islocked` | `full_potential_islocked` | Mode Name           | Joystick Controls Used                                                                                                    | Control Variables Initialized                                                                                                 | Description                                                                                                    |
|--------|------------------|--------------------------|---------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| 1      | True             | True                     | Locked Steering     | Buttons: `forward_btn`, `parallel_btn`, `rotinplace_btn` <br> Axes: `fb_axis`, `lr_axis`, axis 3                         | `steering_ctrl_locked = [forward_btn, parallel_btn, rotinplace_btn]` <br> `drive_ctrl = [fb_axis, -lr_axis]` <br> `curve_opp_str = axis 3` | Normal driving mode.<br>Steering is preset (all wheels together), drive is via joystick axes.                  |
| 2      | False            | True                     | Unlocked Steering   | Buttons: `forward_btn`, `parallel_btn` <br> Axes: `steer_samedir_axis`, `steer_oppdir_axis`                              | `steering_ctrl_unlocked = [forward_btn, parallel_btn]` <br> `steering_ctrl_pwm = [steer_samedir_axis, steer_oppdir_axis]`    | Advanced steering mode.<br>Operator can steer all wheels together or in opposite directions using joystick.     |
| 3      | True             | False                    | Individual Steering | Axes: `fl_wheel_axis`, `fr_wheel_axis`, `bl_wheel_axis`, `br_wheel_axis`                                                 | `full_potential_pwm = [fl_wheel_axis, fr_wheel_axis, bl_wheel_axis, br_wheel_axis]`                                          | Individual wheel steering.<br>Operator can control each wheel’s steering angle separately.                      |
| 4      | False            | False                    | (Unused/Invalid)    | —                                                                                                                        | —                                                                                                                            | This mode is not entered by design.                                                                            |

## Control Variables Summary

| Variable Name             | Used In Mode(s)         | Description                                                                                  |
|--------------------------|-------------------------|----------------------------------------------------------------------------------------------|
| `steering_ctrl_locked`   | 1 (Locked Steering)     | `[forward_btn, parallel_btn, rotinplace_btn]` — preset steering actions via buttons          |
| `drive_ctrl`             | 1 (Locked Steering)     | `[fb_axis, -lr_axis]` — forward/back and left/right drive via axes                          |
| `curve_opp_str`          | 1 (Locked Steering)     | `axis 3` — for curving with steering                                                        |
| `steering_ctrl_unlocked` | 2 (Unlocked Steering)   | `[forward_btn, parallel_btn]` — for relative 45-degree or other advanced steering functions |
| `steering_ctrl_pwm`      | 2 (Unlocked Steering)   | `[steer_samedir_axis, steer_oppdir_axis]` — for PWM-based steering control                  |
| `full_potential_pwm`     | 3 (Individual Steering) | `[fl_wheel_axis, fr_wheel_axis, bl_wheel_axis, br_wheel_axis]` — individual wheel steering  |
