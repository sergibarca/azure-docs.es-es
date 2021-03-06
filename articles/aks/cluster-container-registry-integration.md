---
title: Integración de Azure Container Registry con Azure Kubernetes Service
description: Obtenga información sobre cómo integrar Azure Kubernetes Service (AKS) con Azure Container Registry (ACR).
services: container-service
manager: gwallace
ms.topic: article
ms.date: 09/17/2018
ms.openlocfilehash: b1f4449728589eca4f64035d7e70d01dbc187bc4
ms.sourcegitcommit: 99ac4a0150898ce9d3c6905cbd8b3a5537dd097e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/25/2020
ms.locfileid: "77596205"
---
# <a name="authenticate-with-azure-container-registry-from-azure-kubernetes-service"></a>Autenticación con Azure Container Registry desde Azure Kubernetes Service

Cuando se usa Azure Container Registry (ACR) con Azure Kubernetes Service (AKS), es preciso establecer un mecanismo de autenticación. En este artículo se proporcionan ejemplos para configurar la autenticación entre estos dos servicios de Azure.

Puede configurar la integración de AKS en ACR con unos pocos comandos sencillos con la CLI de Azure.

## <a name="before-you-begin"></a>Antes de empezar

Estos ejemplos requieren:

* Rol de **propietario** o **administrador de cuenta de Azure** en la **suscripción de Azure**.
* La CLI de Azure, versión 2.0.73 o posterior

## <a name="create-a-new-aks-cluster-with-acr-integration"></a>Creación de un nuevo clúster de AKS con integración de ACR

Puede configurar la integración de AKS y ACR durante la creación inicial del clúster de AKS.  Para permitir que un clúster de AKS interactúe con ACR, se usa una **entidad de servicio** de Azure Active Directory. El siguiente comando CLI permite autorizar un ACR existente en su suscripción y configura el rol **ACRPull** adecuado para la entidad de servicio. Proporcione valores válidos para los parámetros siguientes. 
```azurecli
# set this to the name of your Azure Container Registry.  It must be globally unique
MYACR=myContainerRegistry

# Run the following line to create an Azure Container Registry if you do not already have one
az acr create -n $MYACR -g myContainerRegistryResourceGroup --sku basic

# Create an AKS cluster with ACR integration
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr $MYACR

```
El nombre de ACR se puede especificar mediante un identificador de recursos de ACR, que tiene el siguiente formato:

/subscriptions/\<subscription-id\>/resourceGroups/\<resource-group-name\>/providers/Microsoft.ContainerRegistry/registries/\<name\> 
 
```azurecli
az aks create -n myAKSCluster -g myResourceGroup --generate-ssh-keys --attach-acr /subscriptions/<subscription-id>/resourceGroups/myContainerRegistryResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry
```

Este paso puede tardar varios minutos en completarse.

## <a name="configure-acr-integration-for-existing-aks-clusters"></a>Configuración de la integración de ACR para clústeres de AKS existentes

Para integrar un ACR existente con clústeres de AKS existentes, proporcione valores válidos para **acr-name** o **acr-resource-id**, como se indica a continuación.

```azurecli
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acrName>
```
o bien,
```
az aks update -n myAKSCluster -g myResourceGroup --attach-acr <acr-resource-id>
```

También puede quitar la integración entre un grupo de ACR y un clúster de AKS con lo siguiente
```azurecli
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acrName>
```
or
```
az aks update -n myAKSCluster -g myResourceGroup --detach-acr <acr-resource-id>
```

## <a name="working-with-acr--aks"></a>Uso de ACR y AKS

### <a name="import-an-image-into-your-acr"></a>Importación de una imagen en ACR

Importe una imagen de Docker Hub en ACR mediante la ejecución del código siguiente:


```azurecli
az acr import  -n <myContainerRegistry> --source docker.io/library/nginx:latest --image nginx:v1
```

### <a name="deploy-the-sample-image-from-acr-to-aks"></a>Implementación de la imagen de ejemplo de ACR en AKS

Asegúrese de que tiene las credenciales de AKS adecuadas

```azurecli
az aks get-credentials -g myResourceGroup -n myAKSCluster
```

Cree un archivo llamado **acr-nginx.yaml** que contenga el código siguiente:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx0-deployment
  labels:
    app: nginx0-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx0
  template:
    metadata:
      labels:
        app: nginx0
    spec:
      containers:
      - name: nginx
        image: <replace this image property with you acr login server, image and tag>
        ports:
        - containerPort: 80
```

Después, ejecute esta implementación en el clúster de AKS:
```
kubectl apply -f acr-nginx.yaml
```

Para supervisar la implementación, ejecute:
```
kubectl get pods
```
Debe tener dos pods en ejecución.
```
NAME                                 READY   STATUS    RESTARTS   AGE
nginx0-deployment-669dfc4d4b-x74kr   1/1     Running   0          20s
nginx0-deployment-669dfc4d4b-xdpd6   1/1     Running   0          20s
```

<!-- LINKS - external -->
[AKS AKS CLI]:  https://docs.microsoft.com/cli/azure/aks?view=azure-cli-latest#az-aks-create
