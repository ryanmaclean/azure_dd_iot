# Azure IoT Demo

## Links

### Code Links

* Devices: https://catalog.azureiotsolutions.com/kits?filters={%2212%22:[%221%22]}
* LibSSL Issue for Ubuntu 20.04 - https://github.com/Azure/iotedge/issues/1918

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

In azure cloud shell: 

```bash

#Set up our variables
IOT_HUB_NAME=YourIoTHubName$(date +%s)
CONSUMER=YourConsumerGroupName
RG=default
DEVICE_ID=ubrpi32

# Ensure we've got the az clie extension for iot installed
az extension add --name azure-iot

# Create the hub, consumer group and retrive the connection string
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

# Useful Commands

`sudo iotedge check`
