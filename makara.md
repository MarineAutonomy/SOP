# Makara SOP 
This file contains the standard operating procedures for Makara boat.
## EAP Connection

SSH through the External Access Point (EAP)

Connect to the wifi network MAKARA (password:```password```)

### On Ubuntu:
 
1. Open WiFi settings
2. Go to IPv4 tab
3. Select Manual
4. Put your desired IP (example : 192.168.0.15 except 192.168.0.10) 
5. Put Subnet mask 255.255.0.0 
6. Put gateway 192.168.0.1 
7. Toggle the automatic DNS switch twice


### On Windows:

1. Open Wifi Network Properties through Network and Sharing center (through Control Panel on windows). 
2. Click Properties and select “Internet Protocol Version 4 (TCP/IPv4)”
3. Select Properties and select the radio to enter your IP address manually
4. Put your desired IP (example : 192.168.0.15 except 192.168.0.10) 
5. Put Subnet mask 255.255.0.0 
6. Put gateway 192.168.0.1 
7. Put primary DNS as 192.168.0.1

### SSH into Pi

SSH into the Raspberry Pi (Password: ```password```)
```
 ssh ubuntu@192.168.0.10
```


### SSH into jetson

SSH into the jetson (Password: ```nopass```)

```
ssh jetsonmakara@192.168.0.27
```


### Hotspot mode switching through yaml file (Deprecated)

Never create a copy of 50-cloud-init.yaml in nsame directory. 

For AP mode : 
Edit 50-cloud-init.yaml from etc/netplan directory , this file will create a hotspot Raspberry
50-cloud-init.yaml   ( This file is generated from information provided by the datasource.  Changes
 to it will not persist across an instance reboot.  )
 To disable cloud-init's
 network configuration capabilities, write a file
 /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
```
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.0.10/15]
      gateway4: 192.168.0.254
      nameservers:
        addresses: [192.168.0.10,8.8.8.8]
  wifis:
    wlan0:
      dhcp4: true
      access-points:
         "Raspberry":
          password: "password"
          mode: ap
         #"matsya_kcs":
         # password: "matsya_kcs"

```

```
sudo netplan generate
sudo netplan apply
```

For Wifi mode:
	Comment out mode ap and put the “matsya_kcs” and password:”matsya_kcs” above


### Commands to be run on Pi 
The following scripts will start up 2 rosserial nodes to read encoder data from rudder and proppeller. It will also start another ros node which will publish encoder data through web sockets to jetson.  

```
cd makara
./ros1_run_devdocker.sh ./raspi_entry.sh 
```

### Commands to be run on Jetson 
The  ```./ros2_simulator.sh```  will start simulator in jetson. 

```
cd makara
./ros2_simulator.sh
```


This command will show values published by the simulator into topic ``` /makara_00/imu```. These are not real world values.

```
ros2 topic echo /makara_00/imu
```

The  ```./ros2_gnc.sh```  will initialize waypoint tracking specified for waypoints in ``` inputs.yml ``` file in jetson.  The vessel rudder and propeller will run according to the values published by the simulator into the topics .

```
cd makara
./ros2_gnc.sh
```

## Possible Errors
If you see the following error on the terminal when you run ```./ros1_run_devdocker.sh ./raspi_entry.sh```:

```
[ERROR] [1721830570.453790]: Mismatched protocol version in packet (b'\xff'): lost sync or rosserial_python is from different ros release than the rosserial client
```

Solution : Unplug all the usb cables on Pi and plug in arduino and encoder usb first the plug back in other cables. 

## Other Utilities

### Odometry visualization

Run the following command on jeston terminal to see odometry data.

```
./ros2_odom.sh
```
The odometry data will be available at ```<jetson-ip-address>:8500/odometry```

### Manual stopping of rudder and propeller

If the propeller and rudder need to stopped manually from Pi, run following alias on raspi terminal
```
stopact
```

### Internet Access to Pi 

Getting internet access to raspi has been a persistent problem so far. For giving  internet access to Pi , unplug ethernet and edit yaml file for internet mode. With ethernet connected you will not be able to give internet access to pi.

Another workaround for updating files in jeston and raspi is to connect the ```matsya_kcs``` router to the wall internet port and connect another cable from the router to the network switch on the vehicle. Now you can connect your laptop to the router through WiFi. With netaccess (https://netaccess.iitm.ac.in/account/login) login you can have internet access to your laptop. 

Now you can mount the file system from jetson and raspi onto your laptop using sshfs commands below:

```
mkdir ~/raspi_mount
sshfs ubuntu@192.168.0.10:/home/ubuntu ~/raspi_mount
cd ~/raspi_mount
```

```
mkdir ~/jetson_mount
sshfs jetsonmakara@192.168.0.27:/home/jetsonmakara ~/jetson_mount
cd ~/jetson_mount
```
Once you are inside the folders, you can update the git repositories (https://github.com/MarineAutonomy/makara) as required with ```git pull``` or ```git push``` (after ```git add .``` and ```git commit -m <your-commit-message>```. As your laptop has internet access through the router, the issue of raspi or jetson having or not having access to internet becomes immaterial. 

You can also open the folder in visual code by the command ```code .``` inside these folders.

Once you're finished editing the files, you can unmount the folders with the following commands:

```
fusermount -u ~/raspi_mount
```
```
fusermount -u ~/jetson_mount
```
