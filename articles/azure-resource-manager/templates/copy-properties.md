---
title: Definición de varias instancias de una propiedad
description: Utilice la operación de copia en una plantilla de Azure Resource Manager para realizar varias iteraciones cuando cree una propiedad en un recurso.
ms.topic: conceptual
ms.date: 04/14/2020
ms.openlocfilehash: 61122b01889da832a73f729833ab0af676904d54
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2020
ms.locfileid: "84678467"
---
# <a name="property-iteration-in-arm-templates"></a>Iteración de propiedades en las plantillas de ARM

En este artículo, se explica cómo se crean varias instancias de una propiedad en la plantilla de Azure Resource Manager (ARM). Al agregar el elemento **copy** a la sección de propiedades de un recurso, puede establecer de forma dinámica el número de elementos de una variable durante la implementación. Asimismo, evitará tener que repetir la sintaxis de la plantilla.

También puede usar el elemento copy con [recursos](copy-resources.md), [variables](copy-variables.md) y [salidas](copy-outputs.md).

## <a name="syntax"></a>Sintaxis

El elemento copy tiene el siguiente formato general:

```json
"copy": [
  {
    "name": "<name-of-loop>",
    "count": <number-of-iterations>,
    "input": <values-for-the-property>
  }
]
```

En **nombre**, especifique el nombre de la propiedad del recurso que desea crear.

La propiedad **count** especifica el número de iteraciones que desea realizar en la propiedad.

La propiedad **input** especifica las propiedades que desea repetir. Tiene que crear una matriz de elementos construida a partir del valor de la propiedad **input**.

## <a name="copy-limits"></a>Límites de copia

El valor de count no puede superar 800.

El valor de count no puede ser un número negativo. Puede ser cero si implementa la plantilla con una versión reciente de la CLI de Azure, PowerShell o la API REST. Específicamente, se debe usar:

* Azure PowerShell **2.6** o posterior
* CLI de Azure **2.0.74** o posterior
* API REST versión **2019-05-10** o posterior
* Las [implementaciones vinculadas](linked-templates.md) deben usar la versión **10-05-2019** o posterior de la API para el tipo de recurso de implementación.

Las versiones anteriores de PowerShell, la CLI y API REST no admiten un valor de count de cero.

## <a name="property-iteration"></a>Iteración de propiedades

En el ejemplo siguiente se muestra cómo aplicar `copy` a la propiedad dataDisks en una máquina virtual:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "numberOfDataDisks": {
      "type": "int",
      "minValue": 0,
      "maxValue": 16,
      "defaultValue": 16,
      "metadata": {
        "description": "The number of dataDisks to create."
      }
    },
    ...
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2017-03-30",
      ...
      "properties": {
        "storageProfile": {
          ...
          "copy": [
            {
              "name": "dataDisks",
              "count": "[parameters('numberOfDataDisks')]",
              "input": {
                "diskSizeGB": 1023,
                "lun": "[copyIndex('dataDisks')]",
                "createOption": "Empty"
              }
            }
          ]
        }
      }
    }
  ]
}
```

Tenga en cuenta que, cuando se usa `copyIndex` dentro de una iteración de propiedad, debe proporcionar el nombre de la iteración. La iteración de propiedades también admite un argumento de desplazamiento. El desplazamiento debe aparecer después del nombre de la iteración, como copyIndex ("dataDisks", 1).

Resource Manager expande la matriz `copy` durante la implementación. El nombre de la matriz se convierte en el nombre de la propiedad. Los valores de entrada se convierten en las propiedades del objeto. La plantilla implementada se convierte en:

```json
{
  "name": "examplevm",
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2017-03-30",
  "properties": {
    "storageProfile": {
      "dataDisks": [
        {
          "lun": 0,
          "createOption": "Empty",
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "createOption": "Empty",
          "diskSizeGB": 1023
        },
        {
          "lun": 2,
          "createOption": "Empty",
          "diskSizeGB": 1023
        }
      ],
      ...
```

La operación de copia es útil al trabajar con matrices, ya que puede iterar a través de cada elemento de la matriz. Use la función `length` en la matriz para especificar el número de iteraciones, y `copyIndex` para recuperar el índice actual de la matriz.

En la plantilla de ejemplo siguiente se crea un grupo de conmutación por error para las bases de datos que se pasan como una matriz.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "primaryServerName": {
            "type": "string"
        },
        "secondaryServerName": {
            "type": "string"
        },
        "databaseNames": {
            "type": "array",
            "defaultValue": [
                "mydb1",
                "mydb2",
                "mydb3"
            ]
        }
    },
    "variables": {
        "failoverName": "[concat(parameters('primaryServerName'),'/', parameters('primaryServerName'),'failovergroups')]"
    },
    "resources": [
        {
            "type": "Microsoft.Sql/servers/failoverGroups",
            "apiVersion": "2015-05-01-preview",
            "name": "[variables('failoverName')]",
            "properties": {
                "readWriteEndpoint": {
                    "failoverPolicy": "Automatic",
                    "failoverWithDataLossGracePeriodMinutes": 60
                },
                "readOnlyEndpoint": {
                    "failoverPolicy": "Disabled"
                },
                "partnerServers": [
                    {
                        "id": "[resourceId('Microsoft.Sql/servers', parameters('secondaryServerName'))]"
                    }
                ],
                "copy": [
                    {
                        "name": "databases",
                        "count": "[length(parameters('databaseNames'))]",
                        "input": "[resourceId('Microsoft.Sql/servers/databases', parameters('primaryServerName'), parameters('databaseNames')[copyIndex('databases')])]"
                    }
                ]
            }
        }
    ],
    "outputs": {
    }
}
```

El elemento de copia es una matriz por lo que puede especificar más de una propiedad para el recurso.

```json
{
  "type": "Microsoft.Network/loadBalancers",
  "apiVersion": "2017-10-01",
  "name": "exampleLB",
  "properties": {
    "copy": [
      {
        "name": "loadBalancingRules",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      },
      {
        "name": "probes",
        "count": "[length(parameters('loadBalancingRules'))]",
        "input": {
          ...
        }
      }
    ]
  }
}
```

Puede usar la iteración de recursos y propiedades conjuntamente. Haga referencia a la iteración de la propiedad por el nombre.

```json
{
  "type": "Microsoft.Network/virtualNetworks",
  "apiVersion": "2018-04-01",
  "name": "[concat(parameters('vnetname'), copyIndex())]",
  "copy":{
    "count": 2,
    "name": "vnetloop"
  },
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[parameters('addressPrefix')]"
      ]
    },
    "copy": [
      {
        "name": "subnets",
        "count": 2,
        "input": {
          "name": "[concat('subnet-', copyIndex('subnets'))]",
          "properties": {
            "addressPrefix": "[variables('subnetAddressPrefix')[copyIndex('subnets')]]"
          }
        }
      }
    ]
  }
}
```

## <a name="example-templates"></a>Plantillas de ejemplo

En los ejemplos siguientes, se muestran escenarios comunes en los que se crean varios valores para una propiedad.

|Plantilla  |Descripción  |
|---------|---------|
|[Implementación de máquinas virtuales con un número variable de discos de datos](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-windows-copy-datadisks) |Implementa varios discos de datos con una máquina virtual. |

## <a name="next-steps"></a>Pasos siguientes

* Para seguir los pasos de un tutorial, consulte [Tutorial: Creación de varias instancias de recursos con plantillas de ARM](template-tutorial-create-multiple-instances.md).
* Para otros usos del elemento copy, consulte:
  * [Iteración de recursos en las plantillas de ARM](copy-resources.md)
  * [Iteración de variables en las plantillas de ARM](copy-variables.md)
  * [Iteración de salida en las plantillas de ARM](copy-outputs.md)
* Para más información sobre las secciones de una plantilla, consulte el artículo sobre cómo [crear plantillas de ARM](template-syntax.md).
* Para más información sobre cómo implementar su plantilla, consulte el artículo [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](deploy-powershell.md).

