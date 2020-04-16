# fog-iot-2020-data
LiDAR data supplement for our [Fog-IoT 2020 workshop paper](http://dx.doi.org/10.4230/OASIcs.Fog-IoT.2020.4).

This repository contains LiDAR data collected from a virtual warehouse.

## Repository structure
The dataset is divided into four categories depending on the amount of robots in the simulation.
Folder `robots-1` contains LiDAR data from one robot, while `robots-6`contains data from six robots.

Each folder has three versions of the dataset, divided by the amount of LiDAR points collected per robot per simulation frame (i.e., one simulation timestep). For example, a file named `points-per-frame-1000.hdf5` contains 1000 3D-points for each LiDAR sensor on each frame.

To avoid unnecessarily large datasets, the amount of frames depends on the amount of LiDAR points per frame. More frames equals smoother simulation. The amount of frames in each dataset is shown in the table below.

| Points per frame | Simulation duration (frames) |
|:----------------:|:----------------------------:|
| 1000             | 1200                          |
| 5000             | 400                          |
| 10000            | 200                          |

## Dataset structure

The dataset is divided in to three groups: Metadata, sensors, state

* Metadata: Contains general information about the simulation scenario in JSON format
    * Information about sensors
    * ID for each simulation actor (human, vehicles and sensors)
        * ID is used to identify the actors in the "state" group of the dataset

* Sensors: Contains data from all sensors for all simulation frames

* State: Contains information about the physics state for all simulation frames
    * Actor locations
        * Vectors with x, y, z values in meters. Z-axis points upwards to the ceiling.
    * Actor rotations
        * Vectors with pitch, yaw, roll rotations in degrees.
    


## Dataset reading example

The hdf5 dataset files are easy to read with Python, using the H5py library.
You can find some reading examples below:

#### Example: basics
```python
import h5py
import json
import numpy as np

# Dataset files can be read with the H5py library
data = h5py.File("robots-4/points-per-frame-1000.hdf5", 'r')

# Dataset can be easily traversed
# For example, we can print the groups (keys) in the dataset:
keys = data.keys()
print("Dataset.keys(): " + str(keys))

# Then we can print the groups of a subgroup
keys = data["sensors"].keys()
print("Dataset['sensors'].keys(): " + str(keys))  # Print dataset groups below sensors

# We can access sensor data at any specific moment (frame) in time
frame = 680
sensor_data = data["sensors/robot_1"]
print("Robot_1 lidar data shape: " + str(sensor_data.shape))
print("Robot_1 lidar data shape (single frame): " + str(sensor_data[frame].shape))

# We can read the metadata as JSON:
metadata = json.loads(data["metadata"][()])
```
The output of these prints is:
```
Dataset.keys(): <KeysViewHDF5 ['metadata', 'sensors', 'state']>
Dataset['sensors'].keys(): <KeysViewHDF5 ['robot_1', 'robot_2', 'robot_3', 'robot_4']>
Robot_1 lidar data shape: (1200, 1000, 3)
Robot_1 lidar data shape (single frame): (1000, 3)
Metadata keys: [..., 'robot_1', 'robot_2', 'robot_3', 'robot_4', ...]
```

#### Example: Lidar data and coordinate system conversions
Next example shows how to read lidar data and how to convert in from the local (sensor) coordinate system to
global coordinate system.

```python
# Each sensor has its own entry in the metadata
actor_id = metadata["robot_1"]["id"]  # Fetch ID for lidar of robot_1
print("Robot_1 actor ID: " + str(actor_id))

# Each actor has a unique ID, which can be used to find its index in the dataset
index = list(data["state/id"][frame]).index(actor_id)  # NOTE: All frames should contain the same ID

# With the actor ID and index, we can access the rotation and location of that specific actor
# The rotation and location of the lidar sensor is needed to translate the 3D points to world space
rotation = data["state/rotation"][frame][index]
location = data["state/location"][frame][index]
# Due to a bug in the simulation software, all lidar sensor data is rotated by extra -90 degrees around yaw
rotation[1] += 90  # Fix rotation bug by adding +90 degrees to yaw
print("Sensor rotation:  ({:.2f}, {:.2f}, {:.2f}) ".format(*rotation))
print("Sensor location:  ({:.2f}, {:.2f}, {:.2f}) ".format(*location))


# Next we will read lidar data and move it from the local coordinate system to the global coordinate system
# To do this, we have to rotate and translate the lidar data.

# Lets fetch lidar data from robot_1 at a specific simulation frame
local_space_lidar = data["sensors/robot_1"][frame]
print("Lidar data shape: " + str(local_space_lidar.shape))

# In order to rotate 3d points, we need to create a rotation matrix (https://en.wikipedia.org/wiki/Rotation_matrix)
yaw = rotation[1]  # NOTE: Remember the bug mentioned above! Extra +90 degrees had to be added to yaw!
yaw = np.deg2rad(yaw)  # Convert degrees to radians
rotation_matrix = np.array(((np.cos(yaw), -np.sin(yaw), 0),
                            (np.sin(yaw), np.cos(yaw), 0),
                            (0, 0, 1)))


# Now we are ready to convert the points from the sensor coordinate system to the global coordinate system
# As an example, lets first try to convert the first lidar point
first_point = local_space_lidar[0]
first_point_rotated = np.dot(rotation_matrix, local_space_lidar[0])
first_point_translated = first_point_rotated + location
print("First lidar point:  ({:.2f}, {:.2f}, {:.2f}) ".format(*first_point))
print("First lidar point rotated: ({:.2f}, {:.2f}, {:.2f}) ".format(*first_point_rotated))
print("First lidar point rotated and translated: ({:.2f}, {:.2f}, {:.2f}) ".format(*first_point_translated))

# Finally, lets move all lidar points from local (sensor) space to world (global) space
# In order to do this, we need to first rotate all points to match the global coordinate system
# Then we need to translate the points by adding the location of the sensor
world_space_lidar = np.empty(local_space_lidar.shape)  # Preallocate array
for i in range(local_space_lidar.shape[0]):
    world_space_lidar[i] = np.dot(rotation_matrix, local_space_lidar[i])  # Rotate
    world_space_lidar[i] += location  # Translate
```

Output:

```
Robot_1 actor ID: 30
Sensor rotation:  (0.00, -90.00, 0.00) 
Sensor location:  (8.07, 27.50, 2.00) 
Lidar data shape: (1000, 3)
First lidar point:  (0.00, -8.07, -0.00) 
First lidar point rotated: (-8.07, -0.00, 0.00) 
First lidar point rotated and translated: (-0.00, 27.50, 2.00) 
```

That's it! Now the lidar data is located in the global coordinate system can be used for any desired applications.