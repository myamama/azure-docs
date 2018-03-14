---
# Mandatory fields. See more on aka.ms/skyeye/meta.
title: Quickstart Azure IoT Edge + Linux | Microsoft Docs 
description: Try out Azure IoT Edge by running analytics on a simulated edge device
services: iot-edge
keywords: 
author: kgremban
manager: timlt

ms.author: kgremban
ms.date: 01/11/2018
ms.topic: article
ms.service: iot-edge

---

# Quickstart: Deploy your first IoT Edge module to a Linux or Mac device - preview

Azure IoT Edge moves the power of the cloud to your Internet of Things devices. In this topic, learn how to use the cloud interface to deploy prebuilt code remotely to an IoT Edge device.

If you don't have an active Azure subscription, create a [free account][lnk-account] before you begin.

## Prerequisites

* [Provision an Ubuntu Server 16.04 LTS VM on Azure][lnk-provision-ubuntu-vm]

This quickstart uses your computer or virtual machine like an Internet of Things device. To turn your machine into an IoT Edge device, the following services are required:

* Python pip, to install the IoT Edge runtime.
   * Linux: `sudo apt-get install python-pip`.
   * MacOS: `sudo easy_install pip`.
* Docker, to run the IoT Edge modules
   * [Install Docker for Linux][lnk-docker-ubuntu] and make sure that it's running. If failed to install docker you can try this link [install docker for Ubuntu][lnk-install-docker-ubuntu]
   * [Install Docker for Mac][lnk-docker-mac] and make sure that it's running. 

## Create an IoT hub with Azure CLI

Create an IoT hub in your Azure subscription. The free level of IoT Hub works for this quickstart. If you've used IoT Hub in the past and already have a free hub created, you can skip this section and go on to [Register an IoT Edge device][anchor-register]. Each subscription can only have one free IoT hub. 

1. Sign in to the [Azure portal][lnk-portal]. 
1. Select the **Cloud Shell** button. 

   ![Cloud Shell button][1]

1. Create a resource group. The following code creates a resource group called **IoTEdge** in the **West US** region:

   ```azurecli
   az group create --name IoTEdge --location westus
   ```

1. Create an IoT hub in your new resource group. The following code creates a free **F1** hub called **MyIotHub** in the resource group **IoTEdge**:

   ```azurecli
   az iot hub create --resource-group IoTEdge --name MyIotHub --sku F1 
   ```

## Register an IoT Edge device

Create a device identity for your simulated device so that it can communicate with your IoT hub. Since IoT Edge devices behave and can be managed differently than typical IoT devices, you declare this to be an IoT Edge device from the beginning. 

1. In the Azure portal, navigate to your IoT hub.
1. Select **IoT Edge (preview)**.
1. Select **Add IoT Edge device**.
1. Give your simulated device a unique device ID.
1. Select **Save** to add your device.
1. Select your new device from the list of devices. 
1. Copy the value for **Connection string--primary key** and save it. You'll use this value to configure the IoT Edge runtime in the next section. 

## Install and start the IoT Edge runtime

The IoT Edge runtime is deployed on all IoT Edge devices. It comprises two modules. First, the IoT Edge agent facilitates deployment and monitoring of modules on the IoT Edge device. Second, the IoT Edge hub manages communications between modules on the IoT Edge device, and between the device and IoT Hub. 

On the machine where you'll run the IoT Edge device, download the IoT Edge control script:
```bash
sudo pip install -U azure-iot-edge-runtime-ctl
```

Configure the runtime with your IoT Edge device connection string from the previous section:
```bash
sudo iotedgectl setup --connection-string "{device connection string}" --auto-cert-gen-force-no-passwords
```

Start the runtime:
```bash
sudo iotedgectl start
```

Check Docker to see that the IoT Edge agent is running as a module:
```bash
sudo docker ps
```

![See edgeAgent in Docker](./media/tutorial-simulate-device-linux/docker-ps.png)

## Deploy a module

One of the key capabilities of Azure IoT Edge is being able to deploy modules to your IoT Edge devices from the cloud. An IoT Edge module is an executable package implemented as a container. In this section, you deploy a module that generates telemetry for your simulated device. 

1. In the Azure portal, navigate to your IoT hub.
1. Go to **IoT Edge (preview)** and select your IoT Edge device.
1. Select **Set Modules**.
1. Select **Add IoT Edge Module**.
1. In the **Name** field, enter `tempSensor`. 
1. In the **Image URI** field, enter `microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview`. 
1. Leave the other settings unchanged, and select **Save**.

   ![Save IoT Edge module after entering name and image URI](https://github.com/MicrosoftDocs/azure-docs/blob/master/includes/media/iot-edge-deploy-module/name-image.png)

1. Back in the **Add modules** step, select **Next**.
1. In the **Specify routes** step, select **Next**.
1. In the **Review template** step, select **Submit**.
1. Return to the device details page and select **Refresh**. You should see the new tempSensor module running along the IoT Edge runtime. 

   ![View tempSensor in list of deployed modules][1]

<!-- Images -->
[1]: https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/iot-edge/media/tutorial-simulate-device-windows/view-module.png
## View generated data

In this quickstart, you created a new IoT Edge device and installed the IoT Edge runtime on it. Then, you used the Azure portal to push an IoT Edge module to run on the device without having to make changes to the device itself. In this case, the module that you pushed creates environmental data that you can use for the tutorials. 

Open the command prompt on the computer running your simulated device again. Confirm that the module deployed from the cloud is running on your IoT Edge device:

```bash
sudo docker ps
```

![View three modules on your device](./media/tutorial-simulate-device-linux/docker-ps2.png)

View the messages being sent from the tempSensor module to the cloud:

```bash
sudo docker logs -f tempSensor
```

![View the data from your module](./media/tutorial-simulate-device-linux/docker-logs.png)

You can also view the telemetry the device is sending by using the [IoT Hub explorer tool][lnk-iothub-explorer]. 

## Clean up resources

If you want to remove the simulated device that you created, along with the Docker containers that were started for each module, use the following command: 

```bash
sudo iotedgectl uninstall
```

When you no longer need the IoT Hub you created, you can use the [az iot hub delete][lnk-delete] command to remove the resource and any devices associated with it:

```azurecli
az iot hub delete --name {your iot hub name} --resource-group {your resource group name}
```

## Next steps

You learned how to deploy an IoT Edge module to an IoT Edge device. Now try deploying different types of Azure services as modules, so that you can analyze data at the edge. 

* [Deploy your own code as a module](tutorial-csharp-module.md)
* [Deploy Azure Function as a module](tutorial-deploy-function.md)
* [Deploy Azure Stream Analytics as a module](tutorial-deploy-stream-analytics.md)

Added
* [Customize Temp Sensor Module, and Create Your own DATA GENERATOR](https://github.com/Azure/iot-edge/tree/master/v2/samples/azureiotedge-simulated-temperature-sensor)

<!-- Images -->
[1]: ./media/quickstart/cloud-shell.png

<!-- Links -->
[lnk-docker-ubuntu]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/ 
[lnk-docker-mac]: https://docs.docker.com/docker-for-mac/install/
[lnk-iothub-explorer]: https://github.com/azure/iothub-explorer
[lnk-account]: https://azure.microsoft.com/free
[lnk-portal]: https://portal.azure.com
[lnk-delete]: https://docs.microsoft.com/cli/azure/iot/hub?view=azure-cli-latest#az_iot_hub_delete
[lnk-install-docker-ubuntu]: https://unix.stackexchange.com/questions/363048/unable-to-locate-package-docker-ce-on-a-64bit-ubuntu
[lnk-provision-ubuntu-vm]: https://docs.microsoft.com/en-us/azure/virtual-machines/linux/quick-create-portal

<!-- Anchor links -->
[anchor-register]: #register-an-iot-edge-device
