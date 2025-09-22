# Rename: Pick and Place Tutorial

## Pick and Place

In MoveIt, grasping is done using the MoveGroup interface. To grasp an object you must build a `moveit_msgs::Grasp` message defining all poses and postures involved in the grasp.

Watch this video to see the output of this tutorial:  
https://youtu.be/QBJPxx_63Bs?si=rAOBOgCP4Iy57UAo

---

## Getting Started

If you haven’t already done so, complete the steps in the MoveIt Getting Started guide:  
https://moveit.github.io/moveit_tutorials/doc/getting_started/getting_started.html

---

## Running The Demo

1. Open two terminals.

2. In the first terminal, launch RViz and load the Panda demo:
   ```bash
   roslaunch panda_moveit_config demo.launch
3. In the second terminal, run the pick and place tutorial:
   ```bash
   rosrun moveit_tutorials pick_place_tutorial
   
Understanding moveit_msgs::Grasp
For full documentation refer to the message spec: http://docs.ros.org/en/noetic/api/moveit_msgs/html/msg/Grasp.html

The key fields are:

trajectory_msgs/JointTrajectory pre_grasp_posture Joint positions of the end effector before moving in to grasp.

trajectory_msgs/JointTrajectory grasp_posture Joint positions for the actual grasp.

geometry_msgs/PoseStamped grasp_pose The pose of the end effector when attempting the grasp.

moveit_msgs/GripperTranslation pre_grasp_approach Direction and distance for the approach to the object.

moveit_msgs/GripperTranslation post_grasp_retreat Direction and distance to move once the object is grasped.

moveit_msgs/GripperTranslation post_place_retreat Direction and distance after placing the object.

Creating the Environment
Create a vector of three collision objects:
std::vector<moveit_msgs::CollisionObject> collision_objects;
collision_objects.resize(3);
