---
title: Compilación de configuraciones en Azure Automation State Configuration
description: En este artículo se describe cómo compilar configuraciones de configuración de estado deseado (DSC) para Azure Automation.
services: automation
ms.subservice: dsc
ms.date: 09/10/2018
ms.topic: conceptual
ms.openlocfilehash: a4bdd28d2ad8f692b561d414af15b90b1609bac4
ms.sourcegitcommit: 6ee876c800da7a14464d276cd726a49b504c45c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/19/2020
ms.locfileid: "77462129"
---
# <a name="compiling-dsc-configurations-in-azure-automation-state-configuration"></a>Compilación de configuraciones de DSC en Azure Automation State Configuration

Azure Automation State Configuration permite compilar las configuraciones de Desired State Configuration (DSC) de dos formas: en Azure y en Windows PowerShell. La tabla siguiente le ayudará a determinar cuándo se debe usar cada método en base a las características de cada uno:

- Servicio de compilación de Azure State Configuration
  - Método para principiantes, con una interfaz de usuario interactiva
   - Seguimiento sencillo del estado del trabajo

- Windows PowerShell
  - Llamada desde Windows PowerShell en una estación de trabajo local o un servicio de compilación
  - Integración con la canalización de pruebas de desarrollo
  - Disponibilidad de valores de parámetro complejos
  - Trabajo con datos de nodo y datos que no son de nodo a escala
  - Mejora significativa del rendimiento

Para obtener información detallada sobre la compilación, consulte [Extensión Desired State Configuration con plantillas de Azure Resource Manager](https://docs.microsoft.com/azure/virtual-machines/extensions/dsc-template#details).

## <a name="compiling-a-dsc-configuration-in-azure-state-configuration"></a>Compilación de una configuración de DSC en Azure State Configuration

### <a name="portal"></a>Portal

1. En su cuenta de Automation, haga clic en **State Configuration (DSC)** .
1. Haga clic en la pestaña **Configuraciones** y, a continuación, haga clic en el nombre de la configuración que se va a compilar.
1. Haga clic en **Compilar**.
1. Si la configuración no tiene parámetros, se le pedirá que confirme que desea compilarla. Si la configuración tiene parámetros, se abrirá la hoja **Compilar configuración** para que pueda proporcionar valores de parámetros.
1. Se abre la página Trabajo de compilación para que pueda realizar el seguimiento del estado del trabajo de compilación. También puede usar esta página para realizar el seguimiento de las configuraciones de nodo (documentos de configuración MOF) que el trabajo coloca en el servidor de extracción de State Configuration de Azure Automation.

### <a name="azure-powershell"></a>Azure PowerShell

Puede usar [Start-AzAutomationDscCompilationJob](/powershell/module/az.automation/start-azautomationdsccompilationjob) para empezar a compilar con Windows PowerShell. El código de ejemplo siguiente inicia la compilación de una configuración de DSC llamada SampleConfig.

```powershell
Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'
```

**Start-AzAutomationDscCompilationJob** devuelve un objeto de trabajo de compilación que puede usar para realizar el seguimiento de su estado. Después, puede utilizar este objeto de trabajo de compilación con [Get-AzAutomationDscCompilationJob](/powershell/module/az.automation/get-azautomationdsccompilationjob) para determinar el estado del trabajo de compilación y con [Get-AzAutomationDscCompilationJobOutput](/powershell/module/az.automation/get-azautomationdscconfiguration) para ver sus flujos (salida). El código de ejemplo siguiente inicia la compilación de la configuración SampleConfig, espera hasta que finalice y, a continuación, muestra sus flujos.

```powershell
$CompilationJob = Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'SampleConfig'

while($null -eq $CompilationJob.EndTime -and $null -eq $CompilationJob.Exception)
{
    $CompilationJob = $CompilationJob | Get-AzAutomationDscCompilationJob
    Start-Sleep -Seconds 3
}

$CompilationJob | Get-AzAutomationDscCompilationJobOutput –Stream Any
```

### <a name="declare-basic-parameters"></a>Declaración de los parámetros básicos

La declaración de parámetro en configuraciones DSC, incluidos los tipos de parámetro y las propiedades, funciona igual que en los runbooks de Azure Automation. Vea [Inicio de un runbook en Azure Automation](automation-starting-a-runbook.md) para obtener más información acerca de los parámetros del runbook.

En el ejemplo siguiente se usan dos parámetros, **FeatureName** e **IsPresent**, para determinar los valores de las propiedades de la configuración del nodo **ParametersExample.sample**, que se ha generado durante la compilación.

```powershell
Configuration ParametersExample
{
    param(
        [Parameter(Mandatory=$true)]
        [string] $FeatureName,

        [Parameter(Mandatory=$true)]
        [boolean] $IsPresent
    )

    $EnsureString = 'Present'
    if($IsPresent -eq $false)
    {
        $EnsureString = 'Absent'
    }

    Node 'sample'
    {
        WindowsFeature ($FeatureName + 'Feature')
        {
            Ensure = $EnsureString
            Name   = $FeatureName
        }
    }
}
```

Puede compilar configuraciones DSC que usan parámetros básicos en el portal de DSC de Azure Automation State Configuration o Azure PowerShell:

#### <a name="portal"></a>Portal

En el portal, puede especificar valores de parámetro después de hacer clic **Compilar**.

![Parámetros compilación de configuración](./media/automation-dsc-compile/DSC_compiling_1.png)

#### <a name="azure-powershell"></a>Azure PowerShell

PowerShell requiere parámetros de un [hashtable](/powershell/module/microsoft.powershell.core/about/about_hash_tables) donde la clave coincide con el nombre del parámetro y el valor es igual al valor de parámetro.

```powershell
$Parameters = @{
    'FeatureName' = 'Web-Server'
    'IsPresent' = $False
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ParametersExample' -Parameters $Parameters
```

Para obtener información acerca de cómo pasar PSCredentials como parámetros, consulte más abajo la sección [Recursos de credenciales](#credential-assets).

### <a name="compile-configurations-containing-composite-resources-in-azure-automation"></a>Compilación de configuraciones que contienen recursos compuestos en Azure Automation

La característica **Recursos compuestos** le permite utilizar las configuraciones de DSC como recursos anidados dentro de una configuración. Esto le permite aplicar varias configuraciones a un solo recurso. Consulte [Recursos compuestos: uso de una configuración de DSC como un recurso](/powershell/scripting/dsc/resources/authoringresourcecomposite) para más información acerca de los recursos compuestos.

> [!NOTE]
> Para que las configuraciones que contienen recursos compuestos se compilen correctamente, antes debe asegurarse de que los recursos de DSC en los que se basa el compuesto se importan en Azure Automation.

La incorporación de un recurso compuesto de DSC no es diferente de la incorporación de un módulo de PowerShell a Azure Automation. El proceso se documenta en [Administración de módulos en Azure Automation](/azure/automation/shared-resources/modules).

### <a name="manage-configurationdata-when-compiling-configurations-in-azure-automation"></a>Administración de ConfigurationData al compilar configuraciones en Azure Automation

La característica **ConfigurationData** le permite separar las opciones estructurales de cualquier configuración específica del entorno al usar DSC de PowerShell. Consulte [Diferenciación de "Qué" y "Dónde" en DSC de PowerShell](https://blogs.msdn.com/b/powershell/archive/2014/01/09/continuous-deployment-using-dsc-with-minimal-change.aspx) para más información sobre ConfigurationData.

> [!NOTE]
> Puede usar ConfigurationData cuando compile en State Configuration de Azure Automation con Azure PowerShell, pero no en Azure Portal.

En el siguiente ejemplo, la configuración de DSC usa ConfigurationData mediante las palabras clave $ConfigurationData y $AllNodes. En este ejemplo, también necesitará el [módulo xWebAdministration](https://www.powershellgallery.com/packages/xWebAdministration/):

```powershell
Configuration ConfigurationDataSample
{
    Import-DscResource -ModuleName xWebAdministration -Name MSFT_xWebsite

    Write-Verbose $ConfigurationData.NonNodeData.SomeMessage

    Node $AllNodes.Where{$_.Role -eq 'WebServer'}.NodeName
    {
        xWebsite Site
        {
            Name         = $Node.SiteName
            PhysicalPath = $Node.SiteContents
            Ensure       = 'Present'
        }
    }
}
```

Puede compilar la configuración de DSC anterior con Windows PowerShell. El siguiente script agrega dos configuraciones de nodo al servicio de extracción de State Configuration de Azure Automation: ConfigurationDataSample.MyVM1 y ConfigurationDataSample.MyVM3.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = 'MyVM1'
            Role = 'WebServer'
        },
        @{
            NodeName = 'MyVM2'
            Role = 'SQLServer'
        },
        @{
            NodeName = 'MyVM3'
            Role = 'WebServer'
        }
    )

    NonNodeData = @{
        SomeMessage = 'I love Azure Automation State Configuration and DSC!'
    }
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'ConfigurationDataSample' -ConfigurationData $ConfigData
```

### <a name="work-with-assets-in-azure-automation-during-compilation"></a>Trabajo con recursos en Azure Automation durante la compilación

Las referencias de recursos son las mismas en State Configuration de Azure Automation y en los runbooks. Para obtener más información, vea lo siguiente:

- [Certificados](automation-certificates.md)
- [Conexiones](automation-connections.md)
- [Credenciales](automation-credentials.md)
- [Variables](automation-variables.md)

#### <a name="credential-assets"></a>Recursos de credenciales

Las configuraciones de DSC en Azure Automation pueden hacer referencia a los recursos de credenciales de Automation mediante el uso del cmdlet **Get-AutomationPSCredential**. Si una configuración tiene un parámetro con el tipo PSCredential, puede usar **Get-AutomationPSCredential** pasando el nombre de cadena de un recurso de credencial de Azure Automation para recuperar la credencial. Luego puede usar ese objeto para el parámetro que requiere el objeto PSCredential. En segundo plano, se recuperará el recurso de credenciales de Azure Automation con ese nombre y se pasará a la configuración. En el ejemplo siguiente se muestra esta operación en acción.

Mantener las credenciales seguras en una configuración de nodo exige el cifrado de las credenciales en el archivo MOF de la configuración de nodo. Debe informar a DSC de PowerShell específicamente de que tiene permiso para mostrar en la salida las credenciales en texto sin formato durante la generación del archivo MOF de configuración de nodo. DSC de PowerShell no sabe que Azure Automation cifra todo el archivo MOF después de su generación mediante un trabajo de compilación.

Para conceder el permiso a DSC de PowerShell para mostrar en la salida las credenciales en texto sin formato en los archivos MOF de configuración de nodo generados con datos de configuración, pase `PSDscAllowPlainTextPassword = $true`. Puede pasar esta información mediante ConfigurationData para cada nombre de bloque de nodo que aparezca en la configuración de DSC y use credenciales.

En el ejemplo siguiente se muestra una configuración de DSC que usa un recurso de la credencial de Automation.

```powershell
Configuration CredentialSample
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration
    $Cred = Get-AutomationPSCredential 'SomeCredentialAsset'

    Node $AllNodes.NodeName
    {
        File ExampleFile
        {
            SourcePath      = '\\Server\share\path\file.ext'
            DestinationPath = 'C:\destinationPath'
            Credential      = $Cred
        }
    }
}
```

Puede compilar la configuración de DSC anterior con PowerShell. 

El siguiente código de PowerShell agrega las configuraciones de dos nodos al servidor de extracción de State Configuration de Azure Automation: CredentialSample.MyVM1 y CredentialSample.MyVM2.

```powershell
$ConfigData = @{
    AllNodes = @(
        @{
            NodeName = '*'
            PSDscAllowPlainTextPassword = $True
        },
        @{
            NodeName = 'MyVM1'
        },
        @{
            NodeName = 'MyVM2'
        }
    )
}

Start-AzAutomationDscCompilationJob -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'MyAutomationAccount' -ConfigurationName 'CredentialSample' -ConfigurationData $ConfigData
```

>[!NOTE]
>Cuando se completa la compilación, puede recibir un error que indica: **The "Microsoft.PowerShell.Management" module was not imported because the "Microsoft.PowerShell.Management" snap-in was already imported.** ("El módulo "Microsoft.PowerShell.Management" no se importó porque el complemento "Microsoft.PowerShell.Management" ya se había importado"). Puede omitir esta advertencia sin problemas.

## <a name="compiling-a-dsc-configuration-in-windows-powershell"></a>Compilación de una configuración de DSC en Windows PowerShell

El proceso de compilación de las configuraciones de DSC en Windows PowerShell se incluye en la documentación de DSC de PowerShell, en [Escritura, compilación y aplicación de una configuración](/powershell/scripting/dsc/configurations/write-compile-apply-configuration#compile-the-configuration).
Puede ejecutar el proceso desde una estación de trabajo de desarrollador o dentro de un servicio de compilación como [Azure DevOps](https://dev.azure.com).
Después, puede importar los archivos MOF para las configuraciones de nodo resultantes directamente en el servicio Azure State Configuration. 

>[!NOTE]
>Un archivo de configuración de nodo debe ocupar más de 1 MB para poderlo importar en Azure Automation.

También puede importar las configuraciones de nodo (MOF) que se hayan compilado fuera de Azure. Este enfoque tiene múltiples ventajas, como el rendimiento y la confiabilidad.

La compilación en Windows PowerShell ofrece la opción de firmar el contenido de la configuración, en la que el agente de DSC comprueba localmente una configuración de nodo firmado en un nodo administrado. La comprobación garantiza que la configuración que se aplica al nodo proviene de un origen autorizado. Para más información sobre la firma de configuraciones de nodo, consulte [Mejoras en WMF 5.1: firma de la configuración y del módulo](/powershell/scripting/wmf/whats-new/dsc-improvements#dsc-module-and-configuration-signing-validations).

### <a name="import-a-node-configuration-in-the-azure-portal"></a>Importación de una configuración de nodo en Azure Portal

1. En la cuenta de Automation, haga clic en **State Configuration (DSC)** en **Administración de configuración**.
1. En la página State Configuration (DSC), haga clic en la pestaña **Configuraciones** y, después, haga clic en **+ Agregar**.
1. En la página Importar, haga clic en el icono de carpeta situado junto al cuadro de texto **Archivo de configuración de nodo** para buscar un archivo de configuración de nodo (MOF) en el equipo local.

   ![Buscar archivo local](./media/automation-dsc-compile/import-browse.png)

1. Escriba un nombre en el cuadro de texto **Nombre de la configuración**. Este nombre debe coincidir con el de la configuración desde la que se compiló la configuración del nodo.
1. Haga clic en **OK**.

### <a name="import-a-node-configuration-with-azure-powershell"></a>Importación de una configuración de nodo con Azure PowerShell

Puede usar el cmdlet [Import-AzAutomationDscNodeConfiguration](/powershell/module/az.automation/import-azautomationdscnodeconfiguration) para importar una configuración de nodo en la cuenta de Automation.

```powershell
Import-AzAutomationDscNodeConfiguration -AutomationAccountName 'MyAutomationAccount' -ResourceGroupName 'MyResourceGroup' -ConfigurationName 'MyNodeConfiguration' -Path 'C:\MyConfigurations\TestVM1.mof'
```

## <a name="next-steps"></a>Pasos siguientes

- Para empezar a trabajar, consulte [Introducción a State Configuration de Azure Automation](automation-dsc-getting-started.md).
- Para obtener información acerca de la compilación de configuraciones de DSC para poder asignarlas a los nodos de destino, consulte [Compilación de configuraciones en State Configuration de Azure Automation](automation-dsc-compile.md).
- Para la referencia de cmdlets de PowerShell, consulte [Cmdlets de State Configuration de Azure Automation](/powershell/module/az.automation).
- Para obtener información de precios, consulte [Precios de State Configuration de Azure Automation](https://azure.microsoft.com/pricing/details/automation/).
- Para ver un ejemplo del uso de State Configuration de Azure Automation en una canalización de implementación continua, consulte [Implementación continua mediante State Configuration de Azure Automation y Chocolatey](automation-dsc-cd-chocolatey.md).
