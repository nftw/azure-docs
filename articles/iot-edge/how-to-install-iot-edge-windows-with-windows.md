---
title: How to install Azure IoT Edge on Windows with Windows containers | Microsoft Docs
description: Azure IoT Edge installation instructions on Windows with  Windows containers
author: kgremban
manager: timlt
# this is the PM responsible
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 08/06/2018
ms.author: kgremban
---
# Install Azure IoT Edge runtime on Windows to use with Windows containers

The Azure IoT Edge runtime is deployed on all IoT Edge devices. It has three components. The **IoT Edge security daemon** provides and maintains security standards on the Edge device. The daemon starts on every boot and bootstraps the device by starting the IoT Edge agent. The **IoT Edge agent** facilitates deployment and monitoring of modules on the Edge device, including the IoT Edge hub. The **IoT Edge hub** manages communications between modules on the IoT Edge device, and between the device and IoT Hub.

This article lists the steps to install the Azure IoT Edge runtime on your Windows x64 (AMD/Intel) system. 

Windows support is currently in Preview.

## Supported Windows versions
Azure IoT Edge with Windows containers can be used with:
  * Windows 10/IoT Enterprise/IoT Core with April 2018 update (Build 17134).
  * Windows Server 1803

## Install the container runtime 

>[!NOTE]
>For container engine installation on Windows IoT Core, follow steps from [provision an IoT Core device article][lnk-iot-core] and then continue with instructions below.

Azure IoT Edge relies on a [OCI-compatible][lnk-oci] container runtime (for example, Docker). You can use [Docker for Windows][lnk-docker-for-windows] for development and testing. 

Configure Docker for Windows [to use Windows containers][lnk-docker-config].

## Install the Azure IoT Edge Security Daemon

>[!NOTE]
>Azure IoT Edge software packages are subject to the license terms located in the packages (in the LICENSE directory). Please read the license terms prior to using the package. Your installation and use of the package constitutes your acceptance of these terms. If you do not agree with the license terms, do not use the package.

A single IoT Edge device can be provisioned manually using a device connections string provided by IoT Hub. Or, you can use the Device Provisioning Service to automatically provision devices, which is helpful when you have many devices to provision. Depending on your provisioning choice, choose the appropriate installation script. 

### Install and manually provision

1. Follow the steps in [Register a new Azure IoT Edge device][lnk-dcs] to register your device and retrieve the device connection string. 

2. On your IoT Edge device, run PowerShell as an administrator. 

3. Download and install the IoT Edge runtime. 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Install-SecurityDaemon -Manual -ContainerOs Windows
   ```

4. When prompted for a **DeviceConnectionString**, provide the connection string that you retrieved from IoT Hub. Do not include quotes around the connection string. 

### Install and automatically provision

1. Follow the steps in [Create and provision a simulated TPM Edge device on Windows][lnk-dps] to set up the Device Provisioning Service and retrieve its **Scope ID**, simulate a TPM device and retrieve its **Registration ID**, then create an individual enrollment. Once your device is registered in your IoT Hub, continue with the installation.  

   >[!TIP]
   >Keep the window that's running the TPM simulator open during your installation and testing. 

2. On your IoT Edge device, run PowerShell as an administrator. 

3. Download and install the IoT Edge runtime. 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Install-SecurityDaemon -Dps -ContainerOs Windows
   ```

4. When prompted, provide the **ScopeID** and **RegistrationID** for your provisioning service and device. 

## Verify successful installation

You can check the status of the IoT Edge service by: 

```powershell
Get-Service iotedge
```

Examine service logs for the last 5 minutes using:

```powershell

# Displays logs from last 5 min, newest at the bottom.

Get-WinEvent -ea SilentlyContinue `
  -FilterHashtable @{ProviderName= "iotedged";
    LogName = "application"; StartTime = [datetime]::Now.AddMinutes(-5)} |
  select TimeCreated, Message |
  sort-object @{Expression="TimeCreated";Descending=$false} |
  format-table -autosize -wrap
```

And, list running modules with:

```powershell
iotedge list
```

## Next steps

Now that you have an IoT Edge device provisioned with the runtime installed, you can [deploy IoT Edge modules][lnk-modules].

If you are having problems with the Edge runtime installing properly, check out the [troubleshooting][lnk-trouble] page.


<!-- Images -->
[img-nat]: ./media/how-to-install-iot-edge-windows-with-windows/nat.png

<!-- Links -->
[lnk-docker-config]: https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers
[lnk-dcs]: how-to-register-device-portal.md
[lnk-dps]: how-to-auto-provision-simulated-device-windows.md
[lnk-oci]: https://www.opencontainers.org/
[lnk-moby]: https://mobyproject.org/
[lnk-trouble]: troubleshoot.md
[lnk-docker-for-windows]: https://www.docker.com/docker-windows
[lnk-iot-core]: how-to-install-iot-core.md
[lnk-modules]: how-to-deploy-modules-portal.md
