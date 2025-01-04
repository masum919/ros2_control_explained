# ros2_control_explained
The infamous ros2_control is a framework in ROS2 that facilitates the management and control of a robot’s hardware in a standardized and modular way. The purpose is to simplify integrating new hardware into ROS2 applications by separating controllers (software side) from different systems, actuators and sensors (hardware side). We can think of it as a bridge that connects the software and hardware by translating high-level commands (like robot motion control commands) into low-level motor signals. It also reads feedback from the robot hardware and sends it back to the software.
![Diagram1](https://github.com/user-attachments/assets/ea479bb6-e2e2-4fd9-ad8c-1d3b28e3ac10)
Suppose we have a robotic arm with 3 revolute joints. And we want to:

- Move the arm to a specific position (like moving the end-effector to a desired position)
- Use feedback from the arm joints to improve the position accuracy of the end-effector
![Diagram2](https://github.com/user-attachments/assets/0c5d162b-8457-4311-a778-feee45828f6f)

Each joint of the robot arm is driven by a motor, but different motors require different methods for communications. Example:

- DC motors: controlled using PWM and possibly a motor driver chip
- Servo motors: controlled via position commands
- Stepper motors: controlled by sending a specific number of step impulses

Also, motors can be controlled via serial or CAN bus for controlling speed, position, torque etc.

**Without ros2_control**, we’d have to

- Understand the protocol for each motor type and handle their differences
- Write low-level software to send commands to each motor
- Write custom control logic to move each joint accurately based on high level commands. And this typically involves- calculating the required motor effort (velocity/torque) and adjusting the motor effort based on a feedback loop (like a PID controller). And, these motor effort values need conversion based on motor type- PWM for DC motors, step pulse frequency for stepper motors, CAN position command for servo motors etc.
- We’d need to write codes to read data from different sensors and handle them appropriately. Then we’d have to sync the sensor data with motor control commands
- We’d need to handle feedback loop for each joint. The control loops must run independently for each joint, read current effort value, compare it to the target effort value and then calculate and apply the required motor effort. This means managing multiple threads or processes. And we must synchronize the joints if they need to move together (to follow a trajectory). Handling and debugging multi-threaded control loops could quickly turn into a complicated and a royal pain.

**With ros2_control**, we do not need to worry about most of the above.

- The hardware-interface handles communications with motors and sensors
- Controllers like joint_trajectory_controller handle PID control for us. Also, sensor feedback is integrated into the control loop
- The framework automatically handles the control loop for each joint
- The ros2_control comes with many different and widely used controllers that can interpolate and synchronize motor efforts for us

Now for our 3 joint robot arm, we can update our Diagram 1 like below:
![Diagram3](https://github.com/user-attachments/assets/267d3b4f-fd37-49b8-a06f-436c9c123721)
The building blocks for controlling our robot arm are:

- **Hardware interface:** This is like a translator between our code and physical robot. It knows how to talk to the specific motors and sensors in our robotic arm. This means it knows how many motors and sensors the robot has, and it knows exactly how to send voltage commands to each motor and read from each sensor.
- **Controller manager:** This is like the brain of the operation. It manages different controllers and ensures they don’t conflict. It can switch between controllers smoothly (like switching from position control to torque control).
- **Controllers:** These are specialized workers, each with a specific job. For a robotic arm joint we could use- joint position controller, velocity controller, torque controller etc.

The diagram below shows how different components of ros2_control framework work together in to move our robotic arm
![Diagram4](https://github.com/user-attachments/assets/97cabf61-aa31-4085-9949-937a8c58928a)
***Now let’s say we want to swap out the 3 joint robot arm with a two wheel differential drive robot. It means we want to change the hardware. Do we write whole new software from scratch to control our new hardware? NO.! HECK NO!***

The ros2_control framework allows us to implement hardware abstraction through its hardware interface. This hardware_interface talks to the robot hardware (does not matter how the robot looks like) by representing the hardware through two interfaces.

- **Command interface:** These are the interfaces we can **control** such as different motor efforts (velocity/position/torque)
- **State interface:** These are the interfaces we only **observe** or **monitor** such as sensor reading.
![Diagram5](https://github.com/user-attachments/assets/70d87404-2bec-4804-8eef-183cc64a08e2)

The benefits of such abstraction are:

- Same code works with different robots as the robots are recognized through only command and state interfaces
- We can easily change the robot hardware by simply updating the URDF 
- We can easily switch between the real and simulated robot
- We can focus on the application logic and not worry about the hardware details and the communication methods

Two very important components for robot abstractions are:

- **URDF:** This is the detailed blueprint of the robot. It describes how many joints and their types, and how many links exist and how they are connected. It also describes what sensors and actuators are present
- **YAML configuration:** This is like a settings menu for the robot. It defines what controllers are available, controller parameters, control loop rates etc.

Let’s see a more detailed diagram of this framework when we swap our robot arm with any other type of robot.
![Diagram6](https://github.com/user-attachments/assets/8d842534-255c-450e-a15b-1ee40f08b1bf)
![Diagram7](https://github.com/user-attachments/assets/4e5d4a28-0ab1-41fb-a4e3-46cd240853e2)
We should know that each hardware interface is created as a plugin. The plugins register themselves with the pluginlib. The controller manager uses pluginlib to find and load appropriate plugins. These plugins can be loaded dynamically meaning they can be loaded at runtime (while the program is running). The beauty of using pluginlib in ros2_control is that it makes the system incredibly flexible while maintaining robustness. We can add support for new robots by creating new plugins, we can easily update or debug features without rebuilding the entire system, we can switch between real and simulated hardware. All without changing the application code!

So, ros2_control is basically like using the same standard gamepad console to play all kinds of games! No need to change your console for different games!

**--Written by Masum**  
*Date: January 4, 2025*
