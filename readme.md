# ROS 2 Kinova Gen3 — Robotics Laboratory

This repository provides a Docker-based environment for working with the **Kinova Gen3 6-DOF robotic arm and Robotiq 2F-85 gripper** using ROS 2 Humble, MoveIt 2 and RViz.

The environment uses simulated (fake) hardware, allowing students to develop and test robot control algorithms without access to a physical robotic arm.

## 1. Prerequisites

Before starting, make sure you have installed the following software on your computer. You will have already installed these if you have been following along in the labs. 

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) — to run the ROS 2 environment.
- [Git](https://git-scm.com/downloads) — to clone the repository.
- [Visual Studio Code](https://code.visualstudio.com/) — recommended for editing Python files and laboratory exercises.

**Ensure Docker Desktop is running and your xServer before proceeding.**

**Note:** The container uses ROS 2 Humble and includes the required Kinova packages. You do not need to install ROS 2 directly on your computer.

## 2. Clone the repository

Open a terminal (or PowerShell on Windows), navigate to a suitable directory and run:

```bash
git clone https://github.com/MaxwellHogan/ROS2_kinova.git
```

Navigate into the repository:

```bash
cd ROS2_kinova
```

This directory will serve as your local workspace containing the Docker configuration and laboratory exercises.

## 3. Configure your workspace path

**Important: Complete this step before running the Docker container startup script.**

The Docker container requires access to the files you cloned onto your computer. This is accomplished using a Docker bind mount.

Open the provided container startup script found in the "scripts" directory in VS Code and locate the workspace volume mapping. It will contain a Docker volume argument similar to:

```bash
-v "<PATH_TO_YOUR_REPOSITORY>:/ros2_ws"
```

Replace `<PATH_TO_YOUR_REPOSITORY>` with the **absolute path** to the `ROS2_kinova/workspace` directory on your computer.

For example, on Windows:

```text
-v "C:\Users\Student\Documents\ROS2_kinova\workspace:/ros2_ws"
```

Or on MacOS:

```text
-v "/home/student/ROS2_kinova/workspace:/ros2_ws"
```

The directory on the left is the folder on your computer. The directory on the right (`/ros2_ws`) is the location inside the Docker container.

Save the script after making the change.

**Why is this necessary?**

The bind mount allows you to edit exercise files in VS Code on your computer and run them inside the ROS 2 Docker environment. Changes made to files in the mounted workspace are reflected on both sides.

Be careful not to change the `/ros2_ws` destination unless specifically instructed.

## 4. Build the Docker image

From the root of the cloned repository, run:

```bash
docker build --no-cache -t ros2_kinova -f docker/Dockerfile.kinova .
```

Docker will download and install the required ROS 2 packages, dependencies and Kinova configuration.

The initial build may take several minutes depending on your computer and internet connection.

When the build completes successfully, confirm that the image exists:

```bash
docker images
```

Look for the `ros2_kinova` image.

You do not need to rebuild the image every time you change your Python exercise files.

## 5. Start the Docker container

Run the container startup script supplied with the repository, after completing the workspace-path configuration in Section 3.

The script is responsible for creating the Docker container, mounting your workspace and making the ROS 2 environment available.

On windows the command to run the script is:
```bash
.\scripts\run_container.bat
```

You are going to need to attach multiple terminals to the docker container to do the exercises, you use this command to do this:
```bash
docker exec -it ros2_dev_container bash 
```


Once inside the container, verify the workspace:

```bash
pwd
```

The expected directory is:

```text
/ros2_ws
```

You can also check that ROS 2 is installed:

```bash
ros2 --help
```

## 6. Launch the Kinova robotic arm

From a terminal inside the Docker container, run:

```bash
ros2 launch kinova_gen3_6dof_robotiq_2f_85_moveit_config \
  robot.launch.py \
  robot_ip:=0.0.0.0 \
  use_fake_hardware:=true \
  use_sim_time:=false \
  launch_rviz:=true
```

This starts:

- The Kinova Gen3 robot description.
- The ROS 2 controller manager and simulated hardware interfaces.
- The joint trajectory and Robotiq gripper controllers.
- MoveIt 2 for robot motion planning.
- RViz for visualisation.

**This setup does not require Gazebo or a physical robot.** The fake-hardware interfaces provide joint-state updates as commands are executed, but do not simulate realistic physical dynamics.

You can run this setup with gazebo by running the following two commands in parallel - by opening a second terminal connected to the docker container to run the second command. 

Run the Gazebo
```bash
ros2 launch kortex_bringup kortex_sim_control.launch.py \
    robot_type:=gen3 \
    dof:=6 \
    sim_ignition:=true \
    sim_gazebo:=false \
    use_sim_time:=true \
    launch_rviz:=false
```

Run Movit interface + rviz
```bash 
ros2 launch kinova_gen3_6dof_robotiq_2f_85_moveit_config sim.launch.py \
    use_sim_time:=true \
    launch_rviz:=true
```

**While completing the laboratory exercises you must have a moveit interface running, you can use the gazebo if your pc can handle it otherwise the hardware sim will suffice.**


## 7. Verify the robot is working

When RViz opens, you should see the Kinova Gen3 robotic arm with its Robotiq gripper.

### Test the arm

In RViz:

1. Open the **MotionPlanning** panel.
2. Select the `manipulator` planning group.
3. Choose a valid goal configuration.
4. Set the start state to the robot's current state.
5. Click **Plan** to calculate a trajectory.
6. Click **Execute** to move the simulated robot.

You should see the arm move to the selected configuration.

### Test the gripper

Select the `gripper` planning group and try the available open and closed configurations.

Alternatively, open another terminal inside the container and command the gripper directly:

```bash
ros2 action send_goal \
  /robotiq_gripper_controller/gripper_cmd \
  control_msgs/action/GripperCommand \
  "{command: {position: 0.0, max_effort: 50.0}}"
```

This opens the gripper.

To close the gripper:

```bash
ros2 action send_goal \
  /robotiq_gripper_controller/gripper_cmd \
  control_msgs/action/GripperCommand \
  "{command: {position: 0.79, max_effort: 50.0}}"
```

You should observe the gripper fingers moving in RViz.

If you subsequently plan a gripper trajectory in RViz, update the planning start state to the current position first.

## 8. Laboratory exercises

## Exercise 1 — Explore the ROS 2 system (10 min)

```bash
ros2 node list
ros2 topic list
ros2 topic list -t
ros2 action list -t
ros2 control list_controllers
```

**Record:**
- One node involved in simulation or robot control: __________
- The topic that provides joint state feedback: __________
- The active controller used to move the arm: __________

**Question:** What is the difference between a topic and an action?

## Exercise 2 — Read joint data (10 min)

```bash
ros2 topic echo /joint_states --once
ros2 topic echo /joint_states --field name --once
ros2 topic echo /joint_states --field position --once
ros2 topic echo /joint_states --field velocity --once
ros2 topic echo /joint_states --field effort --once
ros2 interface show sensor_msgs/msg/JointState
```

Press **Ctrl+C** to stop any continuous `ros2 topic echo` command.

**Record:**
- Position of `joint_1`: __________ rad
- Position of `joint_2`: __________ rad
- Velocity of `joint_1` while stationary: __________ rad/s

**Important:** Entries in `position`, `velocity`, and `effort` correspond to the names at the *same indices* in `name`. The `name` array need not be ordered `joint_1`, `joint_2`, ... .

**Question:** Convert 0.3 rad to degrees using 180/π. Approximately how many degrees is it?

## Exercise 3 — Investigate publishers and subscribers (10 min)

```bash
ros2 topic info /joint_states --verbose
ros2 topic hz /joint_states
ros2 topic bw /joint_states
ros2 control list_hardware_interfaces
```

**Record:**
- Average joint-state publishing rate: __________ Hz
- Approximate bandwidth: __________
- Number of publishers on `/joint_states`: __________
- One hardware command interface: __________

**Question:** Why is frequent feedback important when the arm moves?

## Exercise 4 — Send a joint position target (10 min)

Move **joint 1** to **0.3 rad** over **4 seconds**:

```bash
ros2 topic pub --once \
  /joint_trajectory_controller/joint_trajectory \
  trajectory_msgs/msg/JointTrajectory \
  '{
    joint_names: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6],
    points: [{
      positions: [0.3, 0, 0, 0, 0, 0],
      time_from_start: {sec: 4}
    }]
  }'
```

Check the result:

```bash
ros2 topic echo /joint_states --once
```

Return all six joints to zero:

```bash
ros2 topic pub --once \
  /joint_trajectory_controller/joint_trajectory \
  trajectory_msgs/msg/JointTrajectory \
  '{
    joint_names: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6],
    points: [{
      positions: [0, 0, 0, 0, 0, 0],
      time_from_start: {sec: 4}
    }]
  }'
```

**Challenge:** Change the first target to **−0.4 rad**. Which part of the command changed? Return to zero afterwards.

## Exercise 5 — Move several joints (10 min)

```bash
ros2 topic pub --once \
  /joint_trajectory_controller/joint_trajectory \
  trajectory_msgs/msg/JointTrajectory \
  '{
    joint_names: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6],
    points: [{
      positions: [0.3, 0.2, -0.2, 0, 0, 0],
      time_from_start: {sec: 5}
    }]
  }'
```

**Challenges:**
1. Change the duration from 5 to 10 seconds. What happens to the speed?
2. Command a small angle (for example 0.15 rad) at joint 4 while setting the others to zero.
3. Return the arm to its zero configuration using the command in Exercise 4.

## Exercise 6 — Trajectory through several waypoints (10 min)

```bash
ros2 topic pub --once \
  /joint_trajectory_controller/joint_trajectory \
  trajectory_msgs/msg/JointTrajectory \
  '{
    joint_names: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6],
    points: [
      {positions: [0.2, 0, 0, 0, 0, 0], time_from_start: {sec: 3}},
      {positions: [0.2, 0.2, -0.2, 0, 0, 0], time_from_start: {sec: 6}},
      {positions: [0, 0, 0, 0, 0, 0], time_from_start: {sec: 9}}
    ]
  }'
```

**Question:** Are these end-effector XYZ waypoints or joint-angle waypoints? What would be required to command an XYZ target?

## Exercise 7 — Inspect trajectory tracking (10 min)

First check which controller-state topic exists:

```bash
ros2 topic list | grep joint_trajectory_controller
```

For the controller in this lab, inspect its state:

```bash
ros2 topic echo /joint_trajectory_controller/controller_state --once
```

If available, compare the `desired`, `actual`, and `error` fields while executing a longer, small motion. Controller state fields contain arrays in the order described by the controller-state `joint_names` field.

Inspect its message type:

```bash
ros2 topic info /joint_trajectory_controller/controller_state
ros2 interface show control_msgs/msg/JointTrajectoryControllerState
```

**Question:** What is the difference between a commanded joint position and an actual measured position?

## Exercise 8 — Find the XYZ position of the end effector (10 min)

```bash
ros2 topic list | grep tf
ros2 run tf2_ros tf2_echo base_link end_effector_link
```

The **translation** field is the end-effector **(x, y, z)** position, in metres, relative to `base_link` (if the named frames are available in your running robot description). Stop the command with **Ctrl+C**.

**Record:**
- End-effector XYZ at the zero configuration: __________ m
- End-effector XYZ after moving joint 1: __________ m

**Question:** How can a joint-space rotation produce a change in Cartesian position?

## Optional extension — A ROS 2 action instead of a topic

The controller also exposes a `FollowJointTrajectory` action, which provides a goal result and optional feedback:

```bash
ros2 action list -t
ros2 action info /joint_trajectory_controller/follow_joint_trajectory
ros2 interface show control_msgs/action/FollowJointTrajectory
```

To send a small joint motion as an action:

```bash
ros2 action send_goal --feedback \
  /joint_trajectory_controller/follow_joint_trajectory \
  control_msgs/action/FollowJointTrajectory \
  '{
    trajectory: {
      joint_names: [joint_1, joint_2, joint_3, joint_4, joint_5, joint_6],
      points: [{positions: [0, 0, 0, 0, 0, 0], time_from_start: {sec: 4}}]
    }
  }'
```

**Question:** Why might an action be more useful than a topic when a task needs to report success or failure?

## 9. Troubleshooting

**Docker cannot find the workspace directory**

Check that the workspace path in your startup script points to the directory containing the cloned repository.

**RViz does not open**

Check that Docker Desktop is running and that the container startup script supports graphical applications on your operating system.

**MoveIt plans a trajectory but execution fails**

Check the active controllers:

```bash
ros2 control list_controllers
```

The `joint_trajectory_controller`, `joint_state_broadcaster` and `robotiq_gripper_controller` should be active.

Also ensure the planned starting position matches the robot's current state.

**Robot joint states are not updating**

Check the joint-state topic:

```bash
ros2 topic echo /joint_states --once
```

**Python cannot import ROS 2 packages**

Ensure you are executing Python inside the Docker container and that the ROS 2 environment has been sourced.

---

## Software

This environment uses ROS 2 Humble, MoveIt 2, `ros2_control`, the Kinova Kortex ROS 2 packages and RViz.

The Docker image includes local compatibility modifications to the installed Kinova/Robotiq Xacro configuration to support fake-hardware gripper control.

The laboratory environment is intended for teaching, simulation and development rather than direct deployment to a physical robot.
