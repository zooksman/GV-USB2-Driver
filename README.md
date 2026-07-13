GV-USB2 Linux Driver
====================

A linux driver for the IO-DATA GV-USB2 SD capture device.

Installation
===================

Simply run ./install.sh to compile and install the driver. Make sure you have Linux kernel headers installed and a basic GCC build environment. Also make sure you have a modern version of ffmpeg.

Note: the gvusb2-sound module requires alsa sound card index 7 to be open in this install script. If that index is already taken, edit the last command in the script to choose a different index. 

Usage
====================

To set the video source: 

```sudo v4l2-ctl --set-input 1``` for S-video input

```sudo v4l2-ctl --set-input 0``` for Composite input

The default video standard is NTSC. To switch the standard to PAL, use the following command:

```sudo v4l2-ctl -s pal```

To monitor video from the GVUSB2 in real time:

```ffplay  /dev/video0 -vf setdar=4/3,setfield=tff```

Here is the command I use to capture lossless FFV1-encoded video and FLAC audio from the GVUSB2:

```
ffmpeg \
    -f v4l2 -framerate 29.97 -video_size 720x486 -rtbufsize 2G -thread_queue_size 1024 -i /dev/video0  \
    -f alsa -ac 2 -rtbufsize 512M -thread_queue_size 1024 -i hw:CARD=gvusb2,DEV=0 \
    -c:v ffv1 -level 3 -threads 4 -coder 1 -context 1 -slices 30 -slicecrc 1 -g 1 \
    -vf setdar=4/3,setfield=tff -flags +ilme+ildct -fflags +genpts+igndts -fps_mode:v passthrough \
    -c:a flac -compression_level 8 \
    OUTPUT_FILE.mkv
```

IMPORTANT NOTES FOR ARCHIVAL WITH THE ABOVE COMMAND:

- With this command both the video and audio streams are losslessly compressed in realtime, which requires ample available PC resources. 
- You should have at least 8GB free RAM and a minimum of 4 CPU cores to allow enough buffer space and improve multithreading performance. Writing directly to HDDs is NOT RECOMMENDED.
- If you are capturing PAL video, make sure to change framerate to 25 and video_size to 720x576 in the above command.
- For capturing VBI data, change -video_size to the following: 720x516 for NTSC, 720x616 for PAL

Note: These commands assume that /dev/video0 points to the GVUSB2. If you are using a different index, set the -d flag in v4l2_ctl to point to the correct device.

# Streaming with the GVUSB2

Use the instructions provided in this wonderful gist: https://gist.github.com/scaramangado/4e09031d782cbad8a4446ba101f43ef7

Setting Device Controls
====================

Enabling capture of VBI (for a total height of 516px-- the top 9 lines are not able to be captured):

```sudo v4l2-ctl -c vbi_capture=1```

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

Vertical Start (increasing this value shifts the image upward, max 4, default 2):

```sudo v4l2-ctl -c vertical_start=X```

Horizontal Start (increasing this value shifts the image leftward, max 4, default 4):

```sudo v4l2-ctl -c horizontal_start=X```

Disable Automatic Gain Control (AGC):

```sudo v4l2-ctl -c gain_automatic=0```

Manual Gain Setting (only takes effect when AGC is disabled):

```sudo v4l2-ctl -c gain=X```
