# mqtt-camera-streamer
**TLDR:** Publish frames from a connected camera (or MJPEG/RTSP stream) to an MQTT topic, and demonstrate how this can be used to create a simple image processing pipeline that can be used in an IOT or home automation project. The camera stream can be viewed in a browser with [Streamlit](https://github.com/streamlit/streamlit) or Home Assistant.

**Long introduction:** A typical task in IOT/science is that you have a camera connected to one machine and you want to view the camera feed on a different machine, and maybe preprocess the images and save them to disk. I have always found this to be may more work than expected. In particular working with camera streams can get quite complicated, and may force you to experiment with tools like Gstreamer and ffmpeg that have a steep learning curve. In contrast, working with [MQTT](http://mqtt.org/) is very straightforward and is also probably familiar to anyone with an interest in IOT. This code uses MQTT to send frames from a camera over a netork at low FPS. Whilst MQTT is rarely used for this purpose (sending files) I have not encountered any issues doing this. Furthermore it is possible to setup an image processing pipeline by linking MQTT topics together, using an `on_message(topic)` to do some processing and send the processed image downstream on another topic. Note that this is not a high FPS (frames per second) solution, and in practice I achieve around 1 FPS which is practical for tasks such as preprocessing (cropping, rotating) images prior to viewing them. The code is written for simplicity and ease of use, not high performance. If you require high performance/low latency checkout [Frigate](https://github.com/blakeblackshear/frigate) instead.

## Setup
On linux/OSX/Windows use a venv to isolate your environment, and install the required dependencies:
```
$ (base) python3 -m venv venv
$ (base) source venv/bin/activate
$ (venv) pip3 install -r requirements.txt
```

#### Raspberry Pi
Do not use a venv and install openCV system wide using: 
```
$ sudo apt install python3-opencv
$ pip3 install -r requirements.txt
```
I have not tested Streamlit on the Raspberry pi, but you can use the viewer on another machine (WIndows, OSX) anyways so don't worry.

## Camera usage
Use the `config.yaml` file in `config` directory to setup the system (mqtt broker IP etc) and validate the config by running:
```
$ (venv) python3 scripts/validate_config.py
```

To publish camera frames over MQTT:
```
$ (venv) python3 scripts/camera.py
```

To view the camera frames with Streamlit:
```
$ (venv) streamlit run scripts/viewer.py
```

**Note:** if streamlit becomes unresponsive, `ctrl-z` to pause Streamlit then `kill -9 %%`. Also note that the viewer can be run on amy machine on your network. 

## Image processing pipeline
To process a camera stream (the example rotates the image):
```
$ (venv) python3 scripts/processing.py
```

## Save frames
To save frames to disk:
```
$ (venv) python3 scripts/save-captures.py
```

## Camera display
The `viewer.py` script uses Streamlit to display the camera feed:

<p align="center">
<img src="https://github.com/robmarkcole/mqtt-camera-streamer/blob/master/docs/images/viewer_usage.png" width="500">
</p>

## Home Assistant
You can view the camera feed using [Home Assistant](https://www.home-assistant.io/) and configuring an [MQTT camera](https://www.home-assistant.io/components/camera.mqtt/). Add to your `configuration.yaml`:
```yaml
camera:
  - platform: mqtt
    topic: homie/mac_webcam/capture
    name: mqtt_camera
  - platform: mqtt
    topic: homie/mac_webcam/capture/rotated
    name: mqtt_camera_rotated
```

<p align="center">
<img src="https://github.com/robmarkcole/mqtt-camera-streamer/blob/master/docs/images/ha_usage.png" width="500">
</p>

## Listing cameras
If your laptop has a built-in webcam this will generally be listed as `VIDEO_SOURCE = 0`. If you plug in an external USB webcam this takes precedence over the inbuilt webcam, with the external camera becoming `VIDEO_SOURCE = 0` and the built-in webcam becoming `VIDEO_SOURCE = 1`. To check which cameras are detected run:
```
$ (venv) python3 scripts/check-cameras.py
```
Alternatively you can pass a string to an MJPEG/RTSP stream, For example `"rtsp://admin:password@192.168.1.94:554/11" `

## MQTT
Need an MQTT broker? If you have Docker installed I recommend [eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto). A basic broker can be run with 
```
$ docker run --p 1883:1883 -d eclipse-mosquitto`
```
Note that I have stuctured the MQTT topics following the homie MQTT convention, link in the refefrences. This is not necessary but is best practice IMO.

### References
* [imageZMQ](https://github.com/jeffbass/imagezmq) -> inspired this project, but uses ZMQ
* [homie MQTT convention](https://homieiot.github.io/) -> how to structure MQTT topics
* [yolocam_mqtt](https://github.com/LarsAC/yolocam_mqtt/blob/master/yolo_mqtt_server.py) -> another source of ideas