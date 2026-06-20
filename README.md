GV-USB2 Linux Driver
====================

A linux driver for the IO-DATA GV-USB2 SD capture device.

Installation
===================

Simply run ./install.sh to compile and install the driver. Make sure you have Linux kernel headers installed and a basic GCC build environment. Also make sure you have a modern version of ffmpeg.

Usage
====================

To set the input on the GVUSB2: 

```sudo v4l2-ctl --set-input 1``` for S-video input

```sudo v4l2-ctl --set-input 0``` for composite input

The default region is NTSC. To set the region to PAL, use the following command:

```sudo v4l2-ctl -s pal```

To monitor video from the GVUSB2 without capturing:

```ffplay  /dev/video0 -vf setdar=4/3,setfield=tff```

Here is the command I use to capture lossless FFV1-encoded video and audio from the device to an MKV file:
```
ffmpeg \
    -f v4l2 -framerate 29.97 -video_size 720x480 -rtbufsize 2G -thread_queue_size 1024 -i /dev/video0  \
    -f alsa -ac 2 -rtbufsize 500M -thread_queue_size 512 -i hw:CARD=gvusb2,DEV=0 \
    -c:v ffv1 -level 3 -threads 2 -coder 1 -context 1 -slices 4 -slicecrc 1 \
    -vf setdar=4/3,setfield=tff -flags +ilme+ildct -fflags +genpts \
    -c:a flac -compression_level 8 \
    output_file.mkv
```
If you are capturing PAL video, make sure to change framerate to 25 and video_size to 720x576 in the above command. 

Note: These commands assume that /dev/video0 points to the GVUSB2. If you are using a different index, set the -d flag in v4l2_ctl to point to the correct device.

Setting Device Controls
====================

Brightness (default 128):

```sudo v4l2-ctl -c brightness=X```
Contrast (default 105):

```sudo v4l2-ctl -c contrast=X```
Saturation (default 128):

```sudo v4l2-ctl -c saturation=X```
Hue (NTSC only, default 128):

```sudo v4l2-ctl -c hue=X```
Sharpness (Range 0-16, default 0):

```sudo v4l2-ctl -c sharpness=X```
Vertical Start (increasing this value shifts the image upward, max 4, default 2 for NTSC and 1 for PAL):

```sudo v4l2-ctl -c vertical_start=X```
Horizontal Start (increasing this value shifts the image leftward, max 4, default 0):

```sudo v4l2-ctl -c horizontal_start=X```
