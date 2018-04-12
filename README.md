# Fork of https://github.com/gudovskiy/yoloNCS
YOLO for Intel/Movidius Neural Compute Stick (NCS) demo in MapR environment
Run yolo with NCS stick within VirtualBox and stream detected objects to MapR stream

## VirtualBox setup
Ubuntu 16.04 TLS Desktop : https://www.ubuntu.com/download/desktop/thank-you?country=FR&version=16.04.4&architecture=amd64
VirtualBox 5.2.6 for Mac : https://download.virtualbox.org/virtualbox/5.2.8/VirtualBox-5.2.8-121009-OSX.dmg
VB extension pack : https://download.virtualbox.org/virtualbox/5.2.8/Oracle_VM_VirtualBox_Extension_Pack-5.2.8.vbox-extpack

## VM specs
* Ubuntu 64 bits
* 4096MB RAM
* 2 x vCPU
* 1 x 25GB SATA HDD (VDI)
* Audio disabled
* Network : 1 x NAT + 1 x Host-only
* USB ports setup : Ports : USB 3.0 xHCI
* 1st filter : USB2 Movidius 03e7 (vendor ID 03e7)
* 2nd filter : USB3 Movidius 040e (vendor ID 040e)

## After installing Ubuntu
    sudo su -
    apt-get install git
    mkdir -p ~/workspace
    cd ~/workspace
    git clone https://github.com/movidius/ncsdk.git
    git clone https://github.com/kromozome2003/yoloNCS.git
    cd ~/workspace/ncsdk
    make install

## Download Pretrained Caffe Models to ./weights/
YOLO_tiny: https://drive.google.com/file/d/0Bzy9LxvTYIgKNFEzOEdaZ3U0Nms/view?usp=sharing

## Compilation
* Compile .prototxt and corresponding .caffemodel (with the same name) to get NCS graph file.
* For example: "mvNCCompile prototxt/yolo_tiny_deploy.prototxt -w weights/yolo_tiny_deploy.caffemodel -s 12"
* The compiled binary file "graph" has to be in main folder after this step.

## Verify USB stock is present
root@mapr:~/workspace/ncsdk# lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 023: ID 03e7:2150  
Bus 001 Device 002: ID 80ee:0021 VirtualBox USB Tablet
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

root@mapr:~/workspace/ncsdk# usb-devices
T:  Bus=01 Lev=01 Prnt=01 Port=01 Cnt=02 Dev#= 23 Spd=480 MxCh= 0
D:  Ver= 2.00 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
P:  Vendor=03e7 ProdID=2150 Rev=00.01
S:  Manufacturer=Movidius Ltd.
S:  Product=Movidius MA2X5X
S:  SerialNumber=03e72150
C:  #Ifs= 1 Cfg#= 1 Atr=80 MxPwr=500mA
I:  If#= 0 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=11 Prot=ff Driver=(none)

## Single Image Script
* Run "yolo_example.py" to process a single image.
* For example: "python3 py_examples/yolo_example.py images/dog.jpg" to get detections as below.

![](/images/yolo_dog.png)

## Camera Input Script
* Run "object_detection_app.py" to process a videos from your camera.
* For example: "python3 py_examples/object_detection_app.py" to get camera detections as below.
* Modify script arguments if needed.
* Press "q" to exit app.

![](/images/camera.png)

## Try with your webcam
On VirtualBox : Devices->Webcams->Facetime HD Camera

## Sending detected objects to Kafka stream on MapR platform
* python3 py_examples/object_detection_app.py -g http://<USER>:<PASS>@<IP_ADDR>:8082/topics/ -s <STREAM_PATH> -t <TOPIC>
* Modify script arguments axxording to your MapR environment.
* Press "q" to exit app.
