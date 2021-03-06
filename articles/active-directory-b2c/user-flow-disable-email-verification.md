---
title: Deshabilitación de la comprobación de correos electrónicos durante la suscripción de consumidores
titleSuffix: Azure AD B2C
description: Aprenda a deshabilitar la comprobación del correo electrónico durante la suscripción de consumidores en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 10613bd2d6219272248f882e45ae1c33d2cc9d61
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2020
ms.locfileid: "85384216"
---
# <a name="disable-email-verification-during-customer-sign-up-in-azure-active-directory-b2c"></a>Deshabilitar la comprobación del correo electrónico durante la suscripción de consumidores en Azure Active Directory B2C

[!INCLUDE [disable email verification intro](../../includes/active-directory-b2c-disable-email-verification.md)]

Siga estos pasos para deshabilitar la comprobación de correo electrónico:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com)
1. Use el filtro **Directorio y suscripción** del menú superior para seleccionar el directorio que contiene el inquilino de Azure AD B2C.
1. En el menú de la izquierda, seleccione **Azure AD B2C**. O bien, seleccione **Todos los servicios** y busque y seleccione **Azure AD B2C**.
1. Seleccione **Flujos de usuario**.
1. Seleccione el flujo de usuario para el que desea deshabilitar la comprobación de correo electrónico. Por ejemplo, *B2C_1_signinsignup*.
1. Seleccione **Diseños de página**.
1. Seleccione **Página de suscripción de cuenta local**.
1. En **Atributos de usuario**, seleccione **Dirección de correo electrónico**.
1. En el menú desplegable **REQUIERE VERIFICACIÓN**, seleccione **No**.
1. Seleccione **Guardar**. La comprobación de correo electrónico se deshabilitará en este flujo de usuario.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [personalizar la interfaz de usuario en Azure Active Directory B2C](customize-ui-overview.md)

