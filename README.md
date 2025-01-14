# Introduction
The OpenVINO™ (Open visual inference and neural network optimization) toolkit provides a ROS-adaptered runtime framework of neural network which quickly deploys applications and solutions for vision inference. By leveraging Intel® OpenVINO™ toolkit and corresponding libraries, this runtime framework extends  workloads across Intel® hardware (including accelerators) and maximizes performance.
* Enables CNN-based deep learning inference at the edge
* Supports heterogeneous execution across computer vision accelerators—CPU, GPU, Intel® Movidius™ Neural Compute Stick, and FPGA—using a common API
* Speeds up time to market via a library of functions and preoptimized kernels
* Includes optimized calls for OpenCV and OpenVX*

## Design Architecture
From the view of hirarchical architecture design, the package is divided into different functional components, as shown in below picture. 

![OpenVINO_Architecture](https://github.com/intel/ros_openvino_toolkit/blob/master/data/images/design_arch.PNG "OpenVINO RunTime Architecture")

- **Intel® OpenVINO™ toolkit** is leveraged to provide deep learning basic implementation for data inference. is free software that helps developers and data scientists speed up computer vision workloads, streamline deep learning inference and deployments,
and enable easy, heterogeneous execution across Intel® platforms from edge to cloud. It helps to:
   - Increase deep learning workload performance up to 19x1 with computer vision accelerators from Intel.
   - Unleash convolutional neural network (CNN)-based deep learning inference using a common API.
   - Speed development using optimized OpenCV* and OpenVX* functions.
- **ros OpenVINO Runtime Framework** is the main body of this repo. it provides key logic implementation for pipeline lifecycle management, resource management and ROS system adapter, which extends Intel OpenVINO toolkit and libraries. Furthermore, this runtime framework provides ways to ease launching, configuration and data analytics and re-use.
- **Diversal Input resources** are the data resources to be infered and analyzed with the OpenVINO framework.
- **ROS interfaces and outputs** currently include _Topic_ and _service_. Natively, RViz output and CV image window output are also supported by refactoring topic message and inferrence results.
- **Optimized Models** provides by Model Optimizer component of Intel® OpenVINO™ toolkit. Imports trained models from various frameworks (Caffe*, Tensorflow*, MxNet*, ONNX*, Kaldi*) and converts them to a unified intermediate representation file. It also optimizes topologies through node merging, horizontal fusion, eliminating batch normalization, and quantization.It also supports graph freeze and graph summarize along with dynamic input freezing.

## Logic Flow
From the view of logic implementation, the package introduces the definitions of parameter manager, pipeline and pipeline manager. The below picture depicts how these entities co-work together when the corresponding program is launched.

![Logic_Flow](https://github.com/intel/ros_openvino_toolkit/blob/master/data/images/impletation_logic.PNG "OpenVINO RunTime Logic Flow")

Once a corresponding program is launched with a specified .yaml config file passed in the .launch file or via commandline, _**parameter manager**_ analyzes the configurations about pipeline and the whole framework, then shares the parsed configuration information with pipeline procedure. A _**pipeline instance**_ is created by following the configuration info and is added into _**pipeline manager**_ for lifecycle control and inference action triggering.

The contents in **.yaml config file** should be well structured and follow the supported rules and entity names. Please see [the configuration guidance](https://github.com/intel/ros_openvino_toolkit/blob/master/doc/YAML_CONFIGURATION_GUIDE.md) for how to create or edit the config files.

**Pipeline** fulfills the whole data handling process: initiliazing Input Component for image data gathering and formating; building up the structured inference network and passing the formatted data through the inference network; transfering the inference results and handling output, etc.

**Pipeline manager** manages all the created pipelines according to the inference requests or external demands (say, system exception, resource limitation, or end user's operation). Because of co-working with resource management and being aware of the whole framework, it covers the ability of performance optimization by sharing system resource between pipelines and reducing the burden of data copy.

# Supported Features
## Diversal Input Components
Currently, the package support several kinds of input resources of gaining image data:

|Input Resource|Description|
|--------------------|------------------------------------------------------------------|
|StandardCamera|Any RGB camera with USB port supporting. Currently only the first USB camera if many are connected.|
|RealSenseCamera| Intel RealSense RGB-D Camera, directly calling RealSense Camera via librealsense plugin of openCV.|
|Image Topic| any ROS topic which is structured in image message.|
|Image File| Any image file which can be parsed by openCV, such as .png, .jpeg.|
|Video File| Any video file which can be parsed by openCV.|

## Inference Implementations
Currently, the inference feature list is supported:

|Inference|Description|
|-----------------------|------------------------------------------------------------------|
|Face Detection|Object Detection task applied to face recognition using a sequence of neural networks.|
|Emotion Recognition| Emotion recognition based on detected face image.|
|Age & Gender Recognition| Age and gener recognition based on detected face image.|
|Head Pose Estimation| Head pose estimation based on detected face image.|
|Object Detection| object detection based on SSD-based trained models.|
|Vehicle Detection| Vehicle and passenger detection based on Intel models.|
|Object Segmentation| object detection and segmentation.|
|Person Reidentification| Person Reidentification based on object detection.|

## ROS interfaces and outputs
### Topic
#### Subscribed Topic
- Image topic:
```/camera/color/image_raw```([sensor_msgs::Image](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Image.html))
#### Published Topic
- Face Detection:
```/ros_openvino_toolkit/face_detection```([object_msgs::ObjectsInBoxes](https://github.com/intel/object_msgs/blob/master/msg/ObjectsInBoxes.msg))
- Emotion Recognition:
```/ros_openvino_toolkit/emotions_recognition```([people_msgs::EmotionsStamped](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/msg/EmotionsStamped.msg))
- Age and Gender Recognition:
```/ros_openvino_toolkit/age_genders_Recognition```([people_msgs::AgeGenderStamped](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/msg/AgeGenderStamped.msg))
- Head Pose Estimation:
```/ros_openvino_toolkit/headposes_estimation```([people_msgs::HeadPoseStamped](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/msg/HeadPoseStamped.msg))
- Object Detection:
```/ros_openvino_toolkit/detected_objects```([object_msgs::ObjectsInBoxes](https://github.com/intel/object_msgs/blob/master/msg/ObjectsInBoxes.msg))
- Object Segmentation:
```/ros_openvino_toolkit/segmented_obejcts```([people_msgs::ObjectsInMasks](https://github.com/intel/ros_openvino_toolkit/blob/devel/people_msgs/msg/ObjectsInMasks.msg))
- Person Reidentification:
```/ros_openvino_toolkit/reidentified_persons```([people_msgs::ReidentificationStamped](https://github.com/intel/ros_openvino_toolkit/blob/devel/people_msgs/msg/ReidentificationStamped.msg))
- Rviz Output:
```/ros_openvino_toolkit/image_rviz```([sensor_msgs::Image](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Image.html))

### Service
- Object Detection Service:
```/detect_object``` ([object_msgs::DetectObject](https://github.com/intel/object_msgs/blob/master/srv/DetectObject.srv))
- Face Detection Service:
```/detect_face``` ([object_msgs::DetectObject](https://github.com/intel/object_msgs/blob/master/srv/DetectObject.srv))
- Age & Gender Detection Service:
```/detect_age_gender``` ([people_msgs::AgeGender](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/srv/AgeGenderSrv.srv))
- Headpose Detection Service:
```/detect_head_pose``` ([people_msgs::HeadPose](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/srv/HeadPoseSrv.srv))
- Emotion Detection Service:
```/detect_emotion``` ([people_msgs::Emotion](https://github.com/intel/ros_openvino_toolkit/blob/master/people_msgs/srv/EmotionSrv.srv))


### RViz
RViz dispaly is also supported by the composited topic of original image frame with inference result.
To show in RViz tool, add an image marker with the composited topic:
```/ros_openvino_toolkit/image_rviz```([sensor_msgs::Image](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Image.html))

### Image Window
OpenCV based image window is natively supported by the package.
To enable window, Image Window output should be added into the output choices in .yaml config file. see [the config file guidance](https://github.com/intel/ros_openvino_toolkit/blob/master/doc/YAML_CONFIGURATION_GUIDE.md) for checking/adding this feature in your launching.

## Demo Result Snapshots
See below pictures for the demo result snapshots.
* face detection input from standard camera
![face_detection_demo_image](https://github.com/intel/ros_openvino_toolkit/blob/master/data/images/face_detection.png "face detection demo image")

* object detection input from realsense camera
![object_detection_demo_realsense](https://github.com/intel/ros_openvino_toolkit/blob/master/data/images/object_detection.gif "object detection demo realsense")

* object segmentation input from video
![object_segmentation_demo_video](https://github.com/intel/ros_openvino_toolkit/blob/master/data/images/object_segmentation.gif "object segmentation demo video")

* Person Reidentification input from standard camera
![person_reidentification_demo_video](https://github.com/intel/ros2_openvino_toolkit/blob/master/data/images/person-reidentification.gif "person reidentification demo video")

# Installation & Launching
**NOTE:** Intel releases 2 different series of OpenVINO Toolkit, we call them as [OpenSource Version](https://github.com/opencv/dldt/) and [Tarball Version](https://software.intel.com/en-us/openvino-toolkit). This guidelie uses OpenSource Version as the installation and launching example. **If you want to use Tarball version, please follow [the guide for Tarball Version](https://github.com/intel/ros_openvino_toolkit/blob/master/doc/BINARY_VERSION_README.md).**

## Enable Intel® Neural Compute Stick 2 (Intel® NCS 2) under the OpenVINO Open Source version (Optional) </br>
1. Intel Distribution of OpenVINO toolkit </br>
	* Download OpenVINO toolkit by following the [guide](https://software.intel.com/en-us/openvino-toolkit/choose-download)</br>
	```bash
	cd ~/Downloads
	wget -c http://registrationcenter-download.intel.com/akdlm/irc_nas/15078/l_openvino_toolkit_p_2018.5.455.tgz
	```
	* Install OpenVINO toolkit by following the [guide](https://software.intel.com/en-us/articles/OpenVINO-Install-Linux) </br>
	```bash
	cd ~/Downloads
	tar -xvf l_openvino_toolkit_p_2018.5.455.tgz
	cd l_openvino_toolkit_p_2018.5.455
	# root is required instead of sudo
	sudo -E ./install_cv_sdk_dependencies.sh
	sudo ./install_GUI.sh
	# build sample code under OpenVINO toolkit
	source /opt/intel/computer_vision_sdk/bin/setupvars.sh
 	cd /opt/intel/computer_vision_sdk/deployment_tools/inference_engine/samples/
 	mkdir build
	cd build
 	cmake ..
 	make
	```
	* Configure the Neural Compute Stick USB Driver
	```bash
	cd ~/Downloads
	cat <<EOF > 97-usbboot.rules
	SUBSYSTEM=="usb", ATTRS{idProduct}=="2150", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
	SUBSYSTEM=="usb", ATTRS{idProduct}=="2485", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
	SUBSYSTEM=="usb", ATTRS{idProduct}=="f63b", ATTRS{idVendor}=="03e7", GROUP="users", MODE="0666", ENV{ID_MM_DEVICE_IGNORE}="1"
	EOF
	sudo cp 97-usbboot.rules /etc/udev/rules.d/
	sudo udevadm control --reload-rules
	sudo udevadm trigger
	sudo ldconfig
	rm 97-usbboot.rules
	```
	
2. Configure the environment (you can write the configuration to your ~/.basrch file)</br>
	**Note**: If you used root privileges to install the OpenVINO binary package, it installs the Intel Distribution of OpenVINO toolkit in this directory: */opt/intel/openvino_<version>/*
	```bash
	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/computer_vision_sdk/deployment_tools/inference_engine/samples/build/intel64/Release/lib
	source /opt/intel/computer_vision_sdk/bin/setupvars.sh
	```

## Dependencies Installation
One-step installation scripts are provided for the dependencies' installation. Please see [the guide](https://github.com/intel/ros_openvino_toolkit/blob/master/doc/OPEN_SOURCE_CODE_README.md) for details.

## Launching
* Preparation
	* download and convert a trained model to produce an optimized Intermediate Representation (IR) of the model 
		```bash
		#object segmentation model
		cd /opt/openvino_toolkit/dldt/model-optimizer/install_prerequisites
		sudo ./install_prerequisites.sh
		mkdir -p ~/Downloads/models
		cd ~/Downloads/models
		wget http://download.tensorflow.org/models/object_detection/mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
		tar -zxvf mask_rcnn_inception_v2_coco_2018_01_28.tar.gz
		cd mask_rcnn_inception_v2_coco_2018_01_28
		python3 /opt/openvino_toolkit/dldt/model-optimizer/mo_tf.py --input_model frozen_inference_graph.pb --tensorflow_use_custom_operations_config /opt/openvino_toolkit/dldt/model-optimizer/extensions/front/tf/mask_rcnn_support.json --tensorflow_object_detection_api_pipeline_config pipeline.config --reverse_input_channels --output_dir ./output/
		sudo mkdir -p /opt/models
		sudo ln -s ~/Downloads/models/mask_rcnn_inception_v2_coco_2018_01_28 /opt/models/
		#object detection model
		cd /opt/openvino_toolkit/open_model_zoo/model_downloader
		python3 ./downloader.py --name mobilenet-ssd
 		#FP32 precision model
 		sudo python3 /opt/openvino_toolkit/dldt/model-optimizer/mo.py --input_model /opt/openvino_toolkit/open_model_zoo/model_downloader/object_detection/common/mobilenet-ssd/caffe/mobilenet-ssd.caffemodel --output_dir /opt/openvino_toolkit/open_model_zoo/model_downloader/object_detection/common/mobilenet-ssd/caffe/output/FP32 --mean_values [127.5,127.5,127.5] --scale_values [127.5]
 		#FP16 precision model
 		sudo python3 /opt/openvino_toolkit/dldt/model-optimizer/mo.py --input_model /opt/openvino_toolkit/open_model_zoo/model_downloader/object_detection/common/mobilenet-ssd/caffe/mobilenet-ssd.caffemodel --output_dir /opt/openvino_toolkit/open_model_zoo/model_downloader/object_detection/common/mobilenet-ssd/caffe/output/FP16 --data_type=FP16 --mean_values [127.5,127.5,127.5] --scale_values [127.5]
	* download the optimized Intermediate Representation (IR) of model (excute _once_)<br>
		```bash
		cd /opt/openvino_toolkit/open_model_zoo/model_downloader
		python3 downloader.py --name face-detection-adas-0001
		python3 downloader.py --name age-gender-recognition-retail-0013
		python3 downloader.py --name emotions-recognition-retail-0003
		python3 downloader.py --name head-pose-estimation-adas-0001
		python3 downloader.py --name person-detection-retail-0013
 		python3 downloader.py --name person-reidentification-retail-0076
		```
	* copy label files (excute _once_)<br>
		```bash
		sudo cp /opt/openvino_toolkit/ros_openvino_toolkit/data/labels/emotions-recognition/FP32/emotions-recognition-retail-0003.labels /opt/openvino_toolkit/open_model_zoo/tools/downloader/Retail/object_attributes/emotions_recognition/0003/dldt
		sudo cp /opt/openvino_toolkit/ros_openvino_toolkit/data/labels/face_detection/face-detection-adas-0001.labels /opt/openvino_toolkit/open_model_zoo/tools/downloader/Transportation/object_detection/face/pruned_mobilenet_reduced_ssd_shared_weights/dldt
		sudo cp /opt/openvino_toolkit/ros_openvino_toolkit/data/labels/object_segmentation/frozen_inference_graph.labels /opt/models/mask_rcnn_inception_v2_coco_2018_01_28/output
		sudo cp /opt/openvino_toolkit/ros_openvino_toolkit/data/labels/object_detection/mobilenet-ssd.labels /opt/openvino_toolkit/open_model_zoo/tools/downloader/object_detection/common/mobilenet-ssd/caffe/output/FP32
		sudo cp /opt/openvino_toolkit/ros_openvino_toolkit/data/labels/object_detection/mobilenet-ssd.labels /opt/openvino_toolkit/open_model_zoo/tools/downloader/object_detection/common/mobilenet-ssd/caffe/output/FP16
		```
	* set ENV LD_LIBRARY_PATH<br>
		```bash
		export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/openvino_toolkit/dldt/inference-engine/bin/intel64/Release/lib
		```
* run face detection sample code input from StandardCamera.
	```bash
	roslaunch vino_launch pipeline_people_oss.launch
	```

* run face detection sample code input from Image.
	```bash
	roslaunch vino_launch pipeline_image_oss.launch
	```
* run object detection sample code input from RealsensCamera.
	```bash
	roslaunch vino_launch pipeline_object_oss.launch
	```
* run object detection sample code input from RealsensCameraTopic.
	```bash
	roslaunch vino_launch pipeline_object_oss_topic.launch
	```
* run object segmentation sample code input from RealSenseCameraTopic.
	```bash
	roslaunch vino_launch pipeline_segmentation.launch
	```
* run object segmentation sample code input from Video.
	```bash
	roslaunch vino_launch pipeline_video.launch
	```
* run person reidentification sample code input from StandardCamera.
	```bash
	roslaunch vino_launch pipeline_reidentification_oss.launch
	```
* run object detection service sample code input from Image  
  Run image processing service:
	```bash
	roslaunch vino_launch image_object_server_oss.launch
	```
  Run example application with an absolute path of an image on another console:
	```bash
	rosrun dynamic_vino_sample image_object_client ~/catkin_ws/src/ros_openvino_toolkit/data/images/car.png
	```
* run face detection service sample code input from Image  
  Run image processing service:
	```bash
	roslaunch vino_launch image_people_server_oss.launch
	```
  Run example application with an absolute path of an image on another console:
	```bash
	rosrun dynamic_vino_sample image_people_client ~/catkin_ws/src/ros_openvino_toolkit/data/images/team.jpg
	```
# TODO Features
* Support **result filtering** for inference process, so that the inference results can be filtered to different subsidiary inference. For example, given an image, firstly we do Object Detection on it, secondly we pass cars to vehicle brand recognition and pass license plate to license number recognition.
* Design **resource manager** to better use such resources as models, engines, and other external plugins.
* Develop GUI based **configuration and management tools** (and monitoring and diagnose tools), in order to provide easy entry for end users to simplify their operation. 

# More Information
* ros OpenVINO discription writen in Chinese: https://mp.weixin.qq.com/s/BgG3RGauv5pmHzV_hkVAdw 

###### *Any security issue should be reported using process at https://01.org/security*
