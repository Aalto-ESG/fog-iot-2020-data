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
| 1000             | 600                          |
| 5000             | 200                          |
| 10000            | 100                          |

## Dataset structure

The dataset is divided in to three groups: Metadata, sensors, state

* Metadata: Contains general information about the simulation scenario in JSON format
    * Information about sensors
    * ID for each simulation actor (human, vehicles and sensors)
        * ID is used to identify the actors in the "state" group of the dataset

* Sensors: Contains data from all sensors for all simulation frames

* State: Contains information about the physics state for all simulation frames
    * Vehicle/Human locations
    * Vehicle/Human rotations

## Dataset reading example

TODO