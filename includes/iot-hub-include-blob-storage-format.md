---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 03/15/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 27a254b664e2035cc66109f7a473a3e9ef4d730e
ms.sourcegitcommit: c62a68ed80289d0daada860b837c31625b0fa0f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/05/2019
ms.locfileid: "73612171"
---
<!-- This is the note explaining about the avro and json formats when routing to blob storage. -->
> [!NOTE]
> Se pueden escribir datos en Blob Storage con formato [Apache Avro](https://avro.apache.org/), que es el predeterminado, o JSON (versión preliminar). 
>    
> La funcionalidad de codificar en formato JSON está en versión preliminar en todas las regiones donde IoT Hub está disponible, excepto en el Este de Estados Unidos, el Oeste de Estados Unidos y Europa occidental. El formato de codificación solo se puede establecer en el momento en que se configura el punto de conexión de Blob Storage. No se puede cambiar el formato de un punto de conexión que ya se ha configurado. Cuando se usa la codificación JSON, debe establecer contentType en JSON y contentEncoding en UTF-8 en las propiedades del sistema de mensajes. 
>
> Para obtener más información sobre el uso de un punto de conexión de Blob Storage, consulte la [guía sobre el enrutamiento Azure Storage](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#azure-storage).
>