---
title: Límites de Gremlin de Azure Cosmos DB
description: Documentación de referencia para las limitaciones en tiempo de ejecución del motor Graph
author: LuisBosquez
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: reference
ms.date: 10/04/2019
ms.author: lbosq
ms.openlocfilehash: 76ad787990c355d29613c05ca9fce31885a2eccc
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/27/2020
ms.locfileid: "72029853"
---
# <a name="azure-cosmos-db-gremlin-limits"></a>Límites de Gremlin de Azure Cosmos DB
En este artículo se tratan los límites del motor de Gremlin de Azure Cosmos DB y se explica cómo pueden afectar a los recorridos de los clientes.

Cosmos DB Gremlin se basa en la infraestructura de Cosmos DB. Por ese motivo, todos los límites que se explican en [Cuotas de servicio de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/concepts-limits) siguen siendo aplicables. 

## <a name="limits"></a>límites

Cuando se alcanza el límite de Gremlin, el recorrido se cancela y **x-ms-status-code** tiene el valor 429, lo que indica un error de limitación. Consulte los [encabezados de respuesta del servidor de Gremlin](gremlin-limits.md) para más información.

**Recurso**    | **Límite predeterminado** | **Explicación**
--- | --- | ---
*Longitud del script* | **64 KB** | Longitud máxima de un script de recorrido de Gremlin por solicitud.
*Profundidad de operadores* | **400** |  Número total de pasos únicos en un recorrido. Por ejemplo, el número de operadores de ```g.V().out()``` es 2: V() y out(); ```g.V('label').repeat(out()).times(100)``` tiene una profundidad de operadores de 3: V(), repeat() y out(), porque ```.times(100)``` es un parámetro para el operador ```.repeat()```.
*Grado de paralelismo* | **32** | Número máximo de particiones de almacenamiento consultadas en una única solicitud a la capa de almacenamiento. Este límite afectará a los grafos con cientos de particiones.
*Límite de repeticiones* | **32** | Número máximo de iteraciones que puede ejecutar un operador ```.repeat()```. En la mayoría de los casos, cada iteración del paso ```.repeat()``` se ejecuta mediante recorridos centrados en la amplitud, lo que significa que cualquier recorrido se limita a un máximo de 32 saltos entre vértices.
*Tiempo de expiración del recorrido* | **30 segundos** | El recorrido se cancelará cuando supere este tiempo. Cosmos DB Graph es una base de datos OLTP en la que la mayoría de los recorridos se completan en cuestión de milisegundos. Para realizar consultas OLAP en Cosmos DB Graph, use [Apache Spark](https://azure.microsoft.com/services/cosmos-db/) con [tramas de datos de grafos](https://spark.apache.org/docs/latest/sql-programming-guide.html#datasets-and-dataframes) y el [conector de Spark de Cosmos DB](https://github.com/Azure/azure-cosmosdb-spark).
*Tiempo de expiración de conexiones inactivas* | **1 hora** | Tiempo que el servicio Gremlin mantendrá abiertas las conexiones de WebSocket inactivas. Los paquetes TCP persistentes o las solicitudes HTTP persistentes no aumentarán la duración de la conexión por encima de este límite. El motor de Cosmos DB Graph considera que una conexión de WebSocket está inactiva si no tiene solicitudes de Gremlin en ejecución.
*Token de recursos por hora* | **100** | Número de tokens de recursos únicos que usan los clientes de Gremlin para conectarse a la cuenta de Gremlin en una región. Cuando la aplicación supera el límite de tokens únicos por hora, se devolverá `"Exceeded allowed resource token limit of 100 that can be used concurrently"` en la siguiente solicitud de autenticación.

## <a name="next-steps"></a>Pasos siguientes
* [Encabezados de respuesta de Gremlin en Azure Cosmos DB](gremlin-headers.md) 
* [Tokens de recursos de Azure Cosmos DB con Gremlin](how-to-use-resource-tokens-gremlin.md)
