---
title: Conectarse y administrar el dispositivo Microsoft Azure Data Box Gateway a través de la interfaz de Windows PowerShell | Microsoft Docs
description: Aquí se describe cómo puede conectarse y administrar Data Box Gateway a través de la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 08/02/2019
ms.author: alkohli
ms.openlocfilehash: 650777d849e172686e8b46502a84db8c519174e7
ms.sourcegitcommit: 4b5dcdcd80860764e291f18de081a41753946ec9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2019
ms.locfileid: "68775190"
---
# <a name="manage-an-azure-data-box-gateway-device-via-windows-powershell"></a>Administrar un dispositivo Azure Data Box Gateway a través de Windows PowerShell

La solución Azure Data Box Gateway le permite enviar datos a Azure a través de la red. En este artículo se describen algunas de las tareas de configuración y administración para su dispositivo Data Box Gateway. Puede usar Azure Portal, la interfaz de usuario web local o la interfaz de Windows PowerShell para administrar su dispositivo.

Este artículo se centra en las tareas que realizará mediante la interfaz de PowerShell.

Asimismo, en este artículo se incluyen las secciones siguientes:

- Conectarse a la interfaz de PowerShell
- Crear un paquete de soporte
- Carga del certificado
- Arrancar en un entorno sin DHCP
- Ver la información del dispositivo

## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Crear un paquete de soporte

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>Carga del certificado

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

## <a name="boot-up-in-non-dhcp-environment"></a>Arrancar en un entorno sin DHCP

[!INCLUDE [Boot up in non-DHCP environment](../../includes/data-box-edge-gateway-boot-non-dhcp.md)]

## <a name="view-device-information"></a>Ver la información del dispositivo

[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="next-steps"></a>Pasos siguientes

- Implemente [Azure Data Box Gateway](data-box-gateway-deploy-prep.md) en Azure Portal.
