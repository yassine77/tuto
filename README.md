# Sensorlab
## Brief
This document is a tutorial that describes how to use **Sensorlab-cli**, the command line interface used to control the nodes remotely.
## Important
1. For this laptop, the admin's username is **lora** and the password is **Nougat26***.
2. All directories related to the Sensorlab project, are located in "/home/lora/sensorlab/" , thus, every file's relative path is specified from this location. 
eg. if the relative path to a file named "lora.c" is "./loradir/lora.c", the absolute path is "/home/lora/sensorlab/loradir/lora.c".
3. Three open terminals are needed, for the VPN connection, the connection as a subscriber to the MQTT broker, and for the command line interface **sensorlab-cli**. They have to be kept open.  


## Connect as a client to the OpenVpn Server  

Click the network manager icon in the top menu bar and select **Connexions VPN** then **sensorlab**:

![alt text](/home/lora/Bureau/readme/vpn1.png)

and 

![alt text](/home/lora/Bureau/readme/vpn2.png)

## Connection to the broker as a subscriber
In the second terminal, you connect to the broker to be able to see the messages sent by the observer to the broker:
```bash
cd ~/sensorlab/sensorlab-server/
python server.py --broker_address broker.local --broker_port 1883
```
**Warning:** You have to leave this terminal open until the end of the experiment !

## Sensorlab-cli
### Notations
1. Lines that begins with > are commands, the next one without > is the status, displayed automatically after a command.
2. ID = ID of case, eg. for the node 12, the command "node status <ID>" should be "node status 12".
3. More information on the **sensorlab-cli** can be found on the [Github repository](https://github.com/Orange-OpenSource/sensorlab-observer).   
Now, that you are connected to the Sensorlab Network (VPN), you are able to manipulate the nodes.  
To start the command line interface, run in the third terminal :
```bash
sensorlab-cli
```
To list all the nodes on the network and displays the IDs of the nodes or cases (malettes) :
```bash
>list
12 , 13 , 14 , 2
```
The next step is to setup the node controller and serial drivers :
```bash
>node setup /home/lora/sensorlab/sensorlab-node/nucleolora/profiles/hdlc/nucleolora-profile.tar.gz <ID>
observer<ID> device:configured serial:hdlc
```

### Scenario Mode
Setup the IO Module to connect the observer to the broker:
```bash
>io setup experiment broker.local 1883 60 <ID>
observer <ID> state:ready broker: broker.local:1883 source:experiment
```
Setup an experiment scenario by loading the experiment archive (in this exemple, I use an archive that runs the node for 10 min):
```bash
>experiment setup scenario /home/lora/sensorlab/sensorlab-experiment/loranucleo-behaviors/loramac-10min/<experiment-archive> <ID>
observer<ID> behavior:scenario state:ready remaining/duration : undefined/600
```
PS: <experiment-archive> is the name of the chosen archive depending on the activation mode (ABP or OTAA). 

Start the IO module:
```bash
>io start <ID>
observer<ID> state:connecting broker:broker.local:1883 source:experiment
```
and the experiment:
```bash
>experiment start <ID>
observer<ID> behavior:scenario state:running remaining/duration:0:10:00/600
```
Now the experiment and the node are running, and we can see the remaining time until the end of the experiment:
```bash
>experiment status <ID>
observer<ID> behavior:scenario state:running remaining/duration:0:07:43/600
```
Of course, the experiment can always be stopped before its end, by running
```bash
>experiement stop <ID>
```
then 
```bash
>io stop <ID>
```
## Wireshark
By the end of the experiment, a .pcap file is generated, which, after opening it in wireshark, you can see all downlink and uplink frames of each node.  
To open this file in wireshark, open a new terminal and:
```bash
wireshark ~/sensorlab/sensorlab-server/sensorlab.pcap
```

## JSON file
It is possible to generate a JSON file from the previous PCAP:
```bash
cp ~/sensorlab/sensorlab-server/sensorlab.pcap ~/sensorlab/sensorlab-parser/pcap/
cd ~/sensorlab/sensorlab-parser/
node TreatPcapFile.js
```
The generated JSON file sensorlab.json is located in _outputJson_ directory (~/sensorlab/sensorlab-parser/outputJson).
