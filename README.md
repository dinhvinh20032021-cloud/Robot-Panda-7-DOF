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

cpp
std::vector<moveit_msgs::CollisionObject> collision_objects;
collision_objects.resize(3);
Define table1 (where the cube starts):

cpp
collision_objects[0].id = "table1";
collision_objects[0].header.frame_id = "panda_link0";
collision_objects[0].primitives.resize(1);
collision_objects[0].primitives[0].type    = collision_objects[0].primitives[0].BOX;
collision_objects[0].primitives[0].dimensions = {0.2, 0.4, 0.4};
collision_objects[0].primitive_poses.resize(1);
collision_objects[0].primitive_poses[0].position.x = 0.5;
collision_objects[0].primitive_poses[0].position.z = 0.2;
collision_objects[0].primitive_poses[0].orientation.w = 1.0;
Define table2 (where the cube will be placed):

cpp
collision_objects[1].id = "table2";
collision_objects[1].header.frame_id = "panda_link0";
collision_objects[1].primitives.resize(1);
collision_objects[1].primitives[0].type    = collision_objects[1].primitives[0].BOX;
collision_objects[1].primitives[0].dimensions = {0.4, 0.2, 0.4};
collision_objects[1].primitive_poses.resize(1);
collision_objects[1].primitive_poses[0].position.y = 0.5;
collision_objects[1].primitive_poses[0].position.z = 0.2;
collision_objects[1].primitive_poses[1].orientation.w = 1.0;
Define the object to manipulate:

cpp
collision_objects[2].id = "object";
collision_objects[2].header.frame_id = "panda_link0";
collision_objects[2].primitives.resize(1);
collision_objects[2].primitives[0].type    = collision_objects[2].primitives[0].BOX;
collision_objects[2].primitives[0].dimensions = {0.02, 0.02, 0.2};
collision_objects[2].primitive_poses.resize(1);
collision_objects[2].primitive_poses[0].position.x = 0.5;
collision_objects[2].primitive_poses[0].position.z = 0.5;
collision_objects[2].primitive_poses[0].orientation.w = 1.0;
Pick Pipeline
Create a single grasp in a vector:

cpp
std::vector<moveit_msgs::Grasp> grasps;
grasps.resize(1);
Grasp pose (frame panda_link0, compensate end-effector palm):

cpp
grasps[0].grasp_pose.header.frame_id = "panda_link0";
tf2::Quaternion orientation;
orientation.setRPY(-tau/4, -tau/8, -tau/4);
grasps[0].grasp_pose.pose.orientation = tf2::toMsg(orientation);
grasps[0].grasp_pose.pose.position = {0.415, 0, 0.5};
Pre-grasp approach (along +X):

cpp
grasps[0].pre_grasp_approach.direction.header.frame_id = "panda_link0";
grasps[0].pre_grasp_approach.direction.vector.x = 1.0;
grasps[0].pre_grasp_approach.min_distance = 0.095;
grasps[0].pre_grasp_approach.desired_distance = 0.115;
Post-grasp retreat (along +Z):

cpp
grasps[0].post_grasp_retreat.direction.header.frame_id = "panda_link0";
grasps[0].post_grasp_retreat.direction.vector.z = 1.0;
grasps[0].post_grasp_retreat.min_distance = 0.1;
grasps[0].post_grasp_retreat.desired_distance = 0.25;
Pre-grasp posture (open gripper):

cpp
openGripper(grasps[0].pre_grasp_posture);
Grasp posture (close gripper):

cpp
closedGripper(grasps[0].grasp_posture);
Set support surface and call pick:

cpp
move_group.setSupportSurfaceName("table1");
move_group.pick("object", grasps);
Place Pipeline
Create a single place location:

cpp
std::vector<moveit_msgs::PlaceLocation> place_locations;
place_locations.resize(1);
Place pose (rotate 90° about Z):

cpp
place_locations[0].place_pose.header.frame_id = "panda_link0";
tf2::Quaternion place_orientation;
place_orientation.setRPY(0, 0, tau/4);
place_locations[0].place_pose.pose.orientation = tf2::toMsg(place_orientation);
place_locations[0].place_pose.pose.position = {0, 0.5, 0.5};
Pre-place approach (along –Z):

cpp
place_locations[0].pre_place_approach.direction.header.frame_id = "panda_link0";
place_locations[0].pre_place_approach.direction.vector.z = -1.0;
place_locations[0].pre_place_approach.min_distance = 0.095;
place_locations[0].pre_place_approach.desired_distance = 0.115;
Post-place retreat (along –Y):

cpp
place_locations[0].post_place_retreat.direction.header.frame_id = "panda_link0";
place_locations[0].post_place_retreat.direction.vector.y = -1.0;
place_locations[0].post_place_retreat.min_distance = 0.1;
place_locations[0].post_place_retreat.desired_distance = 0.25;
Post-place posture (open gripper):

cpp
openGripper(place_locations[0].post_place_posture);
Set support surface and call place:

cpp
group.setSupportSurfaceName("table2");
group.place("object", place_locations);
[&#95;{{{CITATION{{{&#95;1{](https://github.com/mustafaonurbakir/robotic_pick_place/tree/cae83e1fceee34e187693ffec9b500dbfd94f66c/src%2Fmoveit_tutorials%2Fdoc%2Fpick_place%2Fsrc%2Fpick_place_tutorial.cpp)[&#95;{{{CITATION{{{&#95;2{](https://github.com/ros-planning/moveit_tutorials/tree/482dc9db944c785870274c35223b4d06f2f0bc90/doc%2Fpick_place%2Fsrc%2Fpick_place_tutorial.cpp)
