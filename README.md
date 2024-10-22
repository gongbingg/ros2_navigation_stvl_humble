Navigation and 3D Mapping with ROS2
=======================
*University of Genoa A.Y. 2022/23, MSc Robotics Engineering*<br><br>
This package aims at analysing the functionalities provided by [ROS2 Navigation2](https://navigation.ros.org/) combined with the [Spatio-Temporal Voxel Layer (STVL)](https://github.com/SteveMacenski/spatio_temporal_voxel_layer) plugin for sensor fusion and mapping.<br>
The whole package is based on [Linorobot2 Package](https://github.com/linorobot/linorobot2).
This software component has been developed by the following contributors:
- Michele Pestarino (michele.pestarino@outlook.it)
- Thomas Campagnolo (thomas.campagnolo24@gmail.com)
- Federico Sacco (fedezac99@gmail.com)

![STVL Example](https://user-images.githubusercontent.com/82339529/174105565-8df041fd-6025-4307-88d0-d761695f1766.png)
 

Installation
------------
In order to run properly this package the following dependencies are needed.<br>
First of all, make sure to have [ROS2 Galactic Desktop Version](https://docs.ros.org/en/galactic/Installation.html) and the [STVL](https://github.com/SteveMacenski/spatio_temporal_voxel_layer) plugin installed, as well as the [Navigation2](https://navigation.ros.org/). Then, clone inside the selected workspace our [package](https://github.com/mic-pest/SofAr_assignment).<br> 
Firstly, build the workspace and source it:

```bash
cd <your_ros_ws>
rosdep update && rosdep install --from-path src --ignore-src -y --skip-keys microxrcedds_agent --skip-keys micro_ros_agent
colcon build
source install/setup.bash
```

Then, export the currently supported Linorobot robot version, being it the 2 wheel drive base:

```bash
echo "export LINOROBOT2_BASE=2wd" >> ~/.bashrc
source ~/.bashrc
```
Please refer to [Linorobot2 Package](https://github.com/linorobot/linorobot2) for the any other dependency/issue.<br>
Several different maps to test this package on, can be downloaded from [Gazebo Models Worlds Collection Repository](https://github.com/chaolmu/gazebo_models_worlds_collection). Clone it, then copy the `models` folder's content of this package into the `models` folder of the `.gazebo` hidden repository, which can be found at the path `/home/<username>`.<br>
The step is required to launch the simulation with the provided `office_earthquake` map, more in the following.

How To Run
----------
Open two terminals and in one of them run the Gazebo instance

```bash
ros2 launch linorobot2_gazebo gazebo.launch.py
```

In the second one run the navigation package of Linorobot2. 

```bash
ros2 launch linorobot2_navigation navigation.launch.py

ros2 run rqt_tf_tree rqt_tf_tree --force-discover

```
![gif_example](https://user-images.githubusercontent.com/74416077/201300168-8ff7bc5b-3b74-4317-88b4-7e2dc5559e31.gif)

Now, in RViz select *2D pose estimate* and configure it as similar as possible to the initial pose of Linorobot2, which is depicted in Gazebo. You should now be able to see the robot model, the laser scan and the current voxel grid.<br>
Next, in RViz select *Nav2 Goal* and configure to whatever position in the map, in order to see the robot moving, while also rendering the voxel grid. Both voxel color and decay time can be customized in simulation using the *Color Transformer* and *Decay Time* fields, in the left RViz tab. <br>

A known issue is the voxel rendering on systems with some graphic optimization carried out by the Intel or NVIDIA graphic hardware. A popular solution is the following:
```bash
export LIBGL_ALWAYS_SOFTWARE=1
```
If this fix does not solve the problem, probably a change in the settings is required: check [here](http://wiki.ros.org/rviz/Tutorials/Rviz%20in%20Stereo) for further tips.

Available Plug&Play Setups 
--------------------------
### STVL Observation Sources
Two different sensor configurations are available in this package to build the voxel grid: 3D LiDAR and twin RGB-D cameras (one on the front and one on the back).<br>
In order to change the current observation source from which STVL creates the voxel grid, modify in the `navigation.launch.py` file, available in the `linorobot2_navigation` package, the `SENSOR` global variable, by initializing it to `3d` or `rgbd`. Before running again the launch file rebuild the workspace with `colcon build`, as done before.<br>

<p align="center">
  <img src="https://user-images.githubusercontent.com/82339529/173870511-e750a589-7e66-495f-afd1-5799667f23de.jpg" />
</p>

### Maps
Two different environments have already been mapped and are ready to be used: `office_earthquake` (indoor) and `playground` (outdoor).<br>
To change the selected map, modify in the `navigation.launch.py` file, available in the `linorobot2_navigation` package, the `MAP_NAME` global variable, by initializing it to `office_earthquake` or `playground`, in order to visualize in RViz the corresponding GridMap.<br>
Then, to also change the visualized map in Gazebo, select the corresponding world name by changing the value of the `MAP` global variable, in the `gazebo.launch.py` file in package `linorobot2_gazebo`. Before running again the launch file rebuild the workspace with `colcon build`, as done before.<br>

![Office Hearthquake Gazebo Map](https://user-images.githubusercontent.com/82339529/173866774-51625d56-c2be-4642-983c-247862742e7b.jpg)

![Playground Gazebo Map](https://user-images.githubusercontent.com/82339529/173867291-64808577-2824-4f80-835c-38d5c0f6e45b.png)


Want to add something?
----------------------
### Add a sensor
Write or import the `xacro` file of the selected sensors, then put it in the `urdf/sensors` folder of the `linorobot2_description` package. Make sure that the sensor's file is compliant with the ROS2 plugins/requirements available in the [Gazebo Wiki](https://github.com/ros-simulation/gazebo_ros_pkgs/wiki), and that it produces either `LaserScan` or `PointCloud2` datatypes, which, at the moment, are the only types compatible with STVL.

### Stick it on Linorobot2
#### Laser Sensor
Following the Linorobot2 package convention, firstly modify the `generic_laser.urdf.xacro`, which can be found in the folder `urdf/sensors`of `linorobot2_description` package, by importing the sensor's xacro, then adding its block and passing to it the correct parameters (e.g. update_rate, min_angle, etc.).

```xml
<xacro:include filename="$(find linorobot2_description)/urdf/sensors/laser_3d.urdf.xacro" />
.
.
.
<xacro:macro name="generic_laser_3d" params="*origin">
  <xacro:laser_3d
    update_rate="10"
    ray_count="360"
    min_angle="-3.1416"
    max_angle="3.1416"
    min_range="0.21"
    max_range="5.5"
    frame_id="laser_360"
    topic_name="agv/pointcloud"
  >
    <xacro:insert_block name="origin" />
</xacro:laser_3d>
</xacro:macro>
```

Then, in `2wd.urdf.xacro`, still in `linorobot2_description` package, `urdf/robots` folder, add the above define object, by referencing it via its name. Make sure that `generic_laser.urdf.xacro` is already present in the various imports.
```xml
<xacro:include filename="$(find linorobot2_description)/urdf/sensors/generic_laser.urdf.xacro" />
.
. 
. 
<xacro:generic_laser_3d>
  <xacro:insert_block name="laser_3d_pose" />
</xacro:generic_laser_3d>
```

Finally, define the actual sensor pose in the robot model's properties, by changing the `2wd_properties.urdf.xacro`, in `linorobot2_description` package, `urdf` folder.

```xml
<xacro:property name="laser_3d_pose">
  <origin xyz="0.12 0 0.50" rpy="0 0 0"/>
</xacro:property> 
```

#### RGB-D Camera
Similarly, import the sensor's xacro file in `2wd.urdf.xacro`, still in `linorobot2_description` package, `urdf/robots` folder, add the depth sensor, by referencing it via its name and passing its parameters.

```xml
<xacro:include filename="$(find linorobot2_description)/urdf/sensors/depth_sensor.urdf.xacro" />
.
.
.
<xacro:depth_sensor
  camera_link="camera_link1" 
  camera_to_base_link="camera_to_base_link1" 
  camera_depth_link="camera_depth_link1" 
  camera_depth_joint="camera_depth_joint1" 
  camera_name="camera_name1" 
  custom_namespace="custom_namespace1" 
  camera_controller="camera_controller1"
>
  <xacro:insert_block name="depth_sensor_pose1"/>
</xacro:depth_sensor>
```
Finally, define the actual depth camera pose in the robot model's properties, by changing the `2wd_properties.urdf.xacro`, in `linorobot2_description` package, `urdf` folder.

```xml
<xacro:property name="depth_sensor_pose1">
  <origin xyz="0.14 0.0 0.045" rpy="0 0 0"/>
</xacro:property>
```

### STVL Settings
In order to build the voxel grid with the correct sensor data modify the `navigation.yaml` parameter file, which can be found in the `config` folder of the `linorobot2_navigation` package. <br>
STVL can be added to both `global_costmap` and/or `local_costmap`. However, attention has to be paid to the `plugins` array order: the STVL has to come before the inflation layer so that the last one can inflate obstacles present in the voxel grid. 

```yaml
plugins: ["static_layer", "obstacle_layer", "stvl_layer", "inflation_layer"]
```

Then, add the `stvl_layer` entry, after the `plugins` field (order does not matter anymore).<br>
Some setups are already present in the [Example folder of STVL Repo](https://github.com/SteveMacenski/spatio_temporal_voxel_layer/tree/ros2/example) and they can be easily customized in order to be compliant with the added sensor. Pay closer attention to the following fields:

```yaml
plugin: "spatio_temporal_voxel_layer/SpatioTemporalVoxelLayer" # This is the complete name of the plugin
voxel_decay: 15.0  # seconds if linear, e^n if exponential. Time before voxels' decay
decay_model: -1 # For decay time extended to infinity (voxel are persistent), 0 for linear decay, 1 for exponential decay
publish_voxel_map: true  # Set it to true in order to make it visible in RViz
observation_sources: rgbd1_mark rgbd1_clear rgbd2_mark rgbd2_clear # List of all sensors available for voxel grid creation, two entries for each one of them, one for voxel marking and one for voxel clearing
rgbd1_mark: # Name of the observation source
  data_type: PointCloud2 # Data type produced by the sensor
  topic: /custom_namespace1/camera/depth/color/points # Topic to which the sensor publishes to
  marking: true # True if its a marking observation source
  clearing: false # True if its a clearing observation source
  min_obstacle_height: 0.05 # Minimum obstacle height (to be checked on the selected map)
  max_obstacle_height: 10.0 # Maximum obstacle height (to be checked on the selected map)
  inf_is_valid: false # Default is false, for laser scans
rgbd1_clear: # Name of the observation source
  data_type: PointCloud2 # Data type produced by the sensor
  topic: /custom_namespace1/camera/depth/color/points # Topic to which the sensor publishes to
  marking: true # True if its a marking observation source
  clearing: false # True if its a clearing observation source
  vertical_fov_angle: 0.8745  # Vertical field of view angle of the sensor, in radians
  horizontal_fov_angle: 1.50098 # Horizontal field of view angle of the sensor, in radians
  decay_acceleration: 5.0 # In 1/s^2. If the laser scanner MUST be 0
  model_type: 0 # 0=depth camera, 1=3d lidar
```

STVL allows for the fusion of different observation sources when creating the voxel grid: each one of them has to be specified in `observation_sources` and the plugin will manage by itself the underlying sensor fusion.

### Work on a new map
Initially, the `new_map.world` file has to be stored in the `worlds` folder in `linorobot2_gazebo` package, then modify the `gazebo.launch.py` file in that package at line 36.

```python
world_path = PathJoinSubstitution(
  [FindPackageShare("linorobot2_gazebo"), "worlds", "new_map.world"]
)
```

After that, the new world file has to be mapped via the slam feature of Linorobot2, available with `slam.launch.py` in the `linorobot2_navigation` package: open three terminals, in each of them launch, respectively, `gazebo.launch.py`, `slam.launch.py` and run a teleoperation node.

```bash
ros2 launch linorobot2_gazebo gazebo.launch.py
ros2 launch linorobot2_navigation slam.launch.py
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Drive the robot around the environment, until a fine enough map has been created. Finally, save the map using `nav2_map_server` saver node: this will cause the generation of a yaml file and a pgm picture.

```bash
cd linorobot2/linorobot2_navigation/maps
ros2 run nav2_map_server map_saver_cli -f <map_name> --ros-args -p save_map_timeout:=10000.0
```

To change the working environment, refer to the Maps section of the Available Plug&Play Setups chapter. Pay attention to the both map and worlds files' name.


Theory Concepts
----------------------
### Navigation 2
The Navigation 2 package allows to move a mobile robot safely from the initial position to the goal position and can also be applied in other applications that involve the navigation of the robot, such as following dynamic points.

Nav2 uses Behavior Trees (BT) to call several modular servers to complete an action. An action can consist of calculating a path, checking effort, recovery or any other navigation-related action.
These are independent nodes that communicate with the Behavior Tree (BT) on a ROS action server. The following diagram represents the structure of Nav2.

![Nav2 Structure](https://user-images.githubusercontent.com/82339529/173867968-ed07e595-3628-4562-b677-856c13903c89.png)

The possible independent servers and plugins available are:
- **Map Server**, to load and store maps
- **AMCL**, to localize the robot in the map
- **Nav2 Planner**, to plan a path from A to B around the obstacle inside the environment
- **Nav2 Smoother**, to smooth out the planned trajectories and make navigation more continuous and feasible
- **Nav2 Costmap 2D**, to convert sensor data into a costmap representation of the world
- **Nav2 Recoveries**, to compute recovery behaviors in case of some failure
- **Nav2 Core**, plugins to enable your own custom algorithms and behaviors

More information can be found in [Navigation2](https://navigation.ros.org/).

### Spatio-Temporal Voxel Layer (STVL)
This package, in replacement for the standard voxel grid representation of the environment, is a Navigation2 plugin that efficiently maintains temporal 3D sparse volumetric voxel grid with decay, through sensor models.

#### Spatio-
The Spatio in this package is the representation of the environment in a configurable voxel size and general appearance in RViz.
In addition, it is possible to configure the minimum and maximum height values to perceive obstacles and the range within which the objects are voxelized.
One of the key features of the spatio is that there are no constraints for a maximum number of voxels.

#### -Temporal
The Temporal in this package allows you to introduce the concept of voxel decay time. With dedicated servers it is possible to store the times in each voxel after which the voxel will disappear from the map, thus going to implement graphic optimizations.

#### Local Costmap
The Local Costmap uses all information from the robot before the voxel decay time for the local cost map. It is based on the user configuration of the layer to have a short decay time of the voxels (5-15 seconds) in order to plan only in relative space. It is the user's responsibility to choose a decay time that makes sense to the robot's local planner.

#### Global Costmap
With this package, it is certainly possible not to decay voxels in the global map. In the practical application a time of 15-45 seconds is a good balance due to the things moving in the scene (e.g. shop, warehouse, building area, office, etc.) and also permanent voxels set decay to -1, not recommended to use for planar lidars.

#### Mapping
The Mapping package can be used to map an environment in 3D in real-time by enabling the mapping mode, keeping the entire voxel grid and saving the map using the services provided.
You can run multiple instances - one for mapping and one for navigation - if you want to navigate while mapping your environment. The mapping will also save incremental maps in the startup directory, displayed using `vdb_viewer`. The cost map and occupation point clouds will not be generated in this mode.

More information can be found in [Spatio-Temporal Voxel Layer (STVL)](https://github.com/SteveMacenski/spatio_temporal_voxel_layer) GitHub repository.
