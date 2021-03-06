---
title: Reparación de un trabajo de importación de Azure Import/Export (versión 1) | Microsoft Docs
description: Obtenga información sobre cómo reparar un trabajo de importación que se creó y ejecutó con el servicio Azure Import/Export.
author: twooley
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: twooley
ms.subservice: common
ms.openlocfilehash: f5db321d8c4a6e42591a82b0ed8eb6bc6e93bad4
ms.sourcegitcommit: 5ab4f7a81d04a58f235071240718dfae3f1b370b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 12/10/2019
ms.locfileid: "74973890"
---
# <a name="repairing-an-import-job"></a>Reparación de un trabajo de importación
El servicio Microsoft Azure Import/Export puede dar error al copiar algunos de los archivos o partes de un archivo en Azure Blob service. Estas son algunas razones de los errores:  
  
-   Archivos dañados  
  
-   Unidades dañadas  
  
-   La clave de la cuenta de almacenamiento cambió mientras se transfería el archivo.  
  
Puede ejecutar la herramienta Microsoft Azure Import/Export con los archivos de registro de copia del trabajo de importación y dicha herramienta carga los archivos que faltan (o partes de un archivo) a la cuenta de Microsoft Azure Storage para completar el trabajo de importación.  
  
## <a name="repairimport-parameters"></a>Parámetros de RepairImport

Se pueden modificar los parámetros siguientes con **RepairImport**: 
  
|||  
|-|-|  
|**/r:** &lt;ArchivoReparación\>|**Requerido.** Ruta de acceso al archivo de reparación, que realiza un seguimiento del progreso de la reparación y le permite reanudar una reparación interrumpida. Cada unidad debe tener un solo archivo de reparación. Cuando inicie una reparación para una unidad determinada, pasa la ruta de acceso a un archivo de reparación que aún no existe. Para reanudar una reparación interrumpida, debe pasar el nombre de un archivo de reparación existente. Siempre se debe especificar el archivo de reparación correspondiente a la unidad de destino.|  
|**/logdir:** &lt;DirectorioRegistro\>|**Opcional.** El directorio de registro. Los archivos de registro detallados se escriben en este directorio. Si no se especifica un directorio de registro, se usa el directorio actual como directorio de registro.|  
|**/d:** &lt;DirectoriosDestino\>|**Requerido.** Uno o varios directorios separados por puntos y coma que contienen los archivos originales que se importaron. También se puede utilizar la unidad de importación, pero no es necesaria si están disponibles ubicaciones alternativas de los archivos originales.|  
|**/bk:** &lt;ClaveBitLocker\>|**Opcional.** Debe especificar la clave de BitLocker si desea que la herramienta desbloquee una unidad cifrada donde están disponibles los archivos originales.|  
|**/sn:** &lt;NombreCuentaAlmacenamiento\>|**Requerido.** El nombre de la cuenta de almacenamiento para el trabajo de importación.|  
|**/sk:** &lt;ClaveCuentaAlmacenamiento\>|**Necesario** únicamente si no se especifica un SAS del contenedor. La clave de cuenta para la cuenta de almacenamiento correspondiente al trabajo de importación.|  
|**/csas:** &lt;SasContenedor\>|**Requerido** únicamente si no se especifica la clave de cuenta de almacenamiento. El SAS del contenedor para acceder a los blobs asociados al trabajo de importación.|  
|**/CopyLogFile:** &lt;ArchivoRegistroCopiaUnidad\>|**Requerido.** Ruta de acceso al archivo de registro de copia de la unidad (registro detallado o registro de errores). El archivo lo genera el servicio Microsoft Windows Import/Export y se puede descargar desde el almacenamiento de blobs asociado al trabajo. El archivo de registro de copia contiene información sobre blobs con errores o archivos que deben repararse.|  
|**/PathMapFile:** &lt;ArchivoAsignaciónRutaAccesoUnidad\>|**Opcional.** Ruta de acceso a un archivo de texto que puede usar para resolver ambigüedades si tiene varios archivos con el mismo nombre que se importaban en el mismo trabajo. La primera vez que se ejecuta la herramienta, se puede rellenar este archivo con todos los nombres ambiguos. Las ejecuciones posteriores de la herramienta usan este archivo para resolver las ambigüedades.|  
  
## <a name="using-the-repairimport-command"></a>Uso del comando RepairImport  
Para reparar datos de importación haciendo streaming de los datos a través de la red, debe especificar los directorios que contienen los archivos originales que se estaban importando mediante el parámetro `/d`. También debe especificar el archivo de registro de copia que descargó desde la cuenta de almacenamiento. Una línea de comandos típica para reparar un trabajo de importación con errores parciales tiene el siguiente aspecto:  
  
```  
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log  
```  
  
En el ejemplo siguiente de archivo de registro de copia, un fragmento de 64 KB de un archivo estaba dañado en la unidad que se envió para el trabajo de importación. Puesto que es el único error indicado, el resto de los blobs del trabajo se importaron correctamente.  
  
```xml
<?xml version="1.0" encoding="utf-8"?>  
<DriveLog>  
 <DriveId>9WM35C2V</DriveId>  
 <Blob Status="CompletedWithErrors">  
 <BlobPath>pictures/animals/koala.jpg</BlobPath>  
 <FilePath>\animals\koala.jpg</FilePath>  
 <Length>163840</Length>  
 <ImportDisposition Status="Overwritten">overwrite</ImportDisposition>  
 <PageRangeList>  
  <PageRange Offset="65536" Length="65536" Hash="AA2585F6F6FD01C4AD4256E018240CD4" Status="Corrupted" />  
 </PageRangeList>  
 </Blob>  
 <Status>CompletedWithErrors</Status>  
</DriveLog>  
```
  
Cuando este registro de copia se pasa a la herramienta Azure Import/Export, esta intenta finalizar la importación del archivo copiando el contenido que falta a través de la red. Siguiendo el ejemplo anterior, la herramienta busca el archivo original `\animals\koala.jpg` en los dos directorios `C:\Users\bob\Pictures` y `X:\BobBackup\photos`. Si el archivo `C:\Users\bob\Pictures\animals\koala.jpg` existe, la herramienta Azure Import/Export copia el intervalo de datos que falta en el blob correspondiente `http://bobmediaaccount.blob.core.windows.net/pictures/animals/koala.jpg`.  
  
## <a name="resolving-conflicts-when-using-repairimport"></a>Resolución de conflictos cuando se usa RepairImport  
En algunas situaciones, es posible que la herramienta no pueda encontrar o abrir el archivo necesario por uno de los siguientes motivos: no se pudo encontrar el archivo, el archivo no está accesible, el nombre de archivo es ambiguo o el contenido del archivo ya no es correcto.  
  
Podría producirse un error ambiguo si la herramienta intenta buscar `\animals\koala.jpg` y hay un archivo con ese nombre en `C:\Users\bob\pictures` y `X:\BobBackup\photos`. Es decir, tanto `C:\Users\bob\pictures\animals\koala.jpg` como `X:\BobBackup\photos\animals\koala.jpg` existen en las unidades de trabajo de importación.  
  
La opción `/PathMapFile` le permite resolver estos errores. Puede especificar el nombre del archivo que contiene la lista de archivos que la herramienta no pudo identificar correctamente. El ejemplo de línea de comandos siguiente rellena `9WM35C2V_pathmap.txt`:  
  
```
WAImportExport.exe RepairImport /r:C:\WAImportExport\9WM35C2V.rep /d:C:\Users\bob\Pictures;X:\BobBackup\photos /sn:bobmediaaccount /sk:VkGbrUqBWLYJ6zg1m29VOTrxpBgdNOlp+kp0C9MEdx3GELxmBw4hK94f7KysbbeKLDksg7VoN1W/a5UuM2zNgQ== /CopyLogFile:C:\WAImportExport\9WM35C2V.log /PathMapFile:C:\WAImportExport\9WM35C2V_pathmap.txt  
```
  
La herramienta escribirá después las rutas de acceso de archivo problemáticas en `9WM35C2V_pathmap.txt`, una en cada línea. Por ejemplo, el archivo puede contener las entradas siguientes después de ejecutar el comando:  
 
```
\animals\koala.jpg  
\animals\kangaroo.jpg  
```
  
 Para cada archivo de la lista, debería intentar localizarlo y abrirlo para asegurarse de que está disponible para la herramienta. Si desea indicar a la herramienta explícitamente dónde encontrar un archivo, puede modificar el archivo de asignación de rutas de acceso y agregar la ruta de acceso a cada archivo en la misma línea, separada por un carácter de tabulación:  
  
```
\animals\koala.jpg           C:\Users\bob\Pictures\animals\koala.jpg  
\animals\kangaroo.jpg        X:\BobBackup\photos\animals\kangaroo.jpg  
```
  
Después de poner los archivos necesarios a disposición de la herramienta o actualizar el archivo de asignación de rutas de acceso, puede volver a ejecutar la herramienta para completar el proceso de importación.  
  
## <a name="next-steps"></a>Pasos siguientes
 
* [Configuración de la herramienta Azure Import/Export](storage-import-export-tool-setup-v1.md)   
* [Preparación de unidades de disco duro para un trabajo de importación](../storage-import-export-tool-preparing-hard-drives-import-v1.md)   
* [Revisión del estado del trabajo con archivos de registro de copia](storage-import-export-tool-reviewing-job-status-v1.md)   
* [Reparación de un trabajo de exportación](../storage-import-export-tool-repairing-an-export-job-v1.md)   
* [Solución de problemas de la herramienta Azure Import/Export](storage-import-export-tool-troubleshooting-v1.md)
