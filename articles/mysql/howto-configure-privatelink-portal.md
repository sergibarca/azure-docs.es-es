---
title: Private Link para el método de instalación del portal de Azure Database for MySQL (versión preliminar)
description: Aprenda a configurar una instancia de Private Link para Azure Database for MySQL desde Azure Portal
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 01/09/2020
ms.openlocfilehash: 6ad8726e61f4f8e79cfa9c437b3f0ea876b55e27
ms.sourcegitcommit: b8f2fee3b93436c44f021dff7abe28921da72a6d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/18/2020
ms.locfileid: "77425483"
---
# <a name="create-and-manage-private-link-for-azure-database-for-mysql-preview-using-portal"></a>Creación y administración de Private Link para Azure Database for MySQL (versión preliminar) mediante el portal

Un punto de conexión privado es el bloque de creación fundamental para el vínculo privado en Azure. Permite que los recursos de Azure, como las máquinas virtuales, se comuniquen de manera privada con recursos de vínculos privados. En este artículo, obtendrá información sobre cómo usar Azure Portal para crear una VM en una instancia de Azure Virtual Network y un servidor de Azure Database for MySQL con un punto de conexión privado de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

> [!NOTE]
> Esta característica está disponible en todas las regiones de Azure donde Azure Database for MySQL admita los planes de tarifa de uso general y optimizado para memoria.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-an-azure-vm"></a>Creación de una máquina virtual de Azure

En esta sección, va a crear una red virtual y una subred para hospedar la VM que se usa para acceder al recurso de Private Link (un servidor MySQL en Azure).

### <a name="create-the-virtual-network"></a>Crear la red virtual
En esta sección, va a crear una red virtual y una subred para hospedar la máquina virtual que se usa para acceder al recurso de Private Link.

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** > **Redes** > **Red virtual**.
2. En **Creación de una red virtual**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Nombre | Escriba *MyVirtualNetwork*. |
    | Espacio de direcciones | Escriba *10.1.0.0/16*. |
    | Subscription | Seleccione su suscripción.|
    | Resource group | Seleccione **Crear nuevo**, escriba *myResourceGroup* y, después, seleccione **Aceptar**. |
    | Location | Seleccione **Oeste de Europa**.|
    | Subred: nombre | Escriba *mySubnet*. |
    | Subred: intervalo de direcciones | Escriba *10.1.0.0/24*. |
    |||
3. Deje el resto tal como está y seleccione **Crear**.

### <a name="create-virtual-machine"></a>Creación de la máquina virtual

1. En la parte superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Proceso** > **Máquina virtual**.

2. En **Creación de una máquina virtual: conceptos básicos**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **DETALLES DEL PROYECTO** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.  |
    | **DETALLES DE INSTANCIA** |  |
    | Nombre de la máquina virtual | Escriba *myVm*. |
    | Region | Seleccione **Oeste de Europa**. |
    | Opciones de disponibilidad | Deje el valor predeterminado **No se requiere redundancia de la infraestructura**. |
    | Imagen | Seleccione **Windows Server 2019 Datacenter**. |
    | Size | Deje el valor predeterminado **Estándar DS1 v2**. |
    | **CUENTA DE ADMINISTRADOR** |  |
    | Nombre de usuario | Escriba un nombre de usuario de su elección. |
    | Contraseña | Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    | Confirm Password | Vuelva a escribir la contraseña. |
    | **REGLAS DE PUERTO DE ENTRADA** |  |
    | Puertos de entrada públicos | Deje el valor predeterminado **Ninguno**. |
    | **AHORRE DINERO** |  |
    | ¿Ya tiene una licencia de Windows? | Deje el valor predeterminado **No**. |
    |||

1. Seleccione **Siguiente: Discos**.

1. En **Creación de una máquina virtual: Discos**, deje los valores predeterminados y seleccione **Siguiente: Redes**.

1. En **Creación de una máquina virtual: Redes**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Virtual network | Deje el valor predeterminado **MyVirtualNetwork**.  |
    | Espacio de direcciones | Deje el valor predeterminado **10.1.0.0/24**.|
    | Subnet | Deje el valor predeterminado **mySubnet (10.1.0.0/24)** .|
    | Dirección IP pública | Deje el valor predeterminado **(new) myVm-ip**. |
    | Puertos de entrada públicos | Seleccione **Permitir los puertos seleccionados**. |
    | Selección de puertos de entrada | Seleccione **HTTP** y **RDP**.|
    |||


1. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración.

1. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

## <a name="create-an-azure-database-for-mysql"></a>Creación de una instancia de Azure Database for MySQL

En esta sección, creará un servidor de Azure Database for MySQL en Azure. 

1. En la parte superior izquierda de la pantalla en Azure Portal, seleccione **Crear un recurso** > **Bases de datos** > **Azure Database for MySQL**.

1. En **Azure Database for MySQL**, indique la información siguiente:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.|
    | **Detalles del servidor** |  |
    |Nombre de servidor  | Escriba *myServer*. Si el nombre ya existe, cree uno único.|
    | Nombre de usuario administrador| Escriba el nombre de administrador que prefiera. |
    | Contraseña | Escriba una contraseña de su elección. La contraseña debe tener al menos ocho caracteres y cumplir con los requisitos definidos. |
    | Location | Seleccione una región de Azure en la que desea que se encuentre la instancia de MySQL Server. |
    |Versión  | Seleccione la versión de la base de datos de MySQL Server que se requiere.|
    | Proceso y almacenamiento| Seleccione el plan de tarifa que sea necesario para el servidor en función de la carga de trabajo. |
    |||
 
7. Seleccione **Aceptar**. 
8. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración. 
9. Cuando reciba el mensaje Validación superada, seleccione **Crear**. 
10. Cuando reciba el mensaje Validación superada, seleccione Crear. 

## <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado

En esta sección, creará una instancia de MySQL Server y le agregará un punto de conexión privado. 

1. En la parte superior izquierda de la pantalla en Azure Portal, seleccione **Crear un recurso** > **Redes** > **Private Link Center (versión preliminar)** .

2. En **Centro de vínculos privados: Información general**, en la opción**Build a private connection to a service** (Crear una conexión privada a un servicio), seleccione **Start** (Iniciar).

    ![Información general de Private Link](media/concepts-data-access-and-security-private-link/privatelink-overview.png)

1. En **Create a private endpoint (Preview) - Basics** (Crear un punto de conexión privado [versión preliminar]: Conceptos básicos), escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Subscription | Seleccione su suscripción. |
    | Resource group | Seleccione **myResourceGroup**. Lo creó en la sección anterior.|
    | **Detalles de instancia** |  |
    | Nombre | Escriba *myPrivateEndpoint*. Si el nombre ya existe, cree uno único. |
    |Region|Seleccione **Oeste de Europa**.|
    |||

5. Seleccione **Siguiente: Resource** (Siguiente: Recurso).
6. En **Create a private endpoint - Resource** (Crear un punto de conexión privado: recurso), escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    |Método de conexión  | Seleccione Connect to an Azure resource in my directory (Conectarse a un recurso de Azure en mi directorio).|
    | Subscription| Seleccione su suscripción. |
    | Tipo de recurso | Seleccione **Microsoft.DBforMySQL/servers**. |
    | Resource |Seleccione *miServidor*.|
    |Recurso secundario de destino |Seleccione *mysqlServer*.|
    |||
7. Seleccione **Siguiente: Configuration** (Siguiente: Configuración).
8. En **Create a private endpoint (Preview) - Configuración** (Crear un punto de conexión privado [versión preliminar]: Configuración), escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    |**REDES**| |
    | Virtual network| Seleccione *MyVirtualNetwork*. |
    | Subnet | Seleccione  *mySubnet*. |
    |**INTEGRACIÓN DE DNS PRIVADO**||
    |Integración con una zona DNS privada |Seleccione **Sí**. |
    |Zona DNS privada |Seleccione *(New)privatelink.mysql.database.azure.com* |
    |||

1. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración. 
2. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**. 

    ![Private Link creado](media/concepts-data-access-and-security-private-link/show-mysql-private-link.png)

## <a name="connect-to-a-vm-using-remote-desktop-rdp"></a>Conéctese a una máquina virtual mediante Escritorio remoto (RDP)


Después de crear **myVm**, conéctese a ella desde Internet como se indica a continuación: 

1. En la barra de búsqueda del portal, escriba *myVm*.

1. Seleccione el botón **Conectar**. Después de seleccionar el botón **Conectar**, se abre **Conectar a máquina virtual**.

1. Seleccione **Descargar archivo RDP**. Azure crea un archivo de Protocolo de Escritorio remoto ( *.rdp*) y lo descarga en su equipo.

1. Abra el archivo *downloaded.rdp*.

    1. Cuando se le pida, seleccione **Conectar**.

    1. Escriba el nombre de usuario y la contraseña que especificó al crear la VM.

        > [!NOTE]
        > Es posible que tenga que seleccionar **Más opciones** > **Usar otra cuenta** para especificar las credenciales que escribió al crear la máquina virtual.

1. Seleccione **Aceptar**.

1. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe una advertencia de certificado, seleccione **Sí** o **Continuar**.

1. Una vez que aparezca el escritorio de la máquina virtual, minimícelo para volver a su escritorio local.

## <a name="access-the-mysql-server-privately-from-the-vm"></a>Acceso al servidor de MySQL de forma privada desde la VM

1. En el Escritorio remoto de  *myVm*, abra PowerShell.

2. Escriba  `nslookup  myServer.privatelink.mysql.database.azure.com`. 

    Recibirá un mensaje similar a este:
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    myServer.privatelink.mysql.database.azure.com
    Address:  10.1.3.4
    ```

3. Pruebe la conexión de vínculo privado para el servidor MySQL con cualquier cliente disponible. En el ejemplo siguiente se ha usado [MySQL Workbench](https://dev.mysql.com/doc/workbench/en/wb-installing-windows.html) para realizar la operación.

4. En **Nueva conexión**, escriba o seleccione esta información:

    | Configuración | Value |
    | ------- | ----- |
    | Tipo de servidor| Seleccione **MySQL**.|
    | Nombre de servidor| Seleccione *myServer.privatelink.mysql.database.azure.com* |
    | Nombre de usuario | Escriba el nombre de usuario como username@servername, que se proporciona durante la creación del servidor MySQL. |
    |Contraseña |Escriba una contraseña proporcionada durante la creación del servidor MySQL. |
    |SSL|Seleccione **Requerido**.|
    ||

5. Seleccione Conectar.

6. Examine las bases de datos en el menú izquierdo.

7. (Opcionalmente) Cree o consulte la información del servidor MySQL.

8. Cierre la conexión de escritorio remoto a myVm.

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando haya terminado de usar el punto de conexión privado, el servidor MySQL y la VM, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba *myResourceGroup* en el cuadro **Buscar** de la parte superior del portal y seleccione *myResourceGroup* en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba myResourceGroup en **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En esta guía paso a paso ha creado una VM en una red virtual, una instancia de Azure Database for MySQL y un punto de conexión privado para el acceso privado. Se ha conectado a una VM desde Internet y se ha comunicado de forma segura con el servidor MySQL mediante Private Link. Para más información sobre los puntos de conexión privados, consulte [¿Qué es un punto de conexión privado de Azure?](https://docs.microsoft.com/azure/private-link/private-endpoint-overview).
