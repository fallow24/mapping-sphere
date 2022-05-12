# Start the sensors

There are multiple ROS launch files that are located inside ~/catkin_ws/launch/ : 
 - imu.launch: Launches a single IMU and filters the raw data using Madgwick. If multiple IMUs are connected, this will randomly select a single one and will not use the others.
 - lidar.launch: Launches the LIVOX laserscanner and starts data transmission over LAN. Scanning frequency is also specified here. 
 - statictf...launch: Launches a ROS node that publishes the external LIDAR-inertial calibration on tf/ topic. 
 - Note: The Intel T265 tracking camera is started a little differently. See chapters below. 

## LiDAR

The LIVOX Mid-100 laserscanner device fortunatley has a ROS driver.
Its location is ~/ws_livox/, however you wont need to go there ever.
Just use the launch file as specified above.
Note that there are some scripts located in ~, each launches the laserscanner alongside with an inertial tracking method.
For example, ~/launchSimple.sh launches both the laserscanner, as well as a single IMU for inertial reference.

## Single IMU

If you take a closer look at ~/catkin_ws/launch/imu.launch, some parameters may be found.
You might replace the Madgwick filter, e.g., with a complementary filter. 
Or you could disable the usage of Magnetometer data for Yaw estimation.
 
## Multiple IMUs due to Zevering et al. 

In our lab, we developed an estimator that fuses multiple IMU measurements into a full 6DoF pose, instead of just 3D orientation.
The assumption is that the movement of the system is of spherical nature, i.e., a ball rolling around.
The basic idea is the fusion of two popular filters: Madgwick and Complementary. 
Further constraints are present in the algorithm to account for slippage and sliding effects of the sphere. 

To start the filter (it is called imuJasper), connect all three IMUs to the PI.
The serial numbers (they are written at the bottom of the IMU) must be specified in ~/catkin_ws/src/imuJasper/src/imuJasper.h .
Then, start the ROS node:
 $ rosrun imuJasper imuJasper -q -rate 125 -imuRate 250 -autogain 0.2 -z0

Alternativley, you may use ~/launchJasper.sh to launch the laserscanner alongside with the 3 IMUs and the filter.

## Visual-Inertial Tracking Camera 

I am very aware that there exist ROS wrappers for all Intel inertial-tracking cameras, even for T265.
You can find them here: https://github.com/IntelRealSense/realsense-ros
Many launchfiles and examples are given in this repository, yet they refuse to work on the current setup.
That is, a Raspberry PI 4 with an image of "ROSberry Pi" from Ubiquity Robotics. 
It is based on Ubuntu 16.04 and shipped with ROS Kinetic preinstalled.

So, as the official ROS drivers / launch files wont work, we had to come up with a fix.
It seemed that, due to a bug, the camera would loose connection and dont publish pose data after a short amount of time.
To the best of our luck, there exists "librealsense", a more low-level interface for the camera, which made the fix quite easy.
Start the T265 by connecting it to the PI and then run: 
 $ rosrun realsense_pipeline_fix auto_reconnect

Alternativley, you may use ~/launchIntel.sh to launch the laserscanner alongside with the T265.

Note: The T265 uses its own, internal IMU. 

Second Note: There still exists a bug where the camera will not connect properly in the initialisation of the ROS node. 
Be aware of any error messages and watch the ROS topic output before recording data.
If that happens, shutdown the ROS node, reconnect the USB of the camera, then restart the ROS node.

# Record data

 $ rosbag record --all --output-name=MyRecording 

Records all the topics and writes MyRecording.bag

# Transfer recordings to Processing Machine

On your Processing Machine, do the following:

 $ scp ubuntu@10.42.0.1:/path/to/bag/MyRecording.bag /target/path/on/your/PC/ 

which will copy the bagfile from the PI and save it as /target/path/on/your/PC/MyRecording.bag .

