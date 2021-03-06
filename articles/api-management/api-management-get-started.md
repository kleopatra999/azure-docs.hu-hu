---
title: "Az első API kezelése az Azure API Management szolgáltatásban | Microsoft Docs"
description: "Megtudhatja, hogyan hozhat létre API-kat, adhat hozzá műveleteket és kezdheti meg az API Management használatát."
services: api-management
documentationcenter: 
author: steved0x
manager: erikre
editor: 
ms.assetid: 51b7df8b-1c43-43c6-90c9-0aa24f48206b
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 12/15/2016
ms.author: apimpm
translationtype: Human Translation
ms.sourcegitcommit: 30ec6f45da114b6c7bc081f8a2df46f037de61fd
ms.openlocfilehash: 6376faa50613288a797e2c0683a0890fa21879e1


---
# <a name="manage-your-first-api-in-azure-api-management"></a>Az első API kezelése az Azure API Management szolgáltatásban
## <a name="overview"> </a>Áttekintés
Ez az útmutató ismerteti, hogyan teheti meg gyorsan az első lépéseket az Azure API Management szolgáltatással, és hogyan indíthatja az első API-hívását.

## <a name="concepts"> </a>Mi az Azure API Management?
Az Azure API Management szolgáltatással bármely háttérrendszer alapján elindíthat egy teljes értékű API-programot.

A gyakori forgatókönyvek a következők:

* **A mobil infrastruktúra biztonságossá tétele** a hozzáférés API-kulcsokkal történő korlátozásával, a szolgáltatásmegtagadási támadások szabályozással történő megelőzésével vagy speciális biztonsági házirendek, például a JWT-jogkivonat alapú érvényesítés használatával.
* **Az ISV-k partneri rendszereinek engedélyezése** a fejlesztői portálon keresztüli gyors partnerfelvétellel és egy API-homlokzat építésével annak érdekében, hogy a belső megvalósításokról leválassza a partnerek általi használatra nem alkalmas megvalósításokat.
* **Belső API-program futtatása** egy központi hely felajánlásával, ahol a szervezet tagjai kommunikálhatnak az API-k rendelkezésre állásáról és a legújabb módosításaikról, illetve a hozzáférés szervezeti fiókok alapján történő korlátozásával, mindezt az API-átjáró és a háttérrendszer közötti biztonságos csatorna alapján.

A rendszer az alábbi összetevőkből áll:

* Az **API-átjáró** az a végpont, amely:
  
  * Fogadja az API-hívásokat, és továbbítja őket a háttérrendszerekre.
  * Ellenőrzi az API-kulcsokat, JWT-jogkivonatokat, tanúsítványokat és más hitelesítő adatokat.
  * Betartatja a használati kvótákat és a sebességkorlátokat.
  * Villámgyorsan, kódmódosítás nélkül átalakítja az API-kat.
  * Gyorsítótárazza a háttérrendszer válaszait, ahol ez be van állítva.
  * Elemzési céllal naplózza a hívások metaadatait.
* A **közzétevő portál** az a rendszergazdai felület, ahol beállíthatja az API-programot. A következőkre lehet használni:
  
  * API-séma meghatározása vagy importálása.
  * API-k termékekbe csomagolása.
  * Házirendek, például kvóták vagy átalakítások beállítása az API-kra.
  * Elemzések lekérése.
  * Felhasználók kezelése.
* A **fejlesztői portál** a fejlesztők fő webhelye, ahol a következőket tehetik:
  
  * Elolvashatják az API-dokumentációt.
  * API-kat próbálhatnak ki az interaktív konzollal.
  * Létrehozhatnak egy fiókot és előfizethetnek, hogy API-kulcsokat szerezzenek.
  * Hozzáférhetnek a használat adataikról készült elemzésekhez.

## <a name="create-service-instance"> </a>API Management-példány létrehozása
> [!NOTE]
> Az oktatóanyag elvégzéséhez egy Azure-fiókra lesz szüksége. Ha nincs fiókja, néhány perc alatt létrehozhat egy ingyenes fiókot. További információ: [Ingyenes Azure-fiók létrehozása][Azure Free Trial].
> 
> 

Az API Management használatának első lépése egy szolgáltatáspéldány létrehozása. Jelentkezzen be az [Azure Portalra][Azure Portal], és kattintson az **Új**, **Web + mobil**, **API Management** elemre.

![Új API Management-példány][api-management-create-instance-menu]

A **Név** mezőben adjon meg egy egyedi altartománynevet, amely a szolgáltatás URL-címe lesz.

Válassza ki a kívánt **Előfizetést**, **Erőforráscsoportot** és **Helyet** a szolgáltatáspéldányához.

Adja meg a **Contoso Ltd.** nevet a **Szervezet neve** mezőben, majd írja be az e-mail-címét a **Rendszergazda e-mail címe** mezőbe.

> [!NOTE]
> Erre az e-mail-címre fognak érkezni az API Management rendszer értesítései. További információkat a [How to configure notifications and email templates in Azure API Management][How to configure notifications and email templates in Azure API Management] (Az értesítések és e-mail sablonok konfigurálása az Azure API Management szolgáltatásban) című témakörben talál.
> 
> 

![Új API Management szolgáltatás][api-management-create-instance-step1]

Az API Management szolgáltatáspéldányok három csomagban érhetők el: Fejlesztői, Standard és Prémium.

> [!NOTE]
> A Fejlesztői csomag olyan API-programok fejlesztésére, tesztelésére, és próbaüzemére szolgál, amelyeknél nem fontos a magas rendelkezésre állás. A Standard és Prémium csomagokban a fenntartott egységet lehet úgy méretezni, hogy nagyobb forgalmat legyen képes kezelni. A Standard és a Prémium csomagok biztosítják a legnagyobb feldolgozási kapacitást és teljesítményt az API Management szolgáltatása számára. Jelen oktatóanyagot bármely csomaggal elvégezheti. További információt az API Management csomagjairól az [API Management Díjszabás][API Management pricing] oldalon talál.
> 
> 

Kattintson a **Létrehozás** elemre a szolgáltatáspéldány kiépítésének megkezdéséhez.

![Új API Management szolgáltatás][api-management-instance-created]

A szolgáltatáspéldány létrehozása után a következő lépés egy API létrehozása vagy importálása.

## <a name="create-api"> </a>API importálása
Az API egy ügyfélalkalmazásokból meghívható műveletkészletből áll. Az API-műveleteket létező webszolgáltatásokhoz használják proxyként.

Az API-kat létre lehet hozni (és a műveleteket hozzá lehet adni) manuálisan, vagy importálni is lehet. Ebben az oktatóanyagban importálni fogjuk egy Microsoft által biztosított, Azure-ban üzemeltetett minta számológép webszolgáltatás API-ját.

> [!NOTE]
> Útmutató az API-k létrehozásához és a műveletek manuális hozzáadásához: [API-k létrehozása](api-management-howto-create-apis.md) és [Műveletek hozzáadása API-khoz](api-management-howto-add-operations.md).
> 
> 

Az API-kat a közzétevő portálról lehet konfigurálni. Az eléréséhez kattintson a **Közzétevő portálra** a szolgáltatási eszköztárból.

![Közzétevő portál][api-management-management-console]

A számológép API importálásához kattintson a bal oldali **API Management** menü **API-k** elemére, majd kattintson az **API importálása** lehetőségre.

![API importálása gomb][api-management-import-api]

A számológép API konfigurálásához végezze el az alábbi lépéseket:

1. Kattintson az **URL-címről** lehetőségre, írja be a **http://calcapi.cloudapp.net/calcapi.json** címet a **Specifikációs dokumentum URL-címe** szövegmezőbe, majd kattintson a **Swagger** választógombra.
2. Írja be a **calc** kifejezést a **Webes API URL-címének utótagja** szövegmezőbe.
3. Kattintson a **Termékek (választható)** mezőre, és válassza a **Kezdő** lehetőséget.
4. Kattintson a **Mentés** gombra az API importálásához.

![Új API hozzáadása][api-management-import-new-api]

> [!NOTE]
> Az **API Management** jelenleg az 1.2-es és a 2.0-ás verziójú Swagger-dokumentumok importálását is támogatja. Ügyeljen arra, hogy még ha a [Swagger 2.0 specifikációja](http://swagger.io/specification) szerint a `host`, `basePath` és `schemes` tulajdonságok nem is kötelezőek, a Swagger 2.0-ás dokumentumnak akkor is tartalmaznia **KELL** ezeket a tulajdonságokat, máskülönben nem lesz importálva. 
> 
> 

Az API importálása után megjelenik az API összefoglaló lapja a közzétevő portálon.

![API összefoglaló][api-management-imported-api-summary]

Az API szakasz több lapból áll. Az **Összefoglalás** lap az API alapvető mérőszámait és információit jeleníti meg. A [Beállítások](api-management-howto-create-apis.md#configure-api-settings) lap az API-k konfigurációjának megtekintésére és szerkesztésére szolgál. A [Műveletek](api-management-howto-add-operations.md) lap az API műveleteinek kezelésére szolgál. A **Biztonság** lap a háttérkiszolgáló átjárójának egyszerű hitelesítés vagy [kölcsönös tanúsítványhitelesítés](api-management-howto-mutual-certificates.md) használatára való konfigurálására, illetve az [OAuth 2.0 segítségével történő felhasználói hitelesítés](api-management-howto-oauth2.md) konfigurálására szolgál.  A **Problémák** lap az API-kat használó fejlesztők által lejelentett problémák megtekintésére szolgál. A **Termékek** lap az API-t tartalmazó termékek konfigurálására szolgál.

Alapértelmezés szerint az API Management minden példányához az alábbi két mintatermék jár:

* **Kezdő**
* **Korlátlan**

Ebben az oktatóanyagban az API importálásakor a Kezdő termékhez adta hozzá az Egyszerű számológép API-t.

Egy API meghívásához a fejlesztőknek először elő kell fizetniük egy termékre, amely hozzáférést biztosít az API-hoz. A fejlesztők előfizethetnek a termékekre a fejlesztői portálon, vagy a rendszergazdák előfizethetnek a termékekre a fejlesztők nevében a közzétevő portálon. Mivel az oktatóanyag korábbi lépéseiben már létrehozott egy API Management-példányt, rendszergazdának számít, és alapértelmezés szerint minden termékre elő van fizetve.

## <a name="call-operation"> </a>Művelet meghívása a fejlesztői portálról
A műveleteket meg lehet hívni közvetlenül a fejlesztői portálról, ami egy kényelmes módot biztosít az API műveleteinek megtekintésére és tesztelésére. Az oktatóanyag jelen lépésében az Egyszerű számológép API **Két egész szám összeadása** műveletét fogja meghívni. Kattintson a **Fejlesztői portál** lehetőségre a közzétevő portál jobb felső részén látható menüben.

![Fejlesztői portál][api-management-developer-portal-menu]

Kattintson az **API-k** elemre a felső menüben, majd kattintson az **Egyszerű számológép** lehetőségre az elérhető műveletek megtekintéséhez.

![Fejlesztői portál][api-management-developer-portal-calc-api]

Figyelje meg az API-val és a műveletekkel együtt importált mintaleírásokat és paramétereket, amelyek dokumentációként szolgálnak a műveletet használni tervező fejlesztők számára. Ezeket a leírásokat a műveletek manuális hozzáadásakor is hozzá lehet adni.

A **Két egész szám összeadása** művelet meghívásához kattintson a **Kipróbálom** gombra.

![Kipróbálom][api-management-developer-portal-calc-api-console]

Írjon be néhány értéket a paraméterekhez, vagy fogadja el az alapértelmezett értékeket, majd kattintson a **Küldés** gombra.

![HTTP Get][api-management-invoke-get]

A művelet meghívása után a fejlesztői portál megjeleníti a **Válasz állapota**, a **Válasz fejlécei** és a **Válasz tartalma** minden információját.

![Válasz][api-management-invoke-get-response]

## <a name="view-analytics"> </a>Elemzés megtekintése
Az Egyszerű számológép elemzésének megtekintéséhez váltson vissza a közzétevő portálra a fejlesztői portál jobb felső részén látható menü **Kezelés** lehetőségének kiválasztásával.

![Kezelés][api-management-manage-menu]

A közzétevő portál alapértelmezett nézete az **Irányítópult**, amely az API Management példány áttekintését jeleníti meg.

![Irányítópult][api-management-dashboard]

Vigye az egérmutatót a **Egyszerű számológép** ábrájára az API adott időszakban történő használatára vonatkozó mérőszámok megtekintéséhez.

> [!NOTE]
> Ha nem lát vonalakat az ábrán, váltson vissza a fejlesztői portálra és, hívja meg néhányszor az API-t, várjon pár pillanatot, majd térjen vissza az irányítópultra.
> 
> 

Kattintson a **Részletek megtekintése** parancsra az API összefoglaló lapjának megtekintéséhez, ahol a megjelenített mérőszámok is megtalálhatók részletesebben.

![Elemzés][api-management-mouse-over]

![Összefoglalás][api-management-api-summary-metrics]

Részletes metrikákért és jelentésekért kattintson a bal oldali **API Management** menü **Elemzés** lehetőségére.

![Áttekintés][api-management-analytics-overview]

Az **Elemzés** szakasz az alábbi négy lapból áll:

* Az **Áttekintés** az általános használati és állapotmetrikákat, valamint a legnépszerűbb fejlesztőket, termékeket, API-kat és műveleteket jeleníti meg.
* A **Használat** részletes tájékoztatást biztosít az API-hívásokról és a sávszélességről, földrajzi ábrázolással.
* Az **Állapot** az állapotkódokkal, a gyorsítótárazás sikerességének mértékével, a válaszidőkkel, illetve az API-k és szolgáltatások válaszidejével foglalkozik.
* A **Tevékenység** jelentéseket biztosít, amelyek egy adott tevékenységet részleteznek fejlesztő, termék, API és művelet szerint.

## <a name="next-steps"> </a>Következő lépések
* Megtudhatja, hogyan [védheti meg az API-kat sebességkorlátozással](api-management-howto-product-with-rules.md).

[Azure Free Trial]: http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=api_management_hero_a

[Create an API Management instance]: #create-service-instance
[Create an API]: #create-api
[Add an operation]: #add-operation
[Add the new API to a product]: #add-api-to-product
[Subscribe to the product that contains the API]: #subscribe
[Call an operation from the Developer Portal]: #call-operation
[View analytics]: #view-analytics
[Next steps]: #next-steps


[How to manage developer accounts in Azure API Management]: api-management-howto-create-or-invite-developers.md
[Configure API settings]: api-management-howto-create-apis.md#configure-api-settings
[How to configure notifications and email templates in Azure API Management]: api-management-howto-configure-notifications.md
[Responses]: api-management-howto-add-operations.md#responses
[How create and publish a product]: api-management-howto-add-products.md
[API Management pricing]: http://azure.microsoft.com/pricing/details/api-management/

[Azure Portal]: https://portal.azure.com/

[api-management-management-console]: ./media/api-management-get-started/api-management-management-console.png
[api-management-create-instance-menu]: ./media/api-management-get-started/api-management-create-instance-menu.png
[api-management-create-instance-step1]: ./media/api-management-get-started/api-management-create-instance-step1.png
[api-management-create-instance-step2]: ./media/api-management-get-started/api-management-create-instance-step2.png
[api-management-instance-created]: ./media/api-management-get-started/api-management-instance-created.png
[api-management-import-api]: ./media/api-management-get-started/api-management-import-api.png
[api-management-import-new-api]: ./media/api-management-get-started/api-management-import-new-api.png
[api-management-imported-api-summary]: ./media/api-management-get-started/api-management-imported-api-summary.png
[api-management-calc-operations]: ./media/api-management-get-started/api-management-calc-operations.png
[api-management-list-products]: ./media/api-management-get-started/api-management-list-products.png
[api-management-add-api-to-product]: ./media/api-management-get-started/api-management-add-api-to-product.png
[api-management-add-myechoapi-to-product]: ./media/api-management-get-started/api-management-add-myechoapi-to-product.png
[api-management-api-added-to-product]: ./media/api-management-get-started/api-management-api-added-to-product.png
[api-management-developers]: ./media/api-management-get-started/api-management-developers.png
[api-management-add-subscription]: ./media/api-management-get-started/api-management-add-subscription.png
[api-management-add-subscription-window]: ./media/api-management-get-started/api-management-add-subscription-window.png
[api-management-subscription-added]: ./media/api-management-get-started/api-management-subscription-added.png
[api-management-developer-portal-menu]: ./media/api-management-get-started/api-management-developer-portal-menu.png
[api-management-developer-portal-calc-api]: ./media/api-management-get-started/api-management-developer-portal-calc-api.png
[api-management-developer-portal-calc-api-console]: ./media/api-management-get-started/api-management-developer-portal-calc-api-console.png
[api-management-invoke-get]: ./media/api-management-get-started/api-management-invoke-get.png
[api-management-invoke-get-response]: ./media/api-management-get-started/api-management-invoke-get-response.png
[api-management-manage-menu]: ./media/api-management-get-started/api-management-manage-menu.png
[api-management-dashboard]: ./media/api-management-get-started/api-management-dashboard.png

[api-management-add-response]: ./media/api-management-get-started/api-management-add-response.png
[api-management-add-response-window]: ./media/api-management-get-started/api-management-add-response-window.png
[api-management-developer-key]: ./media/api-management-get-started/api-management-developer-key.png
[api-management-mouse-over]: ./media/api-management-get-started/api-management-mouse-over.png
[api-management-api-summary-metrics]: ./media/api-management-get-started/api-management-api-summary-metrics.png
[api-management-analytics-overview]: ./media/api-management-get-started/api-management-analytics-overview.png
[api-management-analytics-usage]: ./media/api-management-get-started/api-management-analytics-usage.png
[api-management-]: ./media/api-management-get-started/api-management-.png
[api-management-]: ./media/api-management-get-started/api-management-.png



<!--HONumber=Dec16_HO3-->


