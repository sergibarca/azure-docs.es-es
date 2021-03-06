---
title: Uso de Azure Functions Core Tools
description: Aprenda a codificar y probar funciones de Azure en el símbolo del sistema o terminal en la máquina local antes de ejecutarlas en Azure Functions.
ms.assetid: 242736be-ec66-4114-924b-31795fd18884
ms.topic: conceptual
ms.date: 03/13/2019
ms.custom: 80e4ff38-5174-43
ms.openlocfilehash: dd36895a34b36bbdf8e796cf629ab031613663cd
ms.sourcegitcommit: 2823677304c10763c21bcb047df90f86339e476a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/14/2020
ms.locfileid: "77208891"
---
# <a name="work-with-azure-functions-core-tools"></a>Uso de Azure Functions Core Tools

Azure Functions Core Tools le permite desarrollar y probar funciones en el equipo local desde el símbolo del sistema o terminal. Las funciones locales pueden conectarse a servicios de Azure en directo, y puede depurar sus funciones en el equipo local con el tiempo de ejecución de Functions completo. Incluso puede implementar una aplicación de función en su suscripción a Azure.

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

Para desarrollar funciones en el equipo local y publicarlas en Azure utilizando Core Tools siga estos pasos básicos:

> [!div class="checklist"]
> * [Instale Core Tools y sus dependencias.](#v2)
> * [Cree un proyecto de aplicación de funciones a partir de una plantilla específica del idioma.](#create-a-local-functions-project)
> * [Registre extensiones de desencadenadores y enlace.](#register-extensions)
> * [Defina el almacenamiento y otras conexiones.](#local-settings-file)
> * [Cree una función desde un desencadenador y una plantilla específica del idioma.](#create-func)
> * [Ejecute la función de forma local.](#start)
> * [Publique el proyecto en Azure.](#publish)

## <a name="core-tools-versions"></a>Versiones de Core Tools

Hay tres versiones de Azure Functions Core Tools. La versión que use depende del entorno de desarrollo local, la [elección del lenguaje](supported-languages.md) y el nivel de compatibilidad necesario:

+ **Versión 1.x**: es compatible con la versión 1.x del entorno en tiempo de ejecución de Azure Functions. Esta versión de las herramientas solo se admite en equipos con Windows y se instala desde un [paquete npm](https://www.npmjs.com/package/azure-functions-core-tools).

+ [**Versión 2.x o 3.x**](#v2): es compatible con la [versión 2.x o 3.x del entorno de ejecución de Azure Functions](functions-versions.md). Estas versiones son compatibles con [Windows](/azure/azure-functions/functions-run-local?tabs=windows#v2), [macOS](/azure/azure-functions/functions-run-local?tabs=macos#v2) y [Linux](/azure/azure-functions/functions-run-local?tabs=linux#v2). Además, usan administradores de paquetes o específicos de la plataforma o npm para la instalación.

A menos que se indique lo contrario, los ejemplos de este artículo son para la versión 3.x.

## <a name="install-the-azure-functions-core-tools"></a>Instalación de Azure Functions Core Tools

[Azure Functions Core Tools] incluye una versión del mismo tiempo de ejecución de Azure Functions que puede ejecutar en el equipo de desarrollo local. También proporciona comandos para crear funciones, conectarse a Azure e implementar proyectos de funciones.

### <a name="v2"></a>Versión 2.x y 3.x

La versión 2.x o 3.x de las herramientas usa el entorno en tiempo de ejecución de Azure Functions que se basa en .NET Core. Esta versión se admite en todas las plataformas que admiten .NET Core, incluidas [Windows](/azure/azure-functions/functions-run-local?tabs=windows#v2), [macOS](/azure/azure-functions/functions-run-local?tabs=macos#v2) y [Linux](/azure/azure-functions/functions-run-local?tabs=linux#v2). 

> [!IMPORTANT]
> Puede omitir el requisito de instalar el SDK de .NET Core con los [conjuntos de extensión].

# <a name="windows"></a>[Windows](#tab/windows)

Los pasos siguientes utilizan npm para instalar Core Tools en Windows. También puede usar [Chocolatey](https://chocolatey.org/). Para más información, consulte el [archivo Léame de Core Tools](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#windows).

1. Instale [Node.js], que incluye npm.
    - Para la versión 2.x de las herramientas, solo se admite Node.js 8.5 y versiones posteriores.
    - Para la versión 3.x de las herramientas, solo se admite Node.js 10 y versiones posteriores.

1. Instale el paquete de Core Tools:

    ##### <a name="v2x"></a>Versión 2.x

    ```bash
    npm install -g azure-functions-core-tools
    ```

    ##### <a name="v3x"></a>Versión 3.x

    ```bash
    npm install -g azure-functions-core-tools@3
    ```

   NPM puede tardar unos minutos en descargar e instalar el paquete de Core Tools.

1. Si no planea usar los [conjuntos de extensión], instale el [SDK de .NET Core 2.x para Windows](https://www.microsoft.com/net/download/windows).

# <a name="macos"></a>[macOS](#tab/macos)

Los pasos siguientes utilizan Homebrew para instalar Core Tools en macOS.

1. Instale [Homebrew](https://brew.sh/), si aún no está instalado.

1. Instale el paquete de Core Tools:

    ##### <a name="v2x"></a>Versión 2.x

    ```bash
    brew tap azure/functions
    brew install azure-functions-core-tools
    ```

    ##### <a name="v3x"></a>Versión 3.x

    ```bash
    brew tap azure/functions
    brew install azure-functions-core-tools@3
    # if upgrading on a machine that has 2.x installed
    brew link --overwrite azure-functions-core-tools@3
    ```

# <a name="linux"></a>[Linux](#tab/linux)

Los siguientes pasos usan [APT](https://wiki.debian.org/Apt) para instalar Core Tools en la distribución de Ubuntu/Debian Linux. Para otras distribuciones de Linux, consulte el [archivo Léame de Core Tools](https://github.com/Azure/azure-functions-core-tools/blob/master/README.md#linux).

1. Instale la clave GPG del repositorio de paquetes de Microsoft para validar la integridad del paquete:

    ```bash
    curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
    sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
    ```

1. Configure la lista de origen de desarrollo de .NET antes de actualizar APT.

   Si desea configurar la lista de origen de APT para Ubuntu, ejecute este comando:

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
    ```

   Si desea configurar la lista de origen de APT para Debian, ejecute este comando:

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/debian/$(lsb_release -rs | cut -d'.' -f 1)/prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
    ```

1. Compruebe si el archivo `/etc/apt/sources.list.d/dotnetdev.list` contiene una de las cadenas que se indican a continuación con la versión de Linux apropiada:

    | Distribución de Linux | Versión |
    | --------------- | ----------- |
    | Debian 9 | `stretch` |
    | Debian 8 | `jessie` |
    | Ubuntu 18.10    | `cosmic`    |
    | Ubuntu 18.04    | `bionic`    |
    | Ubuntu 17.04    | `zesty`     |
    | Ubuntu 16.04/Linux Mint 18    | `xenial`  |

1. Inicie la actualización del origen de APT:

    ```bash
    sudo apt-get update
    ```

1. Instale el paquete de Core Tools:

    ```bash
    sudo apt-get install azure-functions-core-tools
    ```

1. Si no planea usar los [conjuntos de extensión], instale el [SDK de .NET Core 2.x para Linux](https://www.microsoft.com/net/download/linux).

---

## <a name="create-a-local-functions-project"></a>Creación de un proyecto local de Functions

Un directorio de proyecto de funciones contiene los archivos [host.json](functions-host-json.md) y [local.settings.json](#local-settings-file), junto con las subcarpetas que contienen el código de funciones individuales. Este directorio es el equivalente a una aplicación de función en Azure. Para obtener más información sobre la estructura de carpetas de Functions, vea la [Guía para desarrolladores de Azure Functions](functions-reference.md#folder-structure).

Para la versión 2.x deberá seleccionar un lenguaje predeterminado para el proyecto cuando lo inicialice. En la versión 2. x, todas las funciones agregadas usan plantillas de lenguaje predeterminado. En la versión 1.x, debe especificar el lenguaje cada vez que crea una función.

En la ventana de terminal o desde un símbolo del sistema, ejecute el siguiente comando para crear el proyecto y el repositorio de Git local:

```bash
func init MyFunctionProj
```

Al especificar un nombre de proyecto, se crea una carpeta con dicho nombre y posteriormente se inicializa. En caso contrario, se inicializa la carpeta actual.  
En la versión 2.x, cuando ejecute el comando, debe elegir un tiempo de ejecución para el proyecto. 

```output
Select a worker runtime:
dotnet
node
python 
powershell
```

Use las flechas arriba/abajo para elegir un lenguaje, a continuación, presione ENTRAR. Si tiene previsto desarrollar funciones de JavaScript o de TypeScript, elija **nodo** y seleccione el lenguaje. TypeScript tiene [algunos requisitos adicionales](functions-reference-node.md#typescript). 

La salida tendrá un aspecto similar al siguiente ejemplo de un proyecto de JavaScript:

```output
Select a worker runtime: node
Writing .gitignore
Writing host.json
Writing local.settings.json
Writing C:\myfunctions\myMyFunctionProj\.vscode\extensions.json
Initialized empty Git repository in C:/myfunctions/myMyFunctionProj/.git/
```

`func init` admite las siguientes opciones, que corresponden solo a la versión 2.x, a menos que se indique lo contrario:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--csharp`**<br/> **`--dotnet`** | Inicializa un [proyecto de biblioteca de clases de C# (.cs)](functions-dotnet-class-library.md). |
| **`--csx`** | Inicializa un [proyecto de script de C# (.csx)](functions-reference-csharp.md). Debe especificar `--csx` en los siguientes comandos. |
| **`--docker`** | Crea un archivo Dockerfile para un contenedor con una imagen base en función del `--worker-runtime` elegido. Use esta opción cuando vaya a publicar en un contenedor Linux personalizado. |
| **`--docker-only`** |  Agrega un Dockerfile a un proyecto existente. Solicita el entorno de ejecución de trabajo si no se especifica o se establece en local.settings.json. Use esta opción cuando vaya a publicar un proyecto existente en un contenedor de Linux personalizado. |
| **`--force`** | Inicializa el proyecto incluso cuando hay archivos existentes en el proyecto. Este valor sobrescribe los archivos existentes con el mismo nombre. Los otros archivos de la carpeta del proyecto no se ven afectados. |
| **`--java`**  | Inicializa un [proyecto de Java](functions-reference-java.md). |
| **`--javascript`**<br/>**`--node`**  | Inicializa un [proyecto de JavaScript](functions-reference-node.md). |
| **`--no-source-control`**<br/>**`-n`** | Impide la creación de forma predeterminada de un repositorio de Git en la versión 1.x. En la versión 2.x, el repositorio de git no se crea de forma predeterminada. |
| **`--powershell`**  | Inicializa un [proyecto de PowerShell](functions-reference-powershell.md). |
| **`--python`**  | Inicializa un [proyecto de Python](functions-reference-python.md). |
| **`--source-control`** | Controla si se crea un repositorio de git. De forma predeterminada, no se crea un repositorio. Cuando es `true`, se crea un repositorio. |
| **`--typescript`**  | Inicializa un [proyecto de TypeScript](functions-reference-node.md#typescript). |
| **`--worker-runtime`** | Establece el entorno de ejecución del lenguaje del proyecto. Los valores admitidos son: `csharp`, `dotnet`, `java`, `javascript`,`node` (JavaScript), `powershell`, `python` y `typescript`. Si no se establece, deberá elegir el entorno de ejecución durante la inicialización. |

> [!IMPORTANT]
> De manera predeterminada, la versión 2.x de Core Tools crea proyectos de aplicación de función para el runtime de .NET como [proyectos de clase de C#](functions-dotnet-class-library.md) (.csproj). Estos proyectos de C#, que se pueden usar con Visual Studio o con Visual Studio Code, se compilan durante las pruebas y al publicar en Azure. Si en su lugar desea crear y trabajar con los mismos archivos de script de C# (.csx) creados en la versión 1.x y en el portal, debe incluir el parámetro `--csx` cuando cree e implemente las funciones.

[!INCLUDE [functions-core-tools-install-extension](../../includes/functions-core-tools-install-extension.md)]

[!INCLUDE [functions-local-settings-file](../../includes/functions-local-settings-file.md)]

De manera predeterminada, estas opciones de configuración no se migran automáticamente cuando el proyecto se publica en Azure. Use el conmutador `--publish-local-settings`[al publicarlo](#publish) para asegurarse de que la configuración se agregue a la aplicación de función en Azure. Tenga en cuenta que los valores de **ConnectionStrings** nunca se publican.

Esta configuración de la aplicación de función también se puede leer en el código como variables de entorno. Para más información, consulte la sección Variables de entorno de estos temas de referencia específicos del lenguaje:

* [C# precompilado](functions-dotnet-class-library.md#environment-variables)
* [Script de C# (.csx)](functions-reference-csharp.md#environment-variables)
* [Java](functions-reference-java.md#environment-variables)
* [JavaScript](functions-reference-node.md#environment-variables)

Cuando no se establece ninguna cadena de conexión de almacenamiento válida para [`AzureWebJobsStorage`] y no se usa el emulador, se muestra el siguiente mensaje de error:

> Missing value for AzureWebJobsStorage in local.settings.json. This is required for all triggers other than HTTP. Puede ejecutar "FUNC Azure functionapp fetch-App-Settings \<functionAppName\>" o especificar una cadena de conexión en local. Settings. JSON.

### <a name="get-your-storage-connection-strings"></a>Obtención de las cadenas de conexión de almacenamiento

Incluso cuando se usa el Emulador de Microsoft Azure Storage para tareas de desarrollo, recomendamos probar con una conexión de almacenamiento real. Suponiendo que ya [creó una cuenta de almacenamiento](../storage/common/storage-create-storage-account.md), puede obtener una cadena de conexión de almacenamiento válida de una de las maneras siguientes:

- En [Azure Portal], busque y seleccione **Cuentas de almacenamiento**. 
  ![Selección de cuentas de almacenamiento desde Azure Portal](./media/functions-run-local/select-storage-accounts.png)
  
  Seleccione la cuenta de almacenamiento, elija **Claves de acceso** en **Configuración** y, a continuación, copie uno de los valores de **Cadena de conexión**.
  ![Copia de una cadena de conexión desde Azure Portal](./media/functions-run-local/copy-storage-connection-portal.png)

- Use [Explorador de Azure Storage](https://storageexplorer.com/) para conectarse a su cuenta de almacenamiento de Azure. En el **Explorador**, expanda su suscripción, expanda **Cuentas de almacenamiento**, seleccione la cuenta de almacenamiento y copie la cadena de conexión principal o secundaria.

  ![Copia de la cadena de conexión desde el Explorador de Azure Storage](./media/functions-run-local/storage-explorer.png)

+ Utilice Core Tools para descargar la cadena de conexión de Azure con uno de los siguientes comandos:

  + Descargue toda la configuración de una aplicación de función existente:

    ```bash
    func azure functionapp fetch-app-settings <FunctionAppName>
    ```
  + Obtenga la cadena de conexión de una cuenta de almacenamiento concreta:

    ```bash
    func azure storage fetch-connection-string <StorageAccountName>
    ```

    Si aún no ha iniciado sesión en Azure, se le pedirá que lo haga.

## <a name="create-func"></a>Creación de una función

Para crear una función, ejecute el siguiente comando:

```bash
func new
```

En la versión 2.x, al ejecutar `func new` se le pedirá que elija una plantilla en el lenguaje predeterminado de la aplicación de función y, después, también se le pedirá que elija un nombre para la función. En la versión 1.x, también se le pedirá que elija el lenguaje.

```output
Select a language: Select a template:
Blob trigger
Cosmos DB trigger
Event Grid trigger
HTTP trigger
Queue trigger
SendGrid
Service Bus Queue trigger
Service Bus Topic trigger
Timer trigger
```

El código de la función se genera en una subcarpeta con el nombre proporcionado de la función, como se aprecia en la siguiente salida de desencadenador de cola:

```output
Select a language: Select a template: Queue trigger
Function name: [QueueTriggerJS] MyQueueTrigger
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\index.js
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\readme.md
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\sample.dat
Writing C:\myfunctions\myMyFunctionProj\MyQueueTrigger\function.json
```

También puede especificar estas opciones en el comando con los argumentos siguientes:

| Argumento     | Descripción                            |
| ------------------------------------------ | -------------------------------------- |
| **`--csx`** | (Versión 2.x) Genera las mismas plantillas de script de C# (.csx) que se usan en la versión 1.x y en el portal. |
| **`--language`** , **`-l`**| Lenguaje de programación de la plantilla, como C#, F# o JavaScript. Esta opción es obligatoria en la versión 1.x. En la versión 2.x, no utilice esta opción o elija un lenguaje que coincida con el entorno de ejecución del trabajo. |
| **`--name`** , **`-n`** | Nombre de función. |
| **`--template`** , **`-t`** | Use el comando `func templates list` para ver la lista completa de plantillas disponibles para cada lenguaje compatible.   |

Por ejemplo, para crear un desencadenador HTTP de JavaScript en un único comando, ejecute:

```bash
func new --template "Http Trigger" --name MyHttpTrigger
```

Para crear una función desencadenada por la cola en un único comando, ejecute:

```bash
func new --template "Queue Trigger" --name QueueTriggerJS
```

## <a name="start"></a>Ejecución local de funciones

Para ejecutar un proyecto de Functions, ejecute el host de Functions. El host habilita desencadenadores para todas las funciones del proyecto. 

### <a name="version-2x"></a>Versión 2.x

En la versión 2.x del runtime, el comando de inicio varía según el lenguaje del proyecto.

#### <a name="c"></a>C\#

```command
func start --build
```

#### <a name="javascript"></a>JavaScript

```command
func start
```

#### <a name="typescript"></a>TypeScript

```command
npm install
npm start     
```

### <a name="version-1x"></a>Versión 1.x

La versión 1.x del runtime de Functions requiere el comando `host`, como en el ejemplo siguiente:

```command
func host start
```

`func start` admite las siguientes opciones:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--no-build`** | No compile del proyecto actual antes de su ejecución. Solo para proyectos de dotnet. El valor predeterminado se establece en false. Solo versión 2.x. |
| **`--cert`** | La ruta de acceso a un archivo .pfx que contiene una clave privada. Solo se usa con `--useHttps`. Solo versión 2.x. |
| **`--cors-credentials`** | Permitir solicitudes autenticadas de varios orígenes (es decir, cookies y el encabezado de autenticación) Solo versión 2.x. |
| **`--cors`** | Lista separada por comas de orígenes CORS, sin espacios en blanco. |
| **`--language-worker`** | Argumentos para configurar el trabajo del lenguaje. Por ejemplo, puede habilitar la depuración para el trabajo de lenguaje proporcionando el [puerto de depuración y otros argumentos necesarios](https://github.com/Azure/azure-functions-core-tools/wiki/Enable-Debugging-for-language-workers). Solo versión 2.x. |
| **`--nodeDebugPort`** , **`-n`** | Puerto del depurador Node.js que se va a usar. Valor predeterminado: un valor de launch.json o 5858. Solo versión 1.x. |
| **`--password`** | La contraseña o un archivo que contenga la contraseña de un archivo. pfx. Solo se usa con `--cert`. Solo versión 2.x. |
| **`--port`** , **`-p`** | Puerto local en el que se escucha. Valor predeterminado: 7071. |
| **`--pause-on-error`** | Se pone en pausa en espera de entrada adicional antes de salir del proceso. Se utiliza solo cuando se inicia Core Tools desde un entorno de desarrollo integrado (IDE).|
| **`--script-root`** , **`--prefix`** | Se usa para especificar la ruta de acceso a la raíz de la aplicación de función que se va a ejecutar o implementar. Esto se usa para los proyectos compilados que generan archivos de proyecto en una subcarpeta. Por ejemplo, cuando se compila un proyecto de biblioteca de clases de C#, los archivos host.json, local.settings.json y function.json se generan en una subcarpeta *raíz* con una ruta de acceso similar a `MyProject/bin/Debug/netstandard2.0`. En este caso, establezca el prefijo como `--script-root MyProject/bin/Debug/netstandard2.0`. Esta es la raíz de la aplicación de función cuando se ejecuta en Azure. |
| **`--timeout`** , **`-t`** | Tiempo de espera en segundos para que se inicie el host de Functions. Valor predeterminado: 20 segundos.|
| **`--useHttps`** | Enlace con `https://localhost:{port}` en lugar de con `http://localhost:{port}`. De forma predeterminada, esta opción crea un certificado de confianza en el equipo.|

Cuando se inicia el host de Functions, devuelve la dirección URL de las funciones desencadenadas por HTTP:

```output
Found the following functions:
Host.Functions.MyHttpTrigger

Job host started
Http Function MyHttpTrigger: http://localhost:7071/api/MyHttpTrigger
```

>[!IMPORTANT]
>Cuando se ejecuta localmente, no se aplica la autorización para puntos de conexión HTTP. Esto significa que todas las solicitudes HTTP locales se tratan como `authLevel = "anonymous"`. Para obtener más información, consulte el artículo sobre [enlaces HTTP](functions-bindings-http-webhook-trigger.md#authorization-keys).

### <a name="passing-test-data-to-a-function"></a>Paso de datos de prueba a una función

Para probar sus funciones localmente, [inicie el host de Functions](#start) y llame a puntos de conexión del servidor local mediante solicitudes HTTP. El punto de conexión al que llama depende del tipo de función.

>[!NOTE]
> En los ejemplos de este tema se usa la herramienta cURL para enviar solicitudes HTTP desde el terminal o un símbolo del sistema. Puede usar una herramienta de su elección para enviar solicitudes HTTP al servidor local. La herramienta cURL está disponible de forma predeterminada en los sistemas basados en Linux y en la compilación 17063 de Windows 10, y en las posteriores. En las versiones anteriores de Windows, primero debe descargar e instalar la [herramienta cURL](https://curl.haxx.se/).

Para obtener información más general sobre cómo probar funciones, consulte [Estrategias para probar el código en Azure Functions](functions-test-a-function.md).

#### <a name="http-and-webhook-triggered-functions"></a>Funciones desencadenadas por HTTP y webhook

Llama al siguiente punto de conexión para ejecutar de forma local funciones desencadenadas por HTTP y webhook:

    http://localhost:{port}/api/{function_name}

Asegúrese de usar el mismo nombre del servidor y puerto en el que escucha el host de Functions. Puede ver esto en la salida generada al iniciar el host de Functions. Puede llamar a esta dirección URL mediante cualquier método HTTP admitido por el desencadenador.

El siguiente comando cURL desencadena la función de inicio rápido `MyHttpTrigger` desde una solicitud GET con el parámetro _name_ transferido en la cadena de consulta.

```bash
curl --get http://localhost:7071/api/MyHttpTrigger?name=Azure%20Rocks
```

En el siguiente ejemplo está la misma función a la que se llama desde una solicitud POST que transfiere _name_ en el cuerpo de la solicitud:

```bash
curl --request POST http://localhost:7071/api/MyHttpTrigger --data '{"name":"Azure Rocks"}'
```

Puede realizar solicitudes GET desde un explorador que transfiere datos en la cadena de consulta. Para todos los demás métodos HTTP, debe usar cURL, Fiddler, Postman o una herramienta de pruebas HTTP similar.

#### <a name="non-http-triggered-functions"></a>Funciones no desencadenadas por HTTP

En el caso de todos los tipos de funciones distintas de los desencadenadores HTTP y HTTP webhooks, puede probar sus funciones localmente llamando a un punto de conexión de administración. Al llamar a este punto de conexión con una solicitud HTTP POST en el servidor local se desencadena esta función. Opcionalmente, puede transferir los datos de prueba a la ejecución en el cuerpo de la solicitud POST. Esta funcionalidad es similar a la pestaña **Prueba** de Azure Portal.

Se llama al siguiente punto de conexión de administrador para desencadenar funciones ajenas a HTTP:

    http://localhost:{port}/admin/functions/{function_name}

Para transferir datos de prueba al punto de conexión de administrador de una función, debe proporcionar los datos en el cuerpo de un mensaje de solicitud POST. Es necesario que el cuerpo del mensaje tenga el siguiente formato JSON:

```JSON
{
    "input": "<trigger_input>"
}
```

El valor `<trigger_input>` contiene datos en un formato esperado por la función. El siguiente ejemplo de cURL es una solicitud POST dirigida a una función `QueueTriggerJS`. En este caso, la entrada es una cadena que equivale al mensaje que se espera encontrar en la cola.

```bash
curl --request POST -H "Content-Type:application/json" --data '{"input":"sample queue data"}' http://localhost:7071/admin/functions/QueueTriggerJS
```

#### <a name="using-the-func-run-command-in-version-1x"></a>Uso del comando `func run` en la versión 1.x

>[!IMPORTANT]
> El comando `func run` no se admite en la versión 2.x de las herramientas. Para obtener más información, consulte el tema [How to target Azure Functions runtime versions](set-runtime-version.md) (Cómo seleccionar un destino para versiones en tiempo de ejecución de Azure Functions).

También puede invocar una función directamente con `func run <FunctionName>` y proporcionar datos de entrada para la función. Este comando es similar a la ejecución de una función con la pestaña **Prueba** de Azure Portal.

`func run` admite las siguientes opciones:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--content`** , **`-c`** | Contenido alineado. |
| **`--debug`** , **`-d`** | Se asocia un depurador al proceso de host antes de ejecutar la función.|
| **`--timeout`** , **`-t`** | Tiempo de espera (en segundos) hasta que el host local de Functions está listo.|
| **`--file`** , **`-f`** | Nombre del archivo que se usa como contenido.|
| **`--no-interactive`** | No pide entrada. Resulta útil en escenarios de automatización.|

Por ejemplo, para llamar a una función desencadenada por HTTP y pasar cuerpo del contenido, ejecute el siguiente comando:

```bash
func run MyHttpTrigger -c '{\"name\": \"Azure\"}'
```

## <a name="publish"></a>Publicación en Azure

Azure Functions Core Tools admite dos tipos de implementación: la opción de implementar archivos del proyecto de funciones directamente en su aplicación de funciones a través de la [implementación de archivos zip](functions-deployment-technologies.md#zip-deploy) y la [implementación de un contenedor de Docker personalizado](functions-deployment-technologies.md#docker-container). Tiene que tener [creada una aplicación de funciones en su suscripción de Azure](functions-cli-samples.md#create) para implementar su código. Se deben compilar los proyectos que lo requieran para poder implementar los archivos binarios.

Una carpeta de proyecto puede contener archivos y directorios específicos del idioma que no deben publicarse. Los elementos excluidos se enumeran en un archivo .funcignore en la carpeta raíz del proyecto.     

### <a name="project-file-deployment"></a>Implementación (archivos del proyecto)

Para publicar su código local en una aplicación de funciones en Azure, use el comando `publish`:

```bash
func azure functionapp publish <FunctionAppName>
```

Este comando se publica en una aplicación de función existente en Azure. Obtendrá un error si intenta publicarla en un `<FunctionAppName>` que no exista en su suscripción. Para obtener información sobre cómo crear una aplicación de función desde el símbolo del sistema o la ventana de Terminal mediante la CLI de Azure, consulte [Creación de una instancia de Function App para la ejecución sin servidor](./scripts/functions-cli-create-serverless.md). De manera predeterminada, este comando usa la [compilación remota](functions-deployment-technologies.md#remote-build) e implementa la aplicación para [ejecutarla desde el paquete de implementación](run-functions-from-deployment-package.md). Para deshabilitar este modo de implementación recomendado, use la opción `--nozip`.

>[!IMPORTANT]
> Cuando se crea una aplicación de función en Azure Portal, se usa la versión 2.x del entorno de ejecución de Functions de forma predeterminada. Para hacer que la aplicación de función utilice la versión 1.x del entorno de ejecución, siga las instrucciones de [Ejecución en la versión 1.x](functions-versions.md#creating-1x-apps).
> No se puede cambiar la versión del entorno de ejecución de una aplicación de función que tiene funciones existentes.

Las siguientes opciones de publicación se aplican a ambas versiones, 1.x y 2.x:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--publish-local-settings -i`** |  Se publica la configuración de local.settings.json en Azure, se pide que se sobrescriba si la configuración ya existe. Si usa el Emulador de Microsoft Azure Storage, cambie antes la configuración de la aplicación a una [conexión de almacenamiento real](#get-your-storage-connection-strings). |
| **`--overwrite-settings -y`** | Suprime el mensaje de sobrescritura de la configuración de la aplicación cuando se utiliza `--publish-local-settings -i`.|

Las siguientes opciones de publicación solo se admiten en la versión 2.x:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--publish-settings-only`** , **`-o`** |  Solo se publica la configuración y se omite el contenido. El valor predeterminado es Preguntar. |
|**`--list-ignored-files`** | Muestra una lista de archivos que se omiten durante la publicación, según el archivo .funcignore. |
| **`--list-included-files`** | Muestra una lista de archivos que se publican, según el archivo .funcignore. |
| **`--nozip`** | Desactiva el modo `Run-From-Package` predeterminado. |
| **`--build-native-deps`** | Omite la generación de la carpeta .wheels al publicar aplicaciones de función de Python. |
| **`--build`** , **`-b`** | Realiza la acción de compilación cuando se implementa en una aplicación de función de Linux. Acepta `remote` y `local`. |
| **`--additional-packages`** | Lista de paquetes para instalar al crear dependencias nativas. Por ejemplo: `python3-dev libevent-dev`. |
| **`--force`** | Omite la comprobación previa a la publicación en determinados escenarios. |
| **`--csx`** | Publica un proyecto de script de C# (.csx). |
| **`--no-build`** | No compila funciones de la biblioteca de clases .NET. |
| **`--dotnet-cli-params`** | Al publicar funciones de C# compiladas (.csproj), Core Tools llama a "dotnet build --output bin/publish". Todos los parámetros pasados se anexarán a la línea de comandos. |

### <a name="deployment-custom-container"></a>Implementación (contenedor personalizado)

Azure Functions le permite implementar el proyecto de funciones en un [contenedor de Docker personalizado](functions-deployment-technologies.md#docker-container). Para más información, consulte [Creación de una función en Linux con una imagen personalizada](functions-create-function-linux-custom-image.md). Los contenedores personalizados deben tener un archivo Dockerfile. Para crear una aplicación con un archivo Dockerfile, use la opción--dockerfile en `func init`.

```bash
func deploy
```

Están disponibles las siguientes opciones de implementación de contenedor personalizado:

| Opción     | Descripción                            |
| ------------ | -------------------------------------- |
| **`--registry`** | Nombre de un registro de Docker en el que el usuario actual ha iniciado sesión. |
| **`--platform`** | Plataforma de hospedaje de la aplicación de función. Las opciones válidas son `kubernetes` |
| **`--name`** | Nombre de la aplicación de función. |
| **`--max`**  | Opcionalmente, establece el número máximo de instancias de la aplicación de función que se va a implementar. |
| **`--min`**  | Opcionalmente, establece el número mínimo de instancias de la aplicación de función que se va a implementar. |
| **`--config`** | Establece un archivo de configuración de implementación opcional. |

## <a name="monitoring-functions"></a>Supervisión de funciones

La forma recomendada de supervisar la ejecución de sus funciones, es usar la integración con Azure Application Insights. También puede transmitir los registros de ejecución al equipo local. Para más información, consulte [Supervisión de Azure Functions](functions-monitoring.md).

### <a name="application-insights-integration"></a>Integración de Application Insights

Al crear la aplicación de funciones en Azure, la integración de Application Insights debe estar habilitada. Si, por alguna razón, la aplicación de funciones no está conectada a una instancia de Application Insights, es fácil llevar a cabo esta integración en Azure Portal. 

[!INCLUDE [functions-connect-new-app-insights.md](../../includes/functions-connect-new-app-insights.md)]

### <a name="enable-streaming-logs"></a>Habilitación de los registros de streaming

Puede ver una secuencia de archivos de registro que generan las funciones en una sesión de línea de comandos en el equipo local. 

#### <a name="native-streaming-logs"></a>Registros de streaming nativos

[!INCLUDE [functions-streaming-logs-core-tools](../../includes/functions-streaming-logs-core-tools.md)]

Este tipo de registros de streaming requiere que se habilite la integración de Application Insights para la aplicación de funciones.   


## <a name="next-steps"></a>Pasos siguientes

Aprenda a desarrollar, probar y publicar funciones de Azure Functions mediante el [módulo de aprendizaje de Microsoft](https://docs.microsoft.com/learn/modules/develop-test-deploy-azure-functions-with-core-tools/) de Azure Functions Core Tools. Azure Functions Core Tools es de [código abierto y se hospeda en GitHub](https://github.com/azure/azure-functions-cli).  
Para notificar un error o realizar una solicitud de característica, [abra un problema de GitHub](https://github.com/azure/azure-functions-cli/issues).

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Azure Portal]: https://portal.azure.com 
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows
[`FUNCTIONS_WORKER_RUNTIME`]: functions-app-settings.md#functions_worker_runtime
[`AzureWebJobsStorage`]: functions-app-settings.md#azurewebjobsstorage
[conjuntos de extensión]: functions-bindings-register.md#extension-bundles
