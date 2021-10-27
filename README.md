# Ouster OS-1 16 Lidar Interfacing

### Model number: 992005000729

---

### References:

##### [Original repository](https://github.com/ouster-lidar/ouster_example)

##### [Ouster download center](https://ouster.com/downloads/) - You can find the latest firmware image, Ouster Studio and manuals here.
##### [Software user manual](https://data.ouster.io/downloads/software-user-manual/software-user-manual-v2.1.x.pdf?__hstc=34987006.8e77f6e9ca644d455b29f7bbde0c3016.1633439072096.1633439072096.1633442069302.2&__hssc=34987006.1.1633442069302&__hsfp=3100135829)

---

### Connecting the LIDAR:
> What's in the box:
1. Ethernet cable
2. The LIDAR
3. The interfacing board

> What more we need:
1. PC
2. 24V power adapter

> One-time network settings to do:
1. Set the wired connection to `Link-Local Only`
2. Disable the system firewall:
    ```bash
    # Disabling
    sudo ufw disable
    # Checking status
    sudo ufw status
    # Should show:-
    # Status: inactive
    ```
3. Install `ethtool` (generally comes pre-installed):
    ```bash
    sudo apt install ethtool
    ```
4. Increase maximum transmitted units:
    This was most probably the main problem with UDP packet length. 
    ```bash
    sudo ifconfig enp3so mtu 9000
    ```

> Note: 

Throught this document, I am referring to `enp3s0` for the Ethernet IP address. This may vary from system to system.
To check for your system:
```bash
ifconfig
``` 
Check the one labelled Ethernet and keep note of it's `inet` address.

---

### Checking the connection:
Open the local product dashboard on a browser.
`http://os1-992005000729.local/`

If it's up and running the LIDAR was successfully connected. You can check out diagnostics and more information on the LIDAR here.

We can also initiate factory reset of the LIDAR in this dashboard itself.

> Updating the firmware:

First visit the [Ouster download center](https://ouster.com/downloads/) and download the latest firmware image for the LIDAR. 
Then open up the dashboard while the LIDAR is connected. Here you will see the option to upload the firmware image and begin the Update. 

---

### Checking if it works on Ouster Studio:
Download the SDK from [Ouster download center](https://ouster.com/downloads/) and extract and run it. 

It didn't work on Ubuntu, so its better to do it on Windows itself.

After the installation is done, launch the software while disabling your Wifi. You can try the `Scan for connected devices` feature. If it doesn't work, try finding the IP address of the interfacing board(shows up on Wireshark) and do the manual connection.

You can now see the pointcloud data being visualized. This is only to make sure that there are no hardware issues. 

---

### Running the sample visualizer and client on Ubuntu:
These are just the non ROS counterparts provided by Ouster on their [Original repository](https://github.com/ouster-lidar/ouster_example). 
Download the latest stable release and not just clone the main branch.

Go through the instructions as it is on the repository.

> Building on Linux:

Go into the release parent folder. Then:
```bash
sudo apt install build-essential cmake libglfw3-dev libglew-dev libeigen3-dev \
     libjsoncpp-dev libtclap-dev
```
```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release <path to ouster_example>
make
```
where `<path to ouster_example>` is the location of the ouster_example source directory.


> Running the smaple client:

Make sure the sensor is connected to the network. Navigate to `ouster_client` under the `build` directory, which should contain an executable named `ouster_client_example`.

This program will attempt to connect to the sensor, capture lidar data, and write point clouds out to CSV files, located in the `ouster_client` directory:
```bash
./ouster_client_example <sensor hostname> <udp data destination>
```
**Note:**

Here and going forward,

`<sensor hostname>` : `os1-992005000729.local` or the IP address of the interfacing board. 


`<udp data destination>` : The IP address of the ethernet port through which the LIDAR is connected. Here `enp3s0`. 


> Running sample visualizer:

Make sure the sensor is connected to the network. Navigate to `ouster_viz` under the `build` directory, which should contain an executable named `simple_viz`.
```bash
./simple_viz <sensor hostname> <udp data destination>
```
This should open up a visualizer where you can look at the poincloud. 

---

### Interfacing with ROS:
The sample code include tools for publishing sensor data as standard ROS topics. 
I am assuming you have ROS already installed on your PC. 

> First install some build dependencies:

```bash
sudo apt install build-essential cmake libglfw3-dev libglew-dev libeigen3-dev \
     libjsoncpp-dev libtclap-dev
```
You also need these dependencies pre-installed:
- pcl-ros
- tf2
- rviz

> To build:

Don't build the ROS workspace inside the repo directory as ROS build system will be confused by the other material.
```bash
mkdir -p ./myworkspace/src
cd myworkspace
ln -s <path to ouster_example> ./src/
catkin_make -DCMAKE_BUILD_TYPE=Release
```

> Running the example node:

Make sure the sensor is connected to the network. Make a JSON file first for the metadata. 

```bash
roslaunch ouster_ros ouster.launch sensor_hostname:=<sensor hostname> \
                                   metadata:=<path to metadata json>
```

Additional arguments(insert after metadata):
- udp_dest:=<hostname> to specify the hostname or IP to which the sensor should send data
- lidar_mode:=<mode> where mode is one of 512x10, 512x20, 1024x10, 1024x20, or 2048x10, and
- viz:=true to visualize the sensor output, if you have the rviz ROS package installed

> Example:
```bash
roslaunch ouster_ros ouster.launch sensor_hostname:=os1-992005000729.local \
                                   metadata:=./meta.json udp_dest:=169.254.175.88 lidar_mode:=1024x10 viz:=true
```

You can check the topics for the data published and also [record and replay data](https://github.com/ouster-lidar/ouster_example#recording-data) with bag files. 


--- 


### Topics being published:

- `/os1_node/imu_packets` and `/os1_node/lidar_packets` aren't being published.

- Empty for now:

    `/clicked_point`

    `/initialpose`

    `/move_base_simple/goal`

    `/tf`



- Image topics:

    `/img_node/nearir_image`

    `/img_node/range_image`

    `/img_node/reflec_image`

    `/img_node/signal_image`

- IMU:

    `/os_cloud_node/imu` - Well structured.

    `/os_node/imu_packets`

- Point cloud:

    `/os_cloud_node/points`

    `/os_node/lidar_packets` - Lesser data in comparision. 


- Static transform: `/tf_static` 

---


