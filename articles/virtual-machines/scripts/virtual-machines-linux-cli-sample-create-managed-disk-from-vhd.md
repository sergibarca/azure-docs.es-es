---
title: 'Creación de un disco administrado a partir de un archivo de VHD en una cuenta de almacenamiento en la misma suscripción: Ejemplo de la CLI'
description: 'Ejemplo de script de CLI de Azure: creación un disco administrado a partir de un archivo de VHD en una cuenta de almacenamiento en la misma suscripción'
services: virtual-machines-linux
documentationcenter: storage
author: ramankumarlive
manager: kavithag
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: d15e0519e9ee5e5d1d6832e37c10119e40add890
ms.sourcegitcommit: f4f626d6e92174086c530ed9bf3ccbe058639081
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/25/2019
ms.locfileid: "75458487"
---
# <a name="create-a-managed-disk-from-a-vhd-file-in-a-storage-account-in-the-same-subscription-with-cli"></a>Creación un disco administrado a partir de un archivo de VHD en una cuenta de almacenamiento en la misma suscripción con CLI

Este script crea un disco administrado a partir de un archivo de VHD en una cuenta de almacenamiento en la misma suscripción. Utilice este script para importar un archivo VHD especializado (no generalizado o preparado con sysprep) a un disco de sistema operativo administrado para crear una máquina virtual. O bien úselo para importar un VHD de datos en el disco de datos administrado. 


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-managed-data-disks-from-vhd/create-managed-data-disks-from-vhd.sh "Create managed disk from VHD")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un disco administrado a partir de un VHD. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az disk create](https://docs.microsoft.com/cli/azure/disk) | Crea un disco administrado usando el identificador URI de un VHD en una cuenta de almacenamiento en la misma suscripción. |

## <a name="next-steps"></a>Pasos siguientes

[Crear una máquina virtual conectando un disco administrado como disco del SO](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fcli%2fmodule%2ftoc.json)

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Linux](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
