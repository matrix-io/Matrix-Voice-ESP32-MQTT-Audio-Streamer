# Matrix-Voice-ESP32-MQTT-Audio-Streamer

The Audio Steamer is designed to work as an Snips Audio Server, see https://snips.ai/ and also support [Rhasppy](https://rhasspy.readthedocs.io/en/latest/)
Please raise an issue if some of the steps do not work or if they are unclear.

## Features

- Runs standalone, NO raspberry Pi is needed after flashing this program
- Ledring starts RED (no wifi connection), then turns BLUE (Idle mode, with wifi connection)
- Ledring turns GREEN when the hotword is detected, back to BLUE when going to idle
- Uses an asynchronous MQTT client for the playBytes topic
- Uses a synchronous MQTT client for the audiostream.
- OTA Updating
- Dynamic brightness and colors for idle, hotword, updating and disconnected
- Mute / unmute microphones via MQTT
- Reboot device by sending hashed password

## Required Hardware

Before getting started, let's review what you'll need.

- Raspberry Pi 3 (Recommended) or Pi 3 Model B+ (Supported).
- MATRIX Voice ESP32 version - Buy the MATRIX Voice.
- Micro-USB power adapter for Raspberry Pi
- Micro-SD Card (Minimum 8 GB)
- Micro-USB Cable
- A Personal Computer to SSH into your Raspberry Pi
- Internet connection (Ethernet or WiFi)

## Let's Get Started

If you are starting with a fresh install of Raspbian on your Raspberry Pi, follow this guide to set up your OS with the basic MATRIX device packages, as well as Snips.

## 1. Raspberry Pi Setup

Run the following commands inside your Raspberry Pi terminal to install the MATRIX Voice Software. This will keep the FPGA firmware updated and install few tools to flash the ESP-WROOM-32.

Add the MATRIX repository and key.

```
curl https://apt.matrix.one/doc/apt-key.gpg | sudo apt-key add -              
echo "deb https://apt.matrix.one/raspbian $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/matrixlabs.list              
```

Update your repository and packages.

```
sudo apt update              
sudo apt upgrade              
```

Install the MATRIX init package.

```
sudo apt install matrixio-creator-init              
```

Reboot your Raspberry Pi.

```
sudo reboot              
```

SSH back into the Pi, execute this command to enable ESP32 flashing.

```
sudo voice_esp32_enable              
```

Reset the ESP32 flash memory.

```
esptool.py --chip esp32 --port /dev/ttyS0 --baud 115200 --before default_reset --after hard_reset erase_flash            
```

## 2. Editing the Audio Streamer Code

Clone the Audio Streamer repository and enter into the relevant folder.

```
git clone https://github.com/Romkabouter/Matrix-Voice-ESP32-MQTT-Audio-Streamer  
cd Matrix-Voice-ESP32-MQTT-Audio-Streamer-master/MatrixVoiceAudioServer  
```

The MatrixVoiceAudioServer.ino file is the program that is going to run on the MATRIX Voice standalone. It will communicate with the Snips audio server on the Raspberry Pi and send data from the mics through MQTT. This way, the MATRIX Voice can run separate from the Pi and still serve as the input mics and you can place multiple MATRIX Voice satellite modules wherever you want as long as they share a WiFi connection with the Raspberry Pi server.

Within the same directory as the MatrixVoiceAudioServer.ino, there will be a config.h file. In this file, change

- SSID to your WiFi's SSID
- PASSWORD to your WiFi's Password
- MQTT_USER to fit your needs (choose a unique username for each MATRIX Voice satellite that you deploy as MQTT audio streamers)
- MQTT_PASS to a password of your choice

![Screenshot of config.h](config.png)

Additionally, in the MatrixVoiceAudioServer.ino file, change 

- the MQTT_IP to the IP address of your Raspberry Pi
- MQTT_HOST to the IP of your Pi as well
- SITEID to fit your needs (choose a unique ID for each MATRIX Voice satellite, default is "matrixvoice")

![Screenshot of ino file](ino_file.png)

Feel free to edit or add on to the code. We're always happy to see new contributions! You can open this folder in either Visual Studio Code PlatformIO (by following this guide here) or through the Arduino IDE (by following this guide here).
To edit the program, download the following libraries and put them in the Arduino/libraries folder (for Arduino IDE)

- Matrix-Hal-ESP32 https://github.com/matrix-io/matrixio_hal_esp32
- AsynchMqttClient https://github.com/marvinroger/async-mqtt-client
- AsyncTCP https://github.com/me-no-dev/AsyncTCP
- PubSubClient https://github.com/knolleary/pubsubclient. (Change the MQTT_MAX_PACKET_SIZE in PubSubClient.h to 2000)
- ArduinoJSON https://github.com/bblanchon/ArduinoJson

## 3. Uploading and Compiling the Code 

The repository has the firmware bin files and deploy file in the same directory as the MatrixVoiceAudioServer.ino file, so to upload the default code, connect your Voice to the Pi and run the command below. Remember to change the IP in deploy.sh to your PI's IP.

```
sh deploy.sh   
```

When making changes to the code, first compile the file to binary (through either pio run or the arduino sketch->'compile to binary'), and run either deploy.sh or ./install depending on the text editor you are using.
