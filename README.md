IP camera based on Logitech webcam, Beaglebone/Raspberry Pi and gstreamer.

==Update Raspberry Firmware First==
The capture util can also be run under Raspberry Pi, but you should be sure to update the firmware to the latest.
Follow the instruction here: http://wiki.matthiasbock.net/index.php/Logitech_C920,_streaming_H.264#Raspberry_Pi

==How to use the capture util==

===Under Windows===
Run gstreamer in a remote machine, I use Windows as a client to view the video stream
Install all the components of gstreamer-1.0
run the gst-launch from the bin directory of gstreamer, or you can set your environment PATH to gstreamers bin directory.
```
cd D:\gstreamer\1.0\x86_64\bin
D:\gstreamer\1.0\x86_64\bin>gst-launch-1.0 -e -v udpsrc port=8001 ! application/x-rtp, payload=96 ! rtpjitterbuffer ! rtph264depay ! avdec_h264 ! autovideosink sync=false
```
===Under Ubuntu===
Under Ubuntu, should be using the similar commandline as windows. (Not tested yet)

===Under Raspberry Pi===
When using in Raspberry Pi 
```
./capture -c 10000 -o | gst-launch-1.0 -v -e filesrc location=/dev/fd/0 ! h264parse ! rtph264pay ! udpsink host=192.168.1.42 port=8001
```

After running the above command, you should be able to see the video in the remote machine, the result is better than using the vlc
==Alternative method, but have delay==

===Using VLC===
While using vlc in Raspbery Pi:
```
cvlc v4l2:///dev/video0:chroma=h264:width=1080:height=720 --sout '#standard{access=http,mux=ts,dst=0.0.0.0:8080,name=stream,mime=video/ts}' -vvv
```
Run in client side (PC), the ip and port is your Raspberry Pi's ip address and port
```
gst-launch-1.0 playbin uri=http://192.168.1.163:8080
```
===Using V4L2 with gstreamer===
Using v4l2 in Raspberry Pi wil have the following error message:
```
pi@raspberrypi ~ $ gst-launch-1.0 -e -vvvvv v4l2src ! video/x-h264, width=1920, height=1080, framerate=30/1 ! h264parse ! rtph264pay pt=96 config-interval=30 ! udpsink host=192.168.1.42 port=8001
Setting pipeline to PAUSED ...
Pipeline is live and does not need PREROLL ...
Setting pipeline to PLAYING ...
New clock: GstSystemClock
/GstPipeline:pipeline0/GstV4l2Src:v4l2src0.GstPad:src: caps = video/x-h264, width=(int)1920, height=(int)1080, interlace-mode=(string)progressive, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1
/GstPipeline:pipeline0/GstCapsFilter:capsfilter0.GstPad:src: caps = video/x-h264, width=(int)1920, height=(int)1080, interlace-mode=(string)progressive, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1
/GstPipeline:pipeline0/GstH264Parse:h264parse0.GstPad:sink: caps = video/x-h264, width=(int)1920, height=(int)1080, interlace-mode=(string)progressive, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1
/GstPipeline:pipeline0/GstCapsFilter:capsfilter0.GstPad:sink: caps = video/x-h264, width=(int)1920, height=(int)1080, interlace-mode=(string)progressive, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1
ERROR: from element /GstPipeline:pipeline0/GstV4l2Src:v4l2src0: Internal data flow error.
Additional debug info:
gstbasesrc.c(2812): gst_base_src_loop (): /GstPipeline:pipeline0/GstV4l2Src:v4l2src0:
streaming task paused, reason not-negotiated (-4)
EOS on shutdown enabled -- waiting for EOS after Error
Waiting for EOS...
ERROR: from element /GstPipeline:pipeline0/GstH264Parse:h264parse0: No valid frames found before end of stream
Additional debug info:
gstbaseparse.c(1066): gst_base_parse_sink_default (): /GstPipeline:pipeline0/GstH264Parse:h264parse0
```
