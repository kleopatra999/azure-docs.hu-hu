---
title: "Windows-hitelesítés és Azure MFA-kiszolgáló | Microsoft Docs"
description: "Ez az Azure Multi-Factor Authentication-oldal segítséget nyújt a Windows-hitelesítés és az Azure Multi-Factor Authentication-kiszolgáló telepítéséhez."
services: multi-factor-authentication
documentationcenter: 
author: kgremban
manager: femila
ms.assetid: 19a4043f-c4ce-43c0-80e7-2548ee92cb74
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 06/06/2017
ms.author: kgremban
ms.reviewer: yossib
ms.custom: it-pro
ms.translationtype: Human Translation
ms.sourcegitcommit: e66b606433f8924a509f2d04dae67ff00ded6dca
ms.openlocfilehash: e3ad33dd12b4fb831ba43738bc5af4e5561a0682
ms.contentlocale: hu-hu
ms.lasthandoff: 02/07/2017

---
# <a name="windows-authentication-and-azure-multi-factor-authentication-server"></a>Windows-hitelesítés és Azure Multi-Factor Authentication-kiszolgáló
Az Azure Multi-Factor Authentication-kiszolgáló Windows-hitelesítés szakaszának segítségével engedélyezheti és konfigurálhatja a Windows-hitelesítést alkalmazásokhoz. A Windows-hitelesítés beállítása előtt tartsa szem előtt az alábbi listát:

* Beállítás után indítsa újra a terminálszolgáltatások Azure Multi-Factor Authentication szolgáltatását a beállítás életbe léptetéséhez.
* Ha az „Azure Multi-Factor Authentication felhasználói egyeztetés megkövetelése” lehetőség be van jelölve, és Ön nem szerepel a felhasználói listán, az újraindítás után nem fog tudni bejelentkezni a számítógépre.
* A Megbízható IP-címek attól függnek, hogy az alkalmazás képes-e biztosítani az ügyfél IP-címének hitelesítését. Jelenleg csak a Terminálszolgáltatások támogatott.  

> [!NOTE]
> Ez a szolgáltatás nem támogatott a Terminálszolgáltatások védelmének biztosítására Windows Server 2012 R2-n.

## <a name="to-secure-an-application-with-windows-authentication-use-the-following-procedure"></a>Ha egy alkalmazás védelmét Windows-hitelesítéssel szeretné biztosítani, kövesse az alábbi eljárást.
1. Az Azure Multi-Factor Authentication-kiszolgálón kattintson a Windows-hitelesítés ikonra.
   ![Windows-hitelesítés](./media/multi-factor-authentication-get-started-server-windows/windowsauth.png)
2. Jelölje be a **Windows-hitelesítés engedélyezése** jelölőnégyzetet. Alapértelmezés szerint a jelölőnégyzet nincs bejelölve.
3. Az Alkalmazások lapon a rendszergazda konfigurálhatja egy vagy több alkalmazás esetében a Windows-hitelesítést.
4. Kiszolgáló vagy alkalmazás kiválasztása – meghatározza, hogy a kiszolgáló/alkalmazás engedélyezve van-e. Kattintson az **OK** gombra.
5. Kattintson a **Hozzáadás…** gombra.
6. A Megbízható IP-címek lapon beállíthatja, hogy a rendszer kihagyja az Azure Multi-Factor Authenticationt adott IP-címekről származó Windows-munkamenetek esetén. Ha például az alkalmazottak az alkalmazást az irodából és otthonról használják, dönthet úgy, hogy nem szeretné, ha a telefonjaik folyamatosan csörögnének az Azure Multi-Factor Authentication miatt az irodában. Ehhez az irodai alhálózatot Megbízható IP-címek bejegyzésként kell megadni.
7. Kattintson a **Hozzáadás…** gombra.
8. Válassza az **Egyetlen IP**-cím lehetőséget, ha egyetlen IP-címet szeretne kihagyni.
9. Válassza az **IP-címtartomány** lehetőséget, ha egy teljes IP-címtartományt szeretne kihagyni. Példa: 10.63.193.1-10.63.193.100.
10. Válassza az **Alhálózat** lehetőséget, ha egy IP-címtartományt szeretne megadni alhálózat megjelöléssel. Adja meg az alhálózat kezdő IP-címét, és válassza ki a megfelelő hálózati maszkot a legördülő listából.
11. Kattintson az **OK** gombra.

## <a name="next-steps"></a>Következő lépések

- [Külső felektől származó VPN-készülékek konfigurálása Azure MFA-kiszolgálóhoz](multi-factor-authentication-advanced-vpn-configurations.md)

- [Meglévő hitelesítési infrastruktúrájának kiegészítése az Azure MFA NPS bővítményével](multi-factor-authentication-nps-extension.md)

