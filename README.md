# Azure IoT Demo

![Architecture](architecture.png)

## Links

### Code Links

* [Supported Devices](https://catalog.azureiotsolutions.com/kits?filters={%2212%22:[%221%22])
* [LibSSL Issue for Ubuntu 20.04](https://github.com/Azure/iotedge/issues/1918) - we're using 18.04 for now!

### Documentation

* https://docs.microsoft.com/en-us/cli/azure/iot/hub?view=azure-cli-latest#az_iot_hub_create
* Most of the magic! ❤️ https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization
* https://github.com/Azure/azure-iotedge/releases


## Hub Info

Name: 
IOT_HUB_NAME=YourIoTH2N0P3N0P3N0P3

### Keys

```json
      "primaryKey": "N0P3N0P3N0P3N0P3N0P3N0P3N0P3",
      "secondaryKey": "N0P3N0P3N0P3N0P3N0P3N0P3N0P3N0P3N0P3N0P3"
```

## Script

In Azure Cloud Shell: 

```bash

#Set up our variables
IOT_HUB_NAME=YourIoTHubName$(date +%s)
CONSUMER=YourConsumerGroupName
RG=default
DEVICE_ID=ubrpi32

# Ensure we've got the az cli extension for iot installed
az extension add --name azure-iot

# Create the hub, consumer group and retrieve the connection string
az iot hub create --resource-group $RG --name $IOT_HUB_NAME --sku S1 --partition-count 2
az iot hub consumer-group create --hub-name $IOT_HUB_NAME --name $CONSUMER
CONN_STRING=$(az iot hub show-connection-string --hub-name $IOT_HUB_NAME --policy-name service)

# Print the connection string to the screen just in case
echo $CONN_STRING

# Clone the repo and switch to it
git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
cd web-apps-node-iot-hub-data-visualization

# Set up device
az iot hub device-identity create --device-id $DEVICE_ID --hub-name $IOT_HUB_NAME --edge-enabled
az iot hub device-identity list --hub-name $IOT_HUB_NAME
az iot hub device-identity connection-string show --device-id $DEVICE_ID --hub-name $IOT_HUB_NAME

```

On your Raspberry Pi 3/4 or simulator running Ubuntu 18.04.5 on ARM64:

```bash
DD_API=""
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/

curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo apt install -y iotedge
DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=$DD_API DD_SITE="datadoghq.com" bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
```

Configuring the iotedge agent:

```bash
/etc/init.d/iotedge restart
sed -s 's/\<ADD DEVICE CONNECTION STRING HERE\>'
sudo /etc/init.d/iotedge restart
sudo systemctl status iotedge
```

In Azure Cloud Shell, run the app:

```bash
npm install 
IotHubConnectionString="" \
EventHubConsumerGroup=YourConsumerGroupName \
npm start

```

## Sensor Setup

First update apt:

```bash
sudo apt update
```

Next, install `libgpiod`, and fix the 32bit pulsein bug by recompiling it for ARM64:

```bash
sudo apt install -y libgpiod-dev git build-essential
git clone https://github.com/adafruit/libgpiod_pulsein.git
cd libgpiod_pulsein/src
make
cp libgpiod_pulsein ~/.local/lib/python3.6/site-packages/adafruit_blinka/microcontroller/bcm283x/pulseio/libgpiod_pulsein
cd
```

Finally, install the Adafruit Circuit Python DHT pip package, and all the Python bits needed:

```bash
sudo apt install -y python3-dev python3-pip
python3 -m pip install --upgrade pip setuptools wheel
pip3 install adafruit-circuitpython-dht
```
## Python Test Script

From _this tutorial_:

```python
import time
import board
import adafruit_dht
 
# Initialize the dht device, with data pin connected to:
dhtDevice = adafruit_dht.DHT11(board.D18)
 
while True:
    try:
        # Print the values to the serial port
        temperature_c = dhtDevice.temperature
        temperature_f = temperature_c * (9 / 5) + 32
        humidity = dhtDevice.humidity
        print(
            "Temp: {:.1f} F / {:.1f} C    Humidity: {}% ".format(
                temperature_f, temperature_c, humidity
            )
        )
 
    except RuntimeError as error:
        # Errors happen fairly often, just keep trying
        print(error.args[0])
        time.sleep(2.0)
        continue
    except Exception as error:
        dhtDevice.exit()
        raise error
 
    time.sleep(1.0)
```

Results:
```
ubuntu@ubrpi32:~$ sudo python3 test.py
Temp: 73.4 F / 23.0 C    Humidity: 55%
Temp: 73.4 F / 23.0 C    Humidity: 54%
Temp: 73.4 F / 23.0 C    Humidity: 54%
Temp: 73.4 F / 23.0 C    Humidity: 54%
Temp: 73.4 F / 23.0 C    Humidity: 54%
Temp: 73.4 F / 23.0 C    Humidity: 53%
Temp: 73.4 F / 23.0 C    Humidity: 55%
Temp: 77.0 F / 25.0 C    Humidity: 95%
Temp: 77.0 F / 25.0 C    Humidity: 95%
Temp: 80.6 F / 27.0 C    Humidity: 95%
Temp: 80.6 F / 27.0 C    Humidity: 95%
Temp: 80.6 F / 27.0 C    Humidity: 95%
Temp: 82.4 F / 28.0 C    Humidity: 95%
Temp: 84.2 F / 29.0 C    Humidity: 95%
```

Video demo:

![](CHT11_Raspberry_Pi_4_64bit_Ubuntu_18-EXTRALOW.gif)

# Useful Commands

`sudo iotedge check`
