# Fork of https://github.com/gudovskiy/yoloNCS
##YOLO for Intel/Movidius Neural Compute Stick (NCS) demo in MapR environment
Run yolo with NCS stick within VirtualBox and stream detected objects to MapR stream

## VirtualBox setup
* Ubuntu 16.04 TLS Desktop : https://www.ubuntu.com/download/desktop/thank-you?country=FR&version=16.04.4&architecture=amd64
* VirtualBox 5.2.6 for Mac : https://download.virtualbox.org/virtualbox/5.2.8/VirtualBox-5.2.8-121009-OSX.dmg
* VB extension pack : https://download.virtualbox.org/virtualbox/5.2.8/Oracle_VM_VirtualBox_Extension_Pack-5.2.8.vbox-extpack

## VM specs
  Ubuntu 64 bits
  4096MB RAM
  2 x vCPU
  1 x 25GB SATA HDD (VDI)
  Audio disabled
  Network : 1 x NAT + 1 x Host-only
  USB ports setup : Ports : USB 3.0 xHCI
    ![](/images/vb-ports-usb.png | width=80)
  1st filter : USB2 Movidius 03e7 (vendor ID 03e7)
    ![](/images/vb-ports-usb-filter-03e7.png | width=80)
  2nd filter : USB3 Movidius 040e (vendor ID 040e)
    ![](/images/vb-ports-usb-filter-040e.png | width=80)

## After installing Ubuntu
  root@mapr:~# sudo su -
  root@mapr:~# apt-get install git
  root@mapr:~# mkdir -p ~/workspace
  root@mapr:~# cd ~/workspace
  root@mapr:~/workspace# git clone https://github.com/movidius/ncsdk.git
  root@mapr:~/workspace# git clone https://github.com/kromozome2003/MapR-YoloNCS.git
  root@mapr:~/workspace# cd ~/workspace/ncsdk
  root@mapr:~/workspace/ncsdk# make install

## Verify USB stock is present
  root@mapr:~/workspace/MapR-YoloNCS# dmesg | grep -i movidius
  root@mapr:~/workspace/MapR-YoloNCS# lsusb | grep -i 03e7
  root@mapr:~/workspace/ncsdk# usb-devices | grep -i 03e7

    ![](/images/troubleshoot-usb.png | width=80)

## Compilation
* From your VM : Compile .prototxt and corresponding .caffemodel (with the same name) to get NCS graph file.
  root@mapr:~/workspace/ncsdk# cd ~/workspace/MapR-YoloNCS
  root@mapr:~/workspace/MapR-YoloNCS# mkdir -p weights

* From your laptop (not the VM) Download Pretrained Caffe Models to ./weights/
  Download YOLO_tiny: https://drive.google.com/file/d/0Bzy9LxvTYIgKNFEzOEdaZ3U0Nms/view?usp=sharing
  root@kromozome2003:~/Downloads# scp ~/Downloads/yolo_tiny.caffemodel root@192.168.56.101:/root/workspace/MapR-YoloNCS/weights/

* Check the file is at the right location
  root@mapr:~/workspace/MapR-YoloNCS# ls -l weights/

* Compile the model
  root@mapr:~/workspace/MapR-YoloNCS# mvNCCompile prototxt/yolo_tiny_deploy.prototxt -w weights/yolo_tiny.caffemodel -s 12
  root@mapr:~/workspace/MapR-YoloNCS# ls -l ~/workspace/MapR-YoloNCS/graph

  ![](/images/compile-model.png | width=80)

## Single Image Script
* Run "yolo_example.py" to process a single image.
* For example: "python3 py_examples/yolo_example.py images/dog.jpg" to get detections as below.

  ![](/images/yolo_dog.png | width=80)

## Try with your webcam (live detection)
* Add your webcam to VirtualBox : Devices->Webcams->Facetime HD Camera

  ![](/images/webcam-vb.png | width=80)

* Run "object_detection_app.py" to process a videos from your camera.
* For example: "python3 py_examples/object_detection_app.py" to get camera detections as below.
* Modify script arguments if needed.
* Press "q" to exit app.

![](/images/camera.png | width=80)

## Sending detected objects to Kafka stream on MapR platform
* python3 py_examples/object_detection_app.py -g http://<USER>:<PASS>@<IP_ADDR>:8082/topics/ -s <STREAM_PATH> -t <TOPIC>
* Modify script arguments axxording to your MapR environment.
* Press "q" to exit app.

![](/images/mapr-stream-consumer.png | width=80)
