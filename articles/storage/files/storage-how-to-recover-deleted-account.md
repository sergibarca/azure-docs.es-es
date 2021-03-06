---
title: Uso de un recurso compartido de archivos de Azure con Azure Storage | Microsoft Docs
description: Aprenda a usar un recurso compartido de archivos de Azure con Windows y Windows Server.
author: todmccoy
manager: dcscontentpm
ms.service: storage
ms.topic: conceptual
ms.date: 11/19/2019
ms.author: rogarana
ms.subservice: files
services: storage
tags: ''
ms.openlocfilehash: e1e581134b4e3821659c43bb7f55003239594bd9
ms.sourcegitcommit: d6b68b907e5158b451239e4c09bb55eccb5fef89
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/20/2019
ms.locfileid: "74234453"
---
# <a name="how-to-recover-a-deleted-storage-account"></a>Recuperación de una cuenta de almacenamiento eliminada

Azure Storage proporciona resistencia de datos mediante réplicas automatizadas, pero no impide que los usuarios o el código de la aplicación dañen los datos, ya sea de forma accidental o malintencionada. Preservar la fidelidad de los datos en caso de error de la aplicación o del usuario requiere técnicas más avanzadas, como copiar los datos en una ubicación de almacenamiento secundario con un registro de auditoría.

En la tabla siguiente se proporciona información general sobre el ámbito de la recuperación de la cuenta de almacenamiento en función de la estrategia de replicación.

| |LRS|ZRS|GRS|RA - GRS|
|---|---|---|---|---|
|Cuenta de almacenamiento con Azure Resource Manager|Sí|Sí|Sí|Sí|
|Cuenta de almacenamiento clásica|Sí|Sí|Sí|Sí|

Recopile la información siguiente y abra una solicitud de soporte técnico con Soporte técnico de Microsoft:

* Nombre de la cuenta de almacenamiento
* Fecha de eliminación
* Región de la cuenta de almacenamiento
* ¿Cómo se eliminó la cuenta de almacenamiento?
* ¿Qué método eliminó la cuenta de almacenamiento? (Portal, PowerShell, etc.)

Observaciones importantes

* Por lo general, pueden transcurrir hasta 15 días desde el momento de la eliminación hasta que el servicio de almacenamiento puede realizar una recolección de elementos no utilizados, por lo que la recuperación de las cuentas de almacenamiento no se puede realizar con un Acuerdo de Nivel de Servicio.
* El equipo de Soporte técnico de Microsoft hará todo lo posible para recuperar el contenedor o la cuenta pero no puede garantizar la recuperación.

> [!NOTE]
> Es posible que la recuperación no se realice correctamente si se ha vuelto a crear la cuenta. Si ya ha vuelto a crear la cuenta, debe eliminarla antes de que se pueda intentar la recuperación.
