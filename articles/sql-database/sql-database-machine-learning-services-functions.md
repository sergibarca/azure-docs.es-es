---
title: Escritura de funciones de R avanzadas
titleSuffix: Azure SQL Database Machine Learning Services (preview)
description: Obtenga información sobre cómo escribir una función de R para el cálculo estadístico avanzado en Azure SQL Database mediante Machine Learning Services (versión preliminar).
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: r
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: davidph
manager: cgronlun
ms.date: 04/11/2019
ms.openlocfilehash: 939798d5d9eb2843d7bbbbe74680342e4ce6ce95
ms.sourcegitcommit: 41ca82b5f95d2e07b0c7f9025b912daf0ab21909
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/13/2019
ms.locfileid: "60702459"
---
# <a name="write-advanced-r-functions-in-azure-sql-database-using-machine-learning-services-preview"></a>Escritura de funciones de R avanzadas en Azure SQL Database Machine Learning Services (versión preliminar)

En este artículo se describe cómo insertar funciones de utilidad y matemáticas de R en un procedimiento almacenado de SQL. Las funciones estadísticas avanzadas que son difíciles de implementar en T-SQL se pueden llevar a cabo en R con una sola línea de código.

[!INCLUDE[ml-preview-note](../../includes/sql-database-ml-preview-note.md)]

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/) antes de empezar.

- Para ejecutar el código de ejemplo en estos ejercicios, primero debe tener una instancia de Azure SQL Database con Machine Learning Services (con R) habilitada. Durante la versión preliminar pública, Microsoft le incorporará y habilitará el aprendizaje automático para la base de datos nueva o existente. Siga los pasos que se indican en [Suscríbase a la versión preliminar](sql-database-machine-learning-services-overview.md#signup).

- Asegúrese de que tiene instalada la versión más reciente de [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) (SSMS). Puede ejecutar los scripts de R con herramientas de consulta o de otro tipo de administración de base de datos, pero en este tutorial usará SSMS.

## <a name="create-a-stored-procedure-to-generate-random-numbers"></a>Creación de un procedimiento almacenado para generar números aleatorios

Por motivos de simplicidad, vamos a usar el paquete `stats` de R que se instala y carga de forma predeterminada con Azure SQL Database mediante Machine Learning Services (versión preliminar). El paquete contiene cientos de funciones para tareas estadísticas comunes, entre ellas la función `rnorm`. Esta función genera una cantidad especificada de números aleatorios mediante la distribución normal, dada una media y una desviación estándar.

Por ejemplo, el siguiente código R devuelve 100 números en una media de 50, dada una desviación estándar de 3.

```R
as.data.frame(rnorm(100, mean = 50, sd = 3));
```

Para llamar a esta línea de R desde T-SQL, ejecute `sp_execute_external_script` y agregue la función de R en el parámetro de script de R, como este:

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
OutputDataSet <- as.data.frame(rnorm(100, mean = 50, sd =3));
'
    , @input_data_1 = N'   ;'
WITH RESULT SETS(([Density] FLOAT NOT NULL));
```

¿Qué ocurriría si quisiera facilitar la generación de un conjunto diferente de números aleatorios?

Puede hacerlo con facilidad en combinación con SQL. Defina un procedimiento almacenado que obtiene los argumentos del usuario y, a continuación, pase esos argumentos al script de R como variables.

```sql
CREATE PROCEDURE MyRNorm (
    @param1 INT
    , @param2 INT
    , @param3 INT
    )
AS
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
OutputDataSet <- as.data.frame(rnorm(mynumbers, mymean, mysd));
'
    , @input_data_1 = N'   ;'
    , @params = N' @mynumbers int, @mymean int, @mysd int'
    , @mynumbers = @param1
    , @mymean = @param2
    , @mysd = @param3
WITH RESULT SETS(([Density] FLOAT NOT NULL));
```

- La primera línea define cada uno de los parámetros de entrada de SQL que son necesarios cuando se ejecuta el procedimiento almacenado.

- La línea que comienza con `@params` define todas las variables usadas por el código de R y los correspondientes tipos de datos de SQL.

- Las líneas inmediatamente a continuación asignan los nombres de parámetro de SQL a los nombres de variable de R correspondientes.

Ahora que ha ajustado la función de R en un procedimiento almacenado, puede llamar a la función fácilmente y pasar distintos valores, como a continuación:

```sql
EXECUTE MyRNorm @param1 = 100
    , @param2 = 50
    , @param3 = 3
```

## <a name="use-r-utility-functions-for-troubleshooting"></a>Uso de funciones de utilidad de R para solucionar problemas

El paquete `utils`, que se instala de forma predeterminada, proporciona una variedad de funciones de utilidad para investigar el entorno actual de R. Estas funciones pueden resultar útiles si se detectan discrepancias en el modo de actuar del código de R en SQL y en entornos exteriores. Por ejemplo, podría usar la función `memory.limit()` de R para obtener memoria para el entorno actual de R.

Dado que el paquete `utils` está instalado pero no se carga de forma predeterminada, debe usar la función `library()` para cargarlo primero.

```sql
EXECUTE sp_execute_external_script @language = N'R'
    , @script = N'
library(utils);
mymemory <- memory.limit();
OutputDataSet <- as.data.frame(mymemory);
'
    , @input_data_1 = N' ;'
WITH RESULT SETS(([Col1] INT NOT NULL));
```

> [!TIP]
> Muchos usuarios prefieren utilizar las funciones de temporización del sistema en R, como `system.time` y `proc.time`, para capturar el tiempo utilizado por los procesos de R y analizar problemas de rendimiento.
