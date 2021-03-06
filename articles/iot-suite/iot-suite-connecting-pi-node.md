---
title: "Provisionner un appareil Raspberry Pi pour la surveillance à distance en Node.js - Azure| Microsoft Docs"
description: "Explique comment connecter un appareil Raspberry Pi à la solution de surveillance à distance Azure IoT Suite préconfigurée à l’aide d’une application écrite en Node.js."
services: iot-suite
suite: iot-suite
documentationcenter: na
author: dominicbetts
manager: timlt
editor: 
ms.assetid: fc50a33f-9fb9-42d7-b1b8-eb5cff19335e
ms.service: iot-suite
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/18/2017
ms.author: dobett
ms.openlocfilehash: d401dde25bf4ab430ac045fb6cfd90050a7ec2e7
ms.sourcegitcommit: dfd49613fce4ce917e844d205c85359ff093bb9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/31/2017
---
# <a name="connect-your-raspberry-pi-device-to-the-remote-monitoring-preconfigured-solution-nodejs"></a>Connecter votre appareil Raspberry Pi à la solution préconfigurée de surveillance à distance (Node.js)

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

Ce didacticiel montre comment connecter un appareil physique à la solution préconfigurée de surveillance à distance. Dans ce didacticiel, vous utilisez Node.js, qui est une bonne option pour les environnements avec des contraintes minimales en ressources.

### <a name="required-hardware"></a>Matériel requis

Un ordinateur de bureau permettant de vous connecter à distance à la ligne de commande sur le Raspberry Pi.

[Starter Kit Microsoft IoT pour Raspberry Pi 3](https://azure.microsoft.com/develop/iot/starter-kits/) ou composants équivalents. Ce didacticiel utilise les éléments suivants du kit :

- Raspberry Pi 3
- Carte MicroSD (avec NOOBS)
- Un câble mini USB
- Un câble Ethernet

### <a name="required-desktop-software"></a>Logiciels de bureau requis

Vous devez installer le client SSH sur votre ordinateur de bureau afin de pouvoir accéder à distance à la ligne de commande sur le Raspberry Pi.

- Windows n’inclut pas de client SSH. Nous vous recommandons d’utiliser [PuTTY](http://www.putty.org/).
- La plupart des distributions Linux et Mac OS incluent l’utilitaire de ligne de commande SSH. Pour plus d’informations, consultez [SSH Using Linux or Mac OS](https://www.raspberrypi.org/documentation/remote-access/ssh/unix.md) (Utilisation de SSH avec Linux ou Mac OS).

### <a name="required-raspberry-pi-software"></a>Logiciels Raspberry Pi requis

Si ce n’est déjà fait, installez Node.js version 4.0.0 ou ultérieure sur votre appareil Raspberry Pi. Les étapes suivantes vous montrent comment installer Node.js v6.11.4 sur votre appareil Raspberry Pi :

1. Connectez-vous à votre appareil Raspberry Pi avec `ssh`. Pour plus d’informations, consultez [SSH (Secure Shell)](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md) sur le [site web de Raspberry Pi](https://www.raspberrypi.org/).

1. Pour mettre à jour votre Raspberry Pi, utilisez la commande suivante :

    ```sh
    sudo apt-get update
    ```

1. Pour télécharger les fichiers binaires Node.js sur votre Raspberry Pi, utilisez la commande suivante :

    ```sh
    wget https://nodejs.org/dist/v6.11.4/node-v6.11.4-linux-armv7l.tar.gz
    ```

1. Pour installer les binaires, utilisez la commande suivante :

    ```sh
    sudo tar -C /usr/local --strip-components 1 -xzf node-v6.11.4-linux-armv7l.tar.gz
    ```

1. Pour vérifier que Node.js v6.11.4 a bien été installé, utilisez la commande suivante :

    ```sh
    node --version
    ```

## <a name="create-a-nodejs-solution"></a>Créer une application Node.js

Effectuez les étapes suivantes en utilisant la connexion `ssh` à votre appareil Raspberry Pi :

1. Créez un dossier appelé `RemoteMonitoring` dans votre dossier de base sur l’appareil Raspberry Pi. Accédez à ce dossier dans votre ligne de commande :

    ```sh
    cd ~
    mkdir RemoteMonitoring
    cd RemoteMonitoring
    ```

1. Pour télécharger et installer les packages dont vous avez besoin pour accomplir l’exemple d’application, exécutez les commandes suivantes :

    ```sh
    npm init
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

1. Dans le dossier `RemoteMonitoring`, créez un fichier appelé **remote_monitoring.js**. Ouvrez ce fichier dans un éditeur de texte. Sur l’appareil Raspberry Pi, vous pouvez utiliser l’éditeur de texte `nano` ou `vi`.

1. Dans le fichier **remote_monitoring.js**, ajoutez les instructions `require` suivantes :

    ```nodejs
    'use strict';

    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var ConnectionString = require('azure-iot-device').ConnectionString;
    var Message = require('azure-iot-device').Message;
    ```

1. Ajoutez les déclarations de variables suivantes après les instructions `require` . Remplacez les valeurs d’espace réservé `{Device Id}` et `{Device Key}` par les valeurs que vous avez notées pour l’appareil que vous avez provisionné dans la solution de surveillance à distance. Utilisez le nom d’hôte IoT Hub de la solution pour remplacer `{IoTHub Name}`. Par exemple, si votre nom d’hôte IoT Hub est `contoso.azure-devices.net`, remplacez `{IoTHub Name}` par `contoso` :

    ```nodejs
    var connectionString = 'HostName={IoTHub Name}.azure-devices.net;DeviceId={Device Id};SharedAccessKey={Device Key}';
    var deviceId = ConnectionString.parse(connectionString).DeviceId;
    ```

1. Pour définir des données de télémétrie de base, ajoutez les variables suivantes :

    ```nodejs
    var temperature = 50;
    var temperatureUnit = 'F';
    var humidity = 50;
    var humidityUnit = '%';
    var pressure = 55;
    var pressureUnit = 'psig';
    ```

1. Pour définir des valeurs de propriété, ajoutez les variables suivantes :

    ```nodejs
    var temperatureSchema = 'chiller-temperature;v1';
    var humiditySchema = 'chiller-humidity;v1';
    var pressureSchema = 'chiller-pressure;v1';
    var interval = "00:00:05";
    var deviceType = "Chiller";
    var deviceFirmware = "1.0.0";
    var deviceFirmwareUpdateStatus = "";
    var deviceLocation = "Building 44";
    var deviceLatitude = 47.638928;
    var deviceLongitude = -122.13476;
    ```

1. Ajoutez la variable suivante pour définir les propriétés déclarées à envoyer à la solution. Ces propriétés incluent les métadonnées décrivant les méthodes et les données de télémétrie utilisées par l’appareil :

    ```nodejs
    var reportedProperties = {
      "Protocol": "MQTT",
      "SupportedMethods": "Reboot,FirmwareUpdate,EmergencyValveRelease,IncreasePressure",
      "Telemetry": {
        "TemperatureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\"}",
          "MessageSchema": {
            "Name": temperatureSchema,
            "Format": "JSON",
            "Fields": {
              "temperature": "Double",
              "temperature_unit": "Text"
            }
          }
        },
        "HumiditySchema": {
          "Interval": interval,
          "MessageTemplate": "{\"humidity\":${humidity},\"humidity_unit\":\"${humidity_unit}\"}",
          "MessageSchema": {
            "Name": humiditySchema,
            "Format": "JSON",
            "Fields": {
              "humidity": "Double",
              "humidity_unit": "Text"
            }
          }
        },
        "PressureSchema": {
          "Interval": interval,
          "MessageTemplate": "{\"pressure\":${pressure},\"pressure_unit\":\"${pressure_unit}\"}",
          "MessageSchema": {
            "Name": pressureSchema,
            "Format": "JSON",
            "Fields": {
              "pressure": "Double",
              "pressure_unit": "Text"
            }
          }
        }
      },
      "Type": deviceType,
      "Firmware": deviceFirmware,
      "FirmwareUpdateStatus": deviceFirmwareUpdateStatus,
      "Location": deviceLocation,
      "Latitude": deviceLatitude,
      "Longitude": deviceLongitude
    }
    ```

1. Pour imprimer les résultats de l’opération, ajoutez la fonction d’assistance suivante :

    ```nodejs
    function printErrorFor(op) {
        return function printError(err) {
            if (err) console.log(op + ' error: ' + err.toString());
        };
    }
    ```

1. Ajoutez la fonction d’assistance suivante qui permet de rendre aléatoires les valeurs de télémétrie :

    ```nodejs
    function generateRandomIncrement() {
        return ((Math.random() * 2) - 1);
    }
    ```

1. Ajoutez la fonction suivante pour gérer les appels de méthode directe à partir de la solution. La solution utilise des méthodes directes pour agir sur les appareils :

    ```nodejs
    function onDirectMethod(request, response) {
      // Implement logic asynchronously here.
      console.log('Simulated ' + request.methodName);

      // Complete the response
      response.send(200, request.methodName + ' was called on the device', function (err) {
        if (!!err) {
          console.error('An error ocurred when sending a method response:\n' +
            err.toString());
        } else {
          console.log('Response to method \'' + request.methodName +
            '\' sent successfully.');
        }
      });
    }
    ```

1. Ajoutez le code suivant pour envoyer les données de télémétrie à la solution. L’application cliente ajoute des propriétés au message pour identifier le schéma du message :

    ```node.js
    function sendTelemetry(data, schema) {
      var d = new Date();
      var payload = JSON.stringify(data);
      var message = new Message(payload);
      message.properties.add('$$CreationTimeUtc', d.toISOString());
      message.properties.add('$$MessageSchema', schema);
      message.properties.add('$$ContentType', 'JSON');

      console.log('Sending device message data:\n' + payload);
      client.sendEvent(message, printErrorFor('send event'));
    }
    ```

1. Ajoutez le code suivant pour créer une instance de client :

    ```nodejs
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

1. Ajoutez le code suivant à :

    - Ouvrir la connexion.
    - Définir un gestionnaire pour les propriétés souhaitées.
    - Envoyer les propriétés signalées.
    - Inscrire des gestionnaires pour les méthodes directes.
    - Démarrer l’envoi de la télémétrie.

    ```nodejs
    client.open(function (err) {
      if (err) {
        printErrorFor('open')(err);
      } else {
        // Create device Twin
        client.getTwin(function (err, twin) {
          if (err) {
            console.error('Could not get device twin');
          } else {
            console.log('Device twin created');

            twin.on('properties.desired', function (delta) {
              // Handle desired properties set by solution
              console.log('Received new desired properties:');
              console.log(JSON.stringify(delta));
            });

            // Send reported properties
            twin.properties.reported.update(reportedProperties, function (err) {
              if (err) throw err;
              console.log('twin state reported');
            });

            // Register handlers for all the method names we are interested in.
            // Consider separate handlers for each method.
            client.onDeviceMethod('Reboot', onDirectMethod);
            client.onDeviceMethod('FirmwareUpdate', onDirectMethod);
            client.onDeviceMethod('EmergencyValveRelease', onDirectMethod);
            client.onDeviceMethod('IncreasePressure', onDirectMethod);
          }
        });

        // Start sending telemetry
        var sendTemperatureInterval = setInterval(function () {
          temperature += generateRandomIncrement();
          var data = {
            'temperature': temperature,
            'temperature_unit': temperatureUnit
          };
          sendTelemetry(data, temperatureSchema)
        }, 5000);

        var sendHumidityInterval = setInterval(function () {
          humidity += generateRandomIncrement();
          var data = {
            'humidity': humidity,
            'humidity_unit': humidityUnit
          };
          sendTelemetry(data, humiditySchema)
        }, 5000);

        var sendPressureInterval = setInterval(function () {
          pressure += generateRandomIncrement();
          var data = {
            'pressure': pressure,
            'pressure_unit': pressureUnit
          };
          sendTelemetry(data, pressureSchema)
        }, 5000);

        client.on('error', function (err) {
          printErrorFor('client')(err);
          if (sendTemperatureInterval) clearInterval(sendTemperatureInterval);
          if (sendHumidityInterval) clearInterval(sendHumidityInterval);
          if (sendPressureInterval) clearInterval(sendPressureInterval);
          client.close(printErrorFor('client.close'));
        });
      }
    });
    ```

1. Enregistrez les modifications dans le fichier **remote_monitoring.js**.

1. Pour démarrer l’exemple d’application, exécutez la commande suivante depuis l’invite de commande sur l’appareil Raspberry Pi :

    ```sh
    node remote_monitoring.js
    ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]
