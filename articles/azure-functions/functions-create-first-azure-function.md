---
title: "Az első függvény létrehozása az Azure portálon | Microsoft Docs"
description: "Ismerje meg, hogyan hozhatja létre az első Azure-függvényét kiszolgáló nélküli végrehajtáshoz az Azure Portalon."
services: functions
documentationcenter: na
author: ggailey777
manager: erikre
editor: 
tags: 
ms.assetid: 96cf87b9-8db6-41a8-863a-abb828e3d06d
ms.service: functions
ms.devlang: multiple
ms.topic: hero-article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 08/07/2017
ms.author: glenga
ms.custom: mvc
ms.translationtype: HT
ms.sourcegitcommit: 1dbb1d5aae55a4c926b9d8632b416a740a375684
ms.openlocfilehash: d7d3aef1e713cf080b23575f56321e00a0b4af1f
ms.contentlocale: hu-hu
ms.lasthandoff: 08/07/2017

---
# <a name="create-your-first-function-in-the-azure-portal"></a>Az első függvény létrehozása az Azure portálon

Az Azure Functions lehetővé teszi a kód végrehajtását kiszolgáló nélküli környezetben anélkül, hogy először létre kellene hoznia egy virtuális gépet vagy közzé kellene tennie egy webalkalmazást. Ebben a témakörben megismerheti, hogyan használhatja a Functions szolgáltatást egy „hello world” függvény létrehozására az Azure Portalon.

![Függvényalkalmazás létrehozása az Azure Portalon](./media/functions-create-first-azure-function/function-app-in-portal-editor.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="log-in-to-azure"></a>Jelentkezzen be az Azure-ba

Jelentkezzen be az [Azure portálra](https://portal.azure.com/).

## <a name="create-a-function-app"></a>Függvényalkalmazás létrehozása

Rendelkeznie kell egy függvényalkalmazással a függvények végrehajtásának biztosításához. A függvényalkalmazás lehetővé teszi, hogy logikai egységbe csoportosítsa a függvényeket az erőforrások egyszerűbb felügyelete, telepítése és megosztása érdekében. 

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

![A függvényalkalmazás elkészült.](./media/functions-create-first-azure-function/function-app-create-success.png)

[!INCLUDE [functions-portal-favorite-function-apps](../../includes/functions-portal-favorite-function-apps.md)]

Ezután létrehozhat egy függvényt az új függvényalkalmazásban.

## <a name="create-function"></a>HTTP által aktivált függvény létrehozása

1. Bontsa ki az új függvényalkalmazást, majd kattintson a **Függvények** elem melletti **+** gombra.

2.  A **Gyors kezdés** lapon kattintson a **WebHook + API** elemre, **válasszon egy nyelvet** a függvény számára, majd kattintson a **Függvény létrehozása** elemre. 
   
    ![A függvények gyors létrehozása az Azure Portalon.](./media/functions-create-first-azure-function/function-app-quickstart-node-webhook.png)

A rendszer létrehoz egy függvényt a választott nyelven a HTTP által indított függvények sablonjának használatával. Az új függvényt egy HTTP-kérelem küldésével futtathatja.

## <a name="test-the-function"></a>A függvény tesztelése

1. Az új függvényben kattintson a **< /> Függvény URL-címének beolvasása** elemre, válassza a **default (Function key)** (alapértelmezett (funkcióbillentyű)) lehetőséget, majd kattintson a **Másolás** gombra. 

    ![A függvény URL-címének másolása az Azure portálról](./media/functions-create-first-azure-function/function-app-develop-tab-testing.png)

2. Illessze be a függvény URL-címét a böngésző címsorába. Az URL-címhez fűzze hozzá a `&name=<yourname>` lekérdezési karakterláncot, majd nyomja le az `Enter` billentyűt a billentyűzeten a kérés végrehajtásához. A következőkben egy példa látható az Edge böngészőben a függvény által visszaadott válaszra:

    ![A függvény által visszaadott válasz a böngészőben.](./media/functions-create-first-azure-function/function-app-browser-testing.png)

    A kérelem URL-címe alapértelmezés szerint tartalmazza a függvény HTTP protokollon keresztüli eléréséhez szükséges kulcsot.   

3. A függvény futásakor a rendszer nyomkövetési adatok ír a naplókba. Az előző végrehajtás nyomkövetési adatainak megtekintéséhez térjen vissza a függvényhez a portálon, és a **Naplók** elem kibontásához kattintson a képernyő alján található felfelé mutató nyílra. 

   ![A függvények naplómegtekintője az Azure Portalon.](./media/functions-create-first-azure-function/function-view-logs.png)

## <a name="clean-up-resources"></a>Az erőforrások eltávolítása

[!INCLUDE [Clean up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Következő lépések

Létrehozott egy függvényalkalmazást és egy HTTP által aktivált egyszerű függvényt.  

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]

További információ: [Azure Functions – HTTP- és webhookkötések](functions-bindings-http-webhook.md).




