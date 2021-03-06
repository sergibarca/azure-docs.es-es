---
title: Visualización y descarga de los datos de uso y los cargos de Azure
description: En este artículo se describe cómo descargar o ver los datos de uso y los cargos diarios de Azure.
keywords: billing usage, usage charges, usage download, view usage, azure invoice, azure usage
author: bandersmsft
ms.author: banders
tags: billing
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 156f47e7a5571d37d836e68c7817352626fbd734
ms.sourcegitcommit: 333af18fa9e4c2b376fa9aeb8f7941f1b331c11d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/13/2020
ms.locfileid: "77200255"
---
# <a name="view-and-download-your-azure-usage-and-charges"></a>Visualización y descarga de los datos de uso y los cargos de Azure

Puede descargar un desglose diario del uso y los cargos de Azure en Azure Portal. Solo determinados roles tienen permiso para obtener la información de uso de Azure, como el administrador de la cuenta o el administrador de Enterprise. Para obtener más información sobre cómo obtener acceso a la información de facturación, vea [Manage access to Azure billing using roles](../manage/manage-billing-access.md) (Administrar el acceso a la facturación de Azure mediante roles).

Si tiene un contrato de cliente de Microsoft (MCA), debe ser el propietario del perfil de facturación, colaborador, lector o administrador de facturación para ver los datos de uso y los cargos de Azure.  Si tiene un contrato Microsoft Partner Agreement (MPA), solo el administrador global o el agente de administrador de la organización asociada pueden ver y descargar el uso y los cargos de Azure. [Compruebe el tipo de la cuenta de facturación en Azure Portal](#check-your-billing-account-type).

## <a name="download-usage-from-the-azure-portal-csv"></a>Descarga de uso en Azure Portal (.csv)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque *Administración de costos + facturación*.

    ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/download-azure-daily-usage/portal-cm-billing-search.png)

1. Dependiendo de su acceso, es posible que tenga que seleccionar una cuenta de facturación o un perfil de facturación.
1. En el menú de la izquierda, seleccione **Facturas** en **Facturación**.
1. En la cuadrícula de la factura, busque la fila del período de facturación correspondiente al uso que quiere descargar.
1. Seleccione el **icono de descargas** o los puntos suspensivos (`...`) de la derecha.
1. Se abre el panel de descarga a la derecha. Seleccione **Descargar** en la sección **Detalles de uso**.

## <a name="download-usage-for-ea-customers"></a>Descarga del uso para los clientes de EA

Para ver y descargar los datos de uso como cliente de EA, debe ser administrador de Enterprise, o bien propietario de la cuenta o administrador de departamento con la directiva para ver los cargos habilitada.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Busque *Administración de costos + facturación*.

    ![Captura de pantalla que muestra la búsqueda en Azure Portal](./media/download-azure-daily-usage/portal-cm-billing-search.png)

1. Seleccione **Uso + cargos**.
1. Seleccione **Descargar** para el mes que quiere descargar.

## <a name="download-usage-for-pending-charges"></a>Descarga del uso para cargos pendientes

Si tiene un contrato de cliente de Microsoft, puede descargar el uso mensual hasta la fecha para el período de facturación actual. Estos cargos de utilización que todavía no se han facturado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Busque *Administración de costos + facturación*.
3. Seleccione un perfil de facturación. En función de su acceso, es posible que tenga que seleccionar primero una cuenta de facturación.
4. En el área **Información general**, busque los vínculos de descarga debajo de los cargos del mes hasta la fecha.
5. Seleccione **Uso y cargos de Azure**.

    ![Captura de pantalla que muestra la descarga de la hoja Información general](./media/download-azure-daily-usage/open-usage01.png)

## <a name="check-your-billing-account-type"></a>Comprobación del tipo de cuenta de facturación
[!INCLUDE [billing-check-account-type](../../../includes/billing-check-account-type.md)]

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de los cargos de factura y utilización, consulte:

- [Descripción de los términos de uso detallados de Microsoft Azure](understand-usage.md)
- [Comprender la factura de Microsoft Azure](review-individual-bill.md)
- [Visualización y descarga de la factura de Microsoft Azure](download-azure-invoice.md)
- [Visualización y descarga de los precios de Azure de la organización](../manage/ea-pricing.md)

Si tiene un contrato de cliente de Microsoft, vea:

- [Descripción de los términos de la utilización detallada de Azure del contrato de cliente de Microsoft](mca-understand-your-usage.md)
- [Descripción de los cargos de la factura del contrato de cliente de Microsoft](review-customer-agreement-bill.md)
- [Visualización y descarga de la factura de Microsoft Azure](download-azure-invoice.md)
- [Visualización y descarga de los documentos fiscales para el contrato de cliente de Microsoft](mca-download-tax-document.md)
- [Visualización y descarga de los precios de Azure de la organización](../manage/ea-pricing.md)
