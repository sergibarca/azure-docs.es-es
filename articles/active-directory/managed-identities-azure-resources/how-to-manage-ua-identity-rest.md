---
title: 'Administración de identidades administradas asignadas por el usuario mediante REST: Azure AD'
description: Instrucciones paso a paso sobre cómo crear, enumerar y eliminar una identidad administrada asignada por el usuario para hacer llamadas a la API REST.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/26/2018
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 39e108451e4c19e77e01b5bcc5d8dd21e86ad73a
ms.sourcegitcommit: a678f00c020f50efa9178392cd0f1ac34a86b767
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/26/2019
ms.locfileid: "74547429"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-rest-api-calls"></a>Crear, enumerar o eliminar una identidad administrada asignada por el usuario mediante llamadas a la API REST

[!INCLUDE [preview-notice](~/includes/active-directory-msi-preview-notice-ua.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure la capacidad de autenticar servicios que admitan la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo, aprenderá a crear, enumerar y eliminar una identidad administrada asignada por el usuario mediante CURL para hacer llamadas a la API REST.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#how-does-the-managed-identities-for-azure-resources-work)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Si usa Windows, instale el [subsistema de Windows para Linux](https://msdn.microsoft.com/commandline/wsl/about) o [Azure Cloud Shell](../../cloud-shell/overview.md) en Azure Portal.
- Si utiliza el [subsistema de Windows para Linux](https://msdn.microsoft.com/commandline/wsl/about) o un [sistema operativo de distribución de Linux](/cli/azure/install-azure-cli-apt?view=azure-cli-latest), [instale la consola local de la CLI de Azure](/cli/azure/install-azure-cli).
- Si utiliza la consola local de la CLI de Azure, inicie sesión en Azure mediante `az login` con una cuenta asociada a la suscripción de Azure de la que desea implementar o recuperar la información de identidad administrada asignada por el usuario.
- Recupere un token de acceso de portador mediante `az account get-access-token` para realizar las siguientes operaciones de identidad administrada asignada por el usuario.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario 

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X PUT -d '{"loc
ation": "<LOCATION>"}' -H "Content-Type: application/json" -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
PUT https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```

**Encabezados de solicitud**

|Encabezado de solicitud  |DESCRIPCIÓN  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

**Cuerpo de la solicitud**

|NOMBRE  |DESCRIPCIÓN  |
|---------|---------|
|location     | Necesario. Ubicación del recurso        |

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](/azure/role-based-access-control/built-in-roles#managed-identity-operator) o [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview' -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
GET https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities?api-version=2015-08-31-preview HTTP/1.1
```

|Encabezado de solicitud  |DESCRIPCIÓN  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

> [!NOTE]
> La eliminación de una identidad administrada asignada por el usuario no quitará la referencia de ningún recurso al que se haya asignado. Para quitar una identidad administrada asignada por el usuario desde una máquina virtual mediante CURL, consulte [Eliminar una identidad asignada por el usuario de una máquina virtual de Azure](qs-configure-rest-vm.md#remove-a-user-assigned identity-from-an-azure-vm).

```bash
curl 'https://management.azure.com/subscriptions/<SUBSCRIPTION ID>/resourceGroup
s/<RESOURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview' -X DELETE -H "Authorization: Bearer <ACCESS TOKEN>"
```

```HTTP
DELETE https://management.azure.com/subscriptions/80c696ff-5efa-4909-a64d-f1b616f423ca/resourceGroups/TestRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>?api-version=2015-08-31-preview HTTP/1.1
```
|Encabezado de solicitud  |DESCRIPCIÓN  |
|---------|---------|
|*Content-Type*     | Necesario. Establézcalo en `application/json`.        |
|*Autorización*     | Necesario. Establézcalo en un token de acceso `Bearer` válido.        |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo asignar una identidad asignada por el usuario a una máquina virtual o a un conjunto de escalado de máquinas virtuales de Azure mediante CURL, consulte cómo [configurar las identidades administradas para los recursos de Azure en una máquina virtual de Azure mediante las llamadas a API REST](qs-configure-rest-vm.md#user-assigned-managed-identity) y cómo [configurar las identidades administradas para los recursos de Azure en un conjunto de escalado de máquinas virtuales mediante las llamadas a API REST](qs-configure-rest-vmss.md#user-assigned-managed-identity).
