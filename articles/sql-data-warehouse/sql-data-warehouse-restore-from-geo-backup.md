---
title: Restauración de un almacenamiento de datos desde una copia de seguridad geográfica
description: Guía para la restauración geográfica de una instancia de Azure SQL Data Warehouse.
services: sql-data-warehouse
author: anumjs
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.subservice: manage
ms.date: 07/12/2019
ms.author: anjangsh
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 69ba3ed981a27dfff41ea9ea52e1da769a9366c4
ms.sourcegitcommit: b5d646969d7b665539beb18ed0dc6df87b7ba83d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/26/2020
ms.locfileid: "76759642"
---
# <a name="geo-restore-azure-sql-data-warehouse"></a>Restauración geográfica de Azure SQL Data Warehouse

En este artículo, aprenderá a restaurar el almacenamiento de datos desde una copia de seguridad geográfica a través de Azure Portal y PowerShell.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

**Compruebe la capacidad DTU**. Cada instancia de SQL Data Warehouse está hospedada en un servidor SQL Server (por ejemplo, myserver.database.windows.net) que tiene una cuota de DTU predeterminada. Compruebe que el servidor SQL Server tiene suficiente cuota de DTU restante para la base de datos en proceso de restauración. Para más información sobre cómo calcular la unidad DTU necesaria o solicitar más DTU, consulte cómo [solicitar un cambio en la cuota de DTU](sql-data-warehouse-get-started-create-support-ticket.md).

## <a name="restore-from-an-azure-geographical-region-through-powershell"></a>Uso de PowerShell para restaurar desde una región geográfica de Azure

Para realizar una restauración a partir de una copia de seguridad de replicación geográfica use el cmdlet [Get-AzSqlDatabaseGeoBackup](https://docs.microsoft.com/powershell/module/az.sql/get-azsqldatabasegeobackup) y [Restore-AzSqlDatabase](https://docs.microsoft.com/powershell/module/az.sql/restore-azsqldatabase).

> [!NOTE]
> Puede realizar una restauración geográfica en Gen2. Para ello, especifique Gen2 ServiceObjectiveName (p. ej., DW1000**c**) como parámetro opcional.
>

1. Antes de empezar, asegúrese de [instalar Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview).
2. Abra PowerShell.
2. Conéctese a su cuenta de Azure y enumere todas las suscripciones asociadas a su cuenta.
3. Seleccione la suscripción que contiene el almacenamiento de datos que se va a restaurar.
4. Obtenga el almacenamiento de datos que quiere recuperar.
5. Cree la solicitud de recuperación para el almacenamiento de datos.
6. Compruebe el estado del almacenamiento de datos restaurado geográficamente.
7. Para configurar la base de datos una vez finalizada la restauración, consulte [Configuración de la base de datos después de realizar la recuperación]( ../sql-database/sql-database-disaster-recovery.md#configure-your-database-after-recovery).

```Powershell
$SubscriptionName="<YourSubscriptionName>"
$ResourceGroupName="<YourResourceGroupName>"
$ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
$TargetResourceGroupName="<YourTargetResourceGroupName>" # Restore to a different logical server.
$TargetServerName="<YourtargetServerNameWithoutURLSuffixSeeNote>"  
$DatabaseName="<YourDatabaseName>"
$NewDatabaseName="<YourDatabaseName>"
$TargetServiceObjective="<YourTargetServiceObjective-DWXXXc>"

Connect-AzAccount
Get-AzSubscription
Select-AzSubscription -SubscriptionName $SubscriptionName
Get-AzureSqlDatabase -ServerName $ServerName

# Get the data warehouse you want to recover
$GeoBackup = Get-AzSqlDatabaseGeoBackup -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName

# Recover data warehouse
$GeoRestoredDatabase = Restore-AzSqlDatabase –FromGeoBackup -ResourceGroupName $TargetResourceGroupName -ServerName $TargetServerName -TargetDatabaseName $NewDatabaseName –ResourceId $GeoBackup.ResourceID -ServiceObjectiveName $TargetServiceObjective

# Verify that the geo-restored data warehouse is online
$GeoRestoredDatabase.status
```

La base de datos recuperada estará habilitada para TDE si la base de datos de origen está habilitada para TDE.

## <a name="restore-from-an-azure-geographical-region-through-azure-portal"></a>Uso de Azure Portal para restaurar desde una región geográfica de Azure

Siga los pasos que se describen a continuación para restaurar una instancia de Azure SQL Data Warehouse desde una copia de seguridad geográfica:

1. Inicie sesión en su cuenta de [Azure Portal](https://portal.azure.com/).
1. Haga clic en **+ Crear un recurso**, busque SQL Data Warehouse y haga clic en **Crear**.

    ![Nuevo DW](./media/sql-data-warehouse-restore-from-geo-backup/georestore-new.png)
1. Rellene la información solicitada en la pestaña **Aspectos básicos** y haga clic en **Siguiente: Configuración adicional**.

    ![Aspectos básicos](./media/sql-data-warehouse-restore-from-geo-backup/georestore-dw-1.png)
1. En el parámetro **Usar datos existentes**, haga clic en **Copia de seguridad** y luego seleccione la copia de seguridad adecuada en las opciones de desplazamiento. Haga clic en **Revisar y crear**.
 
   ![copia de seguridad](./media/sql-data-warehouse-restore-from-geo-backup/georestore-select.png)
2. Una vez que se haya restaurado el almacenamiento de datos, compruebe que el estado es **En línea**.

## <a name="next-steps"></a>Pasos siguientes
- [Restauración de un almacenamiento de datos existente](sql-data-warehouse-restore-active-paused-dw.md)
- [Restauración de un almacén de datos eliminado](sql-data-warehouse-restore-deleted-dw.md)