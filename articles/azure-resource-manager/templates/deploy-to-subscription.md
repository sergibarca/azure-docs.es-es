---
title: Implementación de recursos en una suscripción
description: Se describe cómo crear un grupo de recursos en una plantilla de Azure Resource Manager. También se muestra cómo implementar recursos en el ámbito de la suscripción de Azure.
ms.topic: conceptual
ms.date: 02/10/2020
ms.openlocfilehash: 50db0b4d46ff4e367411829aa75fa017a168372f
ms.sourcegitcommit: 2823677304c10763c21bcb047df90f86339e476a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2020
ms.locfileid: "77207662"
---
# <a name="create-resource-groups-and-resources-at-the-subscription-level"></a>Creación de grupos de recursos y otros recursos en el nivel de suscripción

Normalmente, implementa los recursos de Azure en un grupo de recursos en su suscripción de Azure. Sin embargo, también puede crear recursos en el nivel de suscripción. Las implementaciones de nivel de suscripción se usan para realizar acciones que tienen sentido en ese nivel, como la creación de grupos de recursos o la asignación del [control de acceso basado en rol](../../role-based-access-control/overview.md).

Para implementar plantillas en el nivel de suscripción, use la CLI de Azure, PowerShell o la API REST. Azure Portal no admite la implementación en el nivel de suscripción.

## <a name="supported-resources"></a>Recursos compatibles

Puede implementar los siguientes tipos de recursos en el nivel de suscripción:

* [deployments](/azure/templates/microsoft.resources/deployments)
* [peerAsns](/azure/templates/microsoft.peering/peerasns)
* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)
* [resourceGroups](/azure/templates/microsoft.resources/resourcegroups)
* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)
* [roleDefinitions](/azure/templates/microsoft.authorization/roledefinitions)

### <a name="schema"></a>Schema

El esquema que se usa para las implementaciones de nivel de suscripción es diferente del esquema de las implementaciones de grupo de recursos.

Para las plantillas, use:

```json
https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#
```

Para los archivos de parámetros, use:

```json
https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentParameters.json#
```

## <a name="deployment-commands"></a>Comandos de implementación

Los comandos para las implementaciones de nivel de suscripción son diferentes de los comandos de las implementaciones de grupo de recursos.

Para la CLI de Azure, use [az deployment create](/cli/azure/deployment?view=azure-cli-latest#az-deployment-create). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurecli-interactive
az deployment create \
  --name demoDeployment \
  --location centralus \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json \
  --parameters rgName=demoResourceGroup rgLocation=centralus
```


Para el comando de implementación de PowerShell, use [New-AzDeployment](/powershell/module/az.resources/new-azdeployment). El ejemplo siguiente implementa una plantilla para crear un grupo de recursos:

```azurepowershell-interactive
New-AzDeployment `
  -Name demoDeployment `
  -Location centralus `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/emptyRG.json `
  -rgName demoResourceGroup `
  -rgLocation centralus
```

Para la API REST, use [Deployments: Create At Subscription Scope](/rest/api/resources/deployments/createorupdateatsubscriptionscope).

## <a name="deployment-location-and-name"></a>Ubicación y nombre de la implementación

En el caso de las implementaciones de nivel de suscripción, debe proporcionar una ubicación para la implementación. La ubicación de la implementación es independiente de la ubicación de los recursos que se implementan. La ubicación de implementación especifica dónde se almacenarán los datos de la implementación.

Puede proporcionar un nombre para la implementación o usar el nombre de implementación predeterminado. El nombre predeterminado es el nombre del archivo de plantilla. Por ejemplo, al implementar una plantilla llamada **azuredeploy.json**, se crea un nombre de predeterminado **azuredeploy**.

Para cada nombre de implementación, la ubicación es inmutable. No se puede crear una implementación en una ubicación si ya existe una implementación con el mismo nombre en otra ubicación. Si recibe el código de error `InvalidDeploymentLocation`, use un nombre diferente o utilice la ubicación de la implementación anterior que tenía ese mismo nombre.

## <a name="use-template-functions"></a>Usar funciones de plantillas

En las implementaciones de nivel de suscripción, hay algunas consideraciones importantes que deben tenerse en cuenta al usar las funciones de plantilla:

* La función [resourceGroup()](template-functions-resource.md#resourcegroup)**no** se admite.
* Se admiten las funciones [reference()](template-functions-resource.md#reference) y [list()](template-functions-resource.md#list).
* La función [resourceId()](template-functions-resource.md#resourceid) sí se admite. Utilícela para obtener el identificador de los recursos que se utilizan en las implementaciones de nivel de suscripción. No proporcione un valor para el parámetro del grupo de recursos.

  Por ejemplo, para obtener el identificador de recurso de una definición de directiva, utilice:
  
  ```json
  resourceId('Microsoft.Authorization/roleDefinitions/', parameters('roleDefinition'))
  ```
  
  El identificador de recurso devuelto tiene el formato siguiente:

  ```json
  /subscriptions/{subscriptionId}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
  ```

  O bien, use la función [subscriptionResourceId ()](template-functions-resource.md#subscriptionresourceid) para obtener el id. de recurso para un recurso de nivel de suscripción.

## <a name="create-resource-groups"></a>Crear grupos de recursos

Para crear un grupo de recursos en una plantilla de Azure Resource Manager, defina un recurso [Microsoft.Resources/resourceGroups](/azure/templates/microsoft.resources/allversions) con un nombre y una ubicación para el grupo de recursos. También se puede crear un grupo de recursos e implementar recursos en él en la misma plantilla.

En la plantilla siguiente se crea un grupo de recursos vacío.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2018-05-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('rgLocation')]",
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Use el [elemento copy](copy-resources.md) con grupos de recursos para crear más de un grupo de recursos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgNamePrefix": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "instanceCount": {
      "type": "int"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2018-05-01",
      "location": "[parameters('rgLocation')]",
      "name": "[concat(parameters('rgNamePrefix'), copyIndex())]",
      "copy": {
        "name": "rgCopy",
        "count": "[parameters('instanceCount')]"
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Para más información sobre la iteración de recursos, consulte este artículo sobre la [implementación de varias instancias de un recurso en las plantillas de Azure Resource Manager](./copy-resources.md) y [Tutorial: Creación de varias instancias de recursos con plantillas de Resource Manager](./template-tutorial-create-multiple-instances.md).

## <a name="resource-group-and-resources"></a>Grupo de recursos y recursos

Para crear el grupo de recursos e implementar recursos en él, utilice una plantilla anidada. La plantilla anidada define los recursos que se van a implementar en el grupo de recursos. Establezca la plantilla anidada como dependiente del grupo de recursos para asegurarse de que el grupo de recursos existe antes de implementar los recursos.

En el ejemplo siguiente se crea un grupo de recursos y se implementa una cuenta de almacenamiento en el grupo de recursos.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string"
    },
    "rgLocation": {
      "type": "string"
    },
    "storagePrefix": {
      "type": "string",
      "maxLength": 11
    }
  },
  "variables": {
    "storageName": "[concat(parameters('storagePrefix'), uniqueString(subscription().id, parameters('rgName')))]"
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2018-05-01",
      "location": "[parameters('rgLocation')]",
      "name": "[parameters('rgName')]",
      "properties": {}
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "name": "storageDeployment",
      "resourceGroup": "[parameters('rgName')]",
      "dependsOn": [
        "[resourceId('Microsoft.Resources/resourceGroups/', parameters('rgName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
            {
              "type": "Microsoft.Storage/storageAccounts",
              "apiVersion": "2017-10-01",
              "name": "[variables('storageName')]",
              "location": "[parameters('rgLocation')]",
              "sku": {
                "name": "Standard_LRS"
              }
              "kind": "StorageV2",
            }
          ],
          "outputs": {}
        }
      }
    }
  ],
  "outputs": {}
}
```

## <a name="create-policies"></a>Creación de directivas

### <a name="assign-policy"></a>Asignación de directiva

En el ejemplo siguiente se asigna una definición de directiva existente a la suscripción. Si la directiva toma parámetros, proporciónelos como un objeto. Si la directiva no toma parámetros, use el objeto vacío predeterminado.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "policyDefinitionID": {
      "type": "string"
    },
    "policyName": {
      "type": "string"
    },
    "policyParameters": {
      "type": "object",
      "defaultValue": {}
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-03-01",
      "name": "[parameters('policyName')]",
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[parameters('policyDefinitionID')]",
        "parameters": "[parameters('policyParameters')]"
      }
    }
  ]
}
```

Para implementar esta plantilla con la CLI de Azure, use:

```azurecli-interactive
# Built-in policy that accepts parameters
definition=$(az policy definition list --query "[?displayName=='Allowed locations'].id" --output tsv)

az deployment create \
  --name demoDeployment \
  --location centralus \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json \
  --parameters policyDefinitionID=$definition policyName=setLocation policyParameters="{'listOfAllowedLocations': {'value': ['westus']} }"
```

Para implementar esta plantilla con PowerShell, use:

```azurepowershell-interactive
$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq 'Allowed locations' }

$locations = @("westus", "westus2")
$policyParams =@{listOfAllowedLocations = @{ value = $locations}}

New-AzDeployment `
  -Name policyassign `
  -Location centralus `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policyassign.json `
  -policyDefinitionID $definition.PolicyDefinitionId `
  -policyName setLocation `
  -policyParameters $policyParams
```

### <a name="define-and-assign-policy"></a>Definición y asignación de directivas

Puede [definir](../../governance/policy/concepts/definition-structure.md) y asignar una directiva en la misma plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Authorization/policyDefinitions",
      "apiVersion": "2018-05-01",
      "name": "locationpolicy",
      "properties": {
        "policyType": "Custom",
        "parameters": {},
        "policyRule": {
          "if": {
            "field": "location",
            "equals": "northeurope"
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    },
    {
      "type": "Microsoft.Authorization/policyAssignments",
      "apiVersion": "2018-05-01",
      "name": "location-lock",
      "dependsOn": [
        "locationpolicy"
      ],
      "properties": {
        "scope": "[subscription().id]",
        "policyDefinitionId": "[resourceId('Microsoft.Authorization/policyDefinitions', 'locationpolicy')]"
      }
    }
  ]
}
```

Para crear la definición de directiva en su suscripción y aplicarla a la suscripción, use el siguiente comando de la CLI:

```azurecli
az deployment create \
  --name demoDeployment \
  --location centralus \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json
```

Para implementar esta plantilla con PowerShell, use:

```azurepowershell
New-AzDeployment `
  -Name definePolicy `
  -Location centralus `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/policydefineandassign.json
```

## <a name="template-samples"></a>Ejemplos de plantillas

* Cree un grupo de recursos, bloquéelo y concédale permisos. Consulte [aquí](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-level-deployments/create-rg-lock-role-assignment).
* Cree un grupo de recursos, una directiva y una asignación de directiva.  Consulte [aquí](https://github.com/Azure/azure-docs-json-samples/blob/master/subscription-level-deployment/azuredeploy.json).

## <a name="next-steps"></a>Pasos siguientes

* Para aprender sobre los roles de asignación, consulte [Administración del acceso a los recursos de Azure mediante RBAC y plantillas de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md).
* Para un ejemplo de implementación de la configuración del área de trabajo para Azure Security Center, consulte [deployASCwithWorkspaceSettings.json](https://github.com/krnese/AzureDeploy/blob/master/ARM/deployments/deployASCwithWorkspaceSettings.json).
* Puede encontrar plantillas de ejemplo en [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/subscription-level-deployments).
* Para más información sobre la creación de plantillas del Administrador de recursos de Azure, consulte [Creación de plantillas](template-syntax.md).
* Para obtener una lista de las funciones disponibles en una plantilla, consulte [Funciones de plantilla](template-functions.md).
