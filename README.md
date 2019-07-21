# mqtt-camera-streamer
Working with camera streams can get quite complicated, and may force you to experiment with tools like Gstreamer and ffmpeg that have a steep learning curve. In contrast working with MQTT is very straightforward, and is also very familiar to anyone with an interest in IOT. It is possible to setup a data processing pipeline by linking MQTT topics together, using an `on_message(topic)` to do some processing and send the processed data downstream on another topic. The aim of this code is to publish frames from a camera feed to an MQTT topic, and demonstrate how this can be used to create an image processing pipeline that can be used in an IOT or home automation project. 

## Setup
Use a venv to isolate your environment, and install the required dependencies:
```
$ (base) python3 -m venv venv
$ (base) source venv/bin/activate
$ (venv) pip3 install -r requirements.txt
```

## Usage

To publish camera frames over MQTT:
```
$ (venv) python3 camera.py
```

(OPTIONAL) To process a camera stream (the example rotates the image):
```
$ (venv) python3 processing.py
```

(OPTIONAL) To save frames to disk:
```
$ (venv) python3 save-captures.py
```

## Camera display
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
$ (venv) python3 check-cameras.py
```

## MQTT
Need an MQTT broker? If you have Docker installed [I recommend eclipse-mosquitto](https://hub.docker.com/_/eclipse-mosquitto). A basic broker can be run with `$ docker run --p 1883:1883 eclipse-mosquitto`

### References
* [homie MQTT convention](https://homieiot.github.io/)
* [yolocam_mqtt](https://github.com/LarsAC/yolocam_mqtt/blob/master/yolo_mqtt_server.py)
* [cusca](https://github.com/dgomes/cusca)
* [imageZMQ](https://github.com/jeffbass/imagezmq)