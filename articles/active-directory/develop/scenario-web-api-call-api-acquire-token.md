---
title: Obtención de un token para una API web que llama a otras API web | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una API web que llama a otras API web que necesitan adquirir un token para la aplicación.
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 2721837459af24f39bb15ee17d394345cbb37eb1
ms.sourcegitcommit: 5d6ce6dceaf883dbafeb44517ff3df5cd153f929
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/29/2020
ms.locfileid: "76834117"
---
# <a name="a-web-api-that-calls-web-apis-acquire-a-token-for-the-app"></a>API web que llama a API web: Adquisición de un token para la aplicación

Una vez que ha creado un objeto de aplicación cliente, lo puede usar para adquirir un token y usarlo para llamar a una API web.

## <a name="code-in-the-controller"></a>Código del controlador

# <a name="aspnet-coretabaspnetcore"></a>[ASP.NET Core](#tab/aspnetcore)

A continuación, se muestra un ejemplo de código que se llama en las acciones de los controladores de API. Llama a una API de bajada denominada *todolist*.

```csharp
private async Task GetTodoList(bool isAppStarting)
{
 ...
 //
 // Get an access token to call the To Do service.
 //
 AuthenticationResult result = null;
 try
 {
  app = BuildConfidentialClient(HttpContext, HttpContext.User);
  result = await app.AcquireTokenSilent(Scopes, account)
                     .ExecuteAsync()
                     .ConfigureAwait(false);
 }
...
}
```

`BuildConfidentialClient()` es similar al escenario de [Una API web que llama a las API web: configuración de la aplicación](scenario-web-api-call-api-app-configuration.md). `BuildConfidentialClient()` crea una instancia de `IConfidentialClientApplication` con una caché que contiene información de una única cuenta. La cuenta se proporciona mediante el método `GetAccountIdentifier`.

El método `GetAccountIdentifier` usa las notificaciones asociadas con la identidad del usuario para el que la API web ha recibido el JSON Web Token (JWT):

```csharp
public static string GetMsalAccountId(this ClaimsPrincipal claimsPrincipal)
{
 string userObjectId = GetObjectId(claimsPrincipal);
 string tenantId = GetTenantId(claimsPrincipal);

 if (    !string.IsNullOrWhiteSpace(userObjectId)
      && !string.IsNullOrWhiteSpace(tenantId))
 {
  return $"{userObjectId}.{tenantId}";
 }

 return null;
}
```

# <a name="javatabjava"></a>[Java](#tab/java)
A continuación, se muestra un ejemplo de código que se llama en las acciones de los controladores de API. Llama a la API de nivel inferior de Microsoft Graph.

```java
@RestController
public class ApiController {

    @Autowired
    MsalAuthHelper msalAuthHelper;

    @RequestMapping("/graphMeApi")
    public String graphMeApi() throws MalformedURLException {

        String oboAccessToken = msalAuthHelper.getOboToken("https://graph.microsoft.com/.default");

        return callMicrosoftGraphMeEndpoint(oboAccessToken);
    }

}
```

# <a name="pythontabpython"></a>[Python](#tab/python)

Una API web de Python deberá usar algún middleware para validar el token de portador recibido del cliente. A continuación, la API web podrá obtener el token de acceso para la API de nivel inferior mediante la biblioteca de MSAL para Python al llamar al método [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of). Aún no hay disponible ningún ejemplo de este flujo con MSAL Python.

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API web que llama a las API web: llamada a una API](scenario-web-api-call-api-call-api.md)
