# use_realsense_d435
Install &amp; Use

### 1.Install the realsense SDK & dev<br>
Follow the realsense website: https://www.intelrealsense.com/developers/#downloads<br>

Add Intel server to the list of repositories : <br>
```
echo 'deb http://realsense-hw-public.s3.amazonaws.com/Debian/apt-repo xenial main' | sudo tee /etc/apt/sources.list.d/realsense-public.list
```
<br>

It is recommended to backup /etc/apt/sources.list.d/realsense-public.list file in case of an upgrade.
<br>

Register the server’s public key :
<br>

```
sudo apt-key adv --keyserver keys.gnupg.net --recv-key 6F3EFCDE
```

Refresh the list of repositories and packages available :
<br>

```
sudo apt-get update
```

In order to run demos install:
<br>

```
sudo apt-get install librealsense2-dkms
sudo apt-get install librealsense2-utils
```

The above two lines will deploy librealsense2 udev rules, kernel drivers, runtime library and executable demos and tools. Reconnect the Intel RealSense depth camera and run:
<br>

```
realsense-viewer
```

Developers shall install additional packages:
<br>

```
sudo apt-get install librealsense2-dev
sudo apt-get install librealsense2-dbg
```

With dev package installed, you can compile an application with librealsense using g++ -std=c++11 filename.cpp -lrealsense2 or an IDE of your choice.
<br>

Verify that the kernel is updated :
<br>

```
modinfo uvcvideo | grep "version:" should include realsense string
```
<br>

![image](https://github.com/zhucheng725/use_realsense_d435/blob/master/2019-06-20%2010-26-23%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

<br>

### 2.update
<br>
Firstly, I start to use 
<br>

```
realsense-viewer
```

to see the IDE, but I can not use this IDE to get frame from the D435.
<br>

And I try updating and use this IDE successfully. You can search some details from the website:
<br>

https://www.intel.com/content/www/us/en/support/articles/000028593/emerging-technologies/intel-realsense-technology.html

Then I download this bin from :
<br>

https://downloadmirror.intel.com/28870/eng/D400_Series_Production_FW_5_11_6_250.zip
<br>

Then to do this command:
```
sudo apt-get install intel-realsense-dfu*
lsusb
```
<br>
Notice “Intel Corp.” bus and device numbers; DFU tool uses these values to identify Intel® RealSenseTM D400 series camera.<br>
(This command specifies bus #, device #, -f flag to force upgrade, and –i flag for complete system path to downloaded FW.bin file.)<br>

```
intel-realsense-dfu –b 002 –d 002 –f –i /home/intel/downloads/Signed_Image_UVC_5_9_2_0.bin
```
<br>

Check firmware with command:
<br>
```
intel-realsense-dfu –p 
```
<br>

### 3.Python<br>
If you wanna to use python wrapper, you must install pyrealsense2
<br>

```
pip3 install --user pyrealsense2
```
<br>

### 4.Example<br>
Some examples can be used from:
<br>
https://github.com/IntelRealSense/librealsense/tree/master/wrappers/python/examples

This code can use D435 to capture the frame and show in screen in real time.
<br>
```
import cv2
import numpy as np
import pyrealsense2 as rs


pipeline = rs.pipeline()
config = rs.config()
config.enable_stream(rs.stream.depth, 640,480, rs.format.z16, 30)
config.enable_stream(rs.stream.color, 640,480, rs.format.bgr8, 30)
pipeline.start(config)

#COLORMAP_AUTUMN = 0,
#COLORMAP_BONE = 1,
#COLORMAP_JET = 2,
#COLORMAP_WINTER = 3,
#COLORMAP_RAINBOW = 4,
#COLORMAP_OCEAN = 5,
#COLORMAP_SUMMER = 6,
#COLORMAP_SPRING = 7,
#COLORMAP_COOL = 8,
#COLORMAP_HSV = 9,
#COLORMAP_PINK = 10,
#COLORMAP_HOT = 11

while(1):
    frames = pipeline.wait_for_frames()
    depth_frame = frames.get_depth_frame()
    color_frame = frames.get_color_frame()
    depth_image = np.asanyarray(depth_frame.get_data())
    color_image = np.asanyarray(color_frame.get_data())
    depth_colormap = cv2.applyColorMap(cv2.convertScaleAbs(depth_image, alpha = 0.1), 2)
    images = np.hstack((color_image, depth_colormap))
    cv2.imshow('Realsense', images)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cv2.destoryAllWindows()
pipeline.stop()
```
<br>

![image](https://github.com/zhucheng725/use_realsense_d435/blob/master/test.jpg)
<br>

### 5.TX2<br>
Next time I want to use D435 in TX2 platform and this website maybe can be used in the future.
<br>
https://dev.intelrealsense.com/docs/nvidia-jetson-tx2-installation
