---
title: "Számítógép csatlakoztatása Azure-beli virtuális hálózathoz pont–hely kapcsolat és tanúsítványalapú hitelesítés használatával: PowerShell | Microsoft Docs"
description: "A tanúsítványalapú hitelesítés használatával biztonságosan csatlakoztathat egy számítógépet a virtuális hálózatához pont–hely VPN Gateway-kapcsolat létrehozásával. Ez a cikk a Resource Manager-alapú üzemi modellre vonatkozik, és a PowerShellt használja."
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 3eddadf6-2e96-48c4-87c6-52a146faeec6
ms.service: vpn-gateway
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/10/2017
ms.author: cherylmc
ms.translationtype: HT
ms.sourcegitcommit: 25e4506cc2331ee016b8b365c2e1677424cf4992
ms.openlocfilehash: 2e072ada13b8c742fe7f2e14737c9376f7677906
ms.contentlocale: hu-hu
ms.lasthandoff: 08/24/2017

---
# <a name="configure-a-point-to-site-connection-to-a-vnet-using-certificate-authentication-powershell"></a>Pont–hely kapcsolat konfigurálása virtuális hálózathoz tanúsítványalapú hitelesítés használatával: PowerShell

Ebből a cikkből megtudhatja, hogyan hozhat létre pont–hely kapcsolattal rendelkező virtuális hálózatot a Resource Manager-alapú üzemi modellben a PowerShell használatával. Ez a konfiguráció tanúsítványokat használ a kapcsolódó ügyfelek hitelesítéséhez. Ezt a konfigurációt más üzembehelyezési eszközzel vagy üzemi modellel is létrehozhatja, ha egy másik lehetőséget választ az alábbi listáról:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-point-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-howto-point-to-site-rm-ps.md)
> * [(Klasszikus) Azure Portal](vpn-gateway-howto-point-to-site-classic-azure-portal.md)
>
>

A pont–hely (P2S) VPN-átjáró lehetővé teszi biztonságos kapcsolat létesítését a virtuális hálózattal egy különálló ügyfélszámítógépről. A pont–hely VPN-kapcsolat akkor hasznos, ha távoli helyről szeretne csatlakozni a virtuális hálózathoz, például otthonról vagy egy konferenciáról. A pont–hely VPN emellett akkor is hasznos megoldás lehet a helyek közötti VPN helyett, ha csak néhány ügyfelet szeretne egy VNetre csatlakoztatni.

A pont–hely kapcsolat a Secure Sockets Tunneling Protocol (SSTP) protokollt használja, amely egy SSL-alapú VPN-protokoll. A pont–hely VPN-kapcsolat létesítéséhez a kapcsolatot az ügyfélszámítógépről kell elindítani.

![Számítógép csatlakoztatása Azure-beli virtuális hálózathoz – pont-hely kapcsolati diagram](./media/vpn-gateway-howto-point-to-site-rm-ps/point-to-site-diagram.png)

A pont–hely tanúsítvánnyal hitelesített kapcsolatokhoz a következőkre van szükség:

* Útvonalalapú VPN-átjáró.
* A nyilvános kulcs (.cer fájl) egy főtanúsítványhoz, amely az Azure-ba van feltöltve. A tanúsítványt a feltöltését követően megbízható tanúsítványnak tekinti a rendszer, és ezt használja hitelesítéshez.
* Egy ügyféltanúsítvány, amely a főtanúsítványból jött létre, és települt a virtuális hálózathoz csatlakozó egyes ügyfélszámítógépekre. A rendszer ezt a tanúsítványt használja ügyfélhitelesítéshez.
* A VPN-ügyfél konfigurációs csomagja. A VPN-ügyfélkonfigurációs csomag azokat az adatokat tartalmazza, amelyekre az ügyfélnek szüksége van a virtuális hálózathoz való csatlakozáshoz. A csomag konfigurálja a Windows operációs rendszer meglévő, natív VPN-ügyfelét. Minden csatlakozó ügyfelet a konfigurációs csomaggal kell konfigurálni.

A pont–hely kapcsolatok nem igényelnek VPN-eszközt vagy helyszíni nyilvános IP-címet. A VPN-kapcsolat kiépítése SSTP (Secure Socket Tunneling Protocol) használatával történik. A kiszolgálói oldalon az SSTP 1.0, 1.1 és 1.2 verziója támogatott. Az ügyfél dönti el, hogy melyik verziót használja. Windows 8.1 és újabb kiadások esetén az SSTP alapértelmezés szerint az 1.2 verziót használja. 

A Pont–hely kapcsolatokról további információt a cikk végén, a [Pont–hely kapcsolatok – gyakori kérdések](#faq) című részben talál.

## <a name="before-beginning"></a>Mielőtt hozzálát

* Győződjön meg arról, hogy rendelkezik Azure-előfizetéssel. Ha még nincs Azure-előfizetése, aktiválhatja [MSDN-előfizetői előnyeit](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details), vagy regisztrálhat egy [ingyenes fiókot](https://azure.microsoft.com/pricing/free-trial).
* Telepítse az Azure Resource Manager PowerShell-parancsmagjainak legújabb verzióját. A PowerShell-parancsmagok telepítéséről további információt a [How to install and configure Azure PowerShell](/powershell/azure/overview) (Az Azure PowerShell telepítése és konfigurálása) című témakörben talál.

### <a name="example"></a>Példaértékek

A példaértékek használatával létrehozhat egy tesztkörnyezetet, vagy a segítségükkel értelmezheti a cikkben szereplő példákat. A változókat a cikk [1](#declare). szakaszában állítjuk be. Megteheti, hogy lépésről lépésre végighalad az eljáráson, és módosítás nélkül ezeket az értékeket használja, de módosíthatja is őket, hogy megfeleljenek a saját környezetének. 

* **Név: VNet1**
* **Címtartomány: 192.168.0.0/16** és **10.254.0.0/16**<br>Ez a példa egynél több címtartományt használ annak szemléltetésére, hogy ez a konfiguráció több címtartománnyal is működik. Azonban nem kötelező több címtartományt megadni ehhez a konfigurációhoz.
* **Alhálózat neve: FrontEnd**
  * **Alhálózati címtartomány: 192.168.1.0/24**
* **Alhálózat neve: BackEnd**
  * **Alhálózati címtartomány: 10.254.1.0/24**
* **Alhálózat neve: GatewaySubnet**<br>Ennek az alhálózatnak kötelező a *GatewaySubnet* nevet adni, ellenkező esetben nem működik a VPN-átjáró.
  * **Átjáró-alhálózat címtartománya: 192.168.200.0/24** 
* **VPN-ügyfelek címkészlete: 172.16.201.0/24**<br>Azok a VPN-ügyfelek, amelyek ezzel a pont–hely kapcsolattal csatlakoznak a virtuális hálózathoz, a VPN-ügyfél címkészletből kapnak IP-címet.
* **Előfizetés:** Ha több előfizetése is van, ellenőrizze, hogy a megfelelőt használja-e.
* **Erőforráscsoport: TestRG**
* **Hely: East US**
* DNS-kiszolgáló: Annak a DNS-kiszolgálónak az **IP-címe**, amelyet névfeloldásra kíván használni.
* **Átjáró neve: Vnet1GW**
* **Nyilvános IP-név: VNet1GWPIP**
* **VPN típusa: RouteBased** 

## <a name="declare"></a>1. Bejelentkezés és a változók beállítása

Ez a szakasz a bejelentkezést és a konfigurációban használt értékek deklarálását ismerteti. A minta parancsprogramok a deklarált értékeket használják. Módosítsa az értékeket úgy, hogy megfeleljenek a saját környezetének. Azt is megteheti, hogy a deklarált értékeket használja, és gyakorlásként halad végig a lépéseken.

1. Nyissa meg emelt a PowerShell-konzolt szintű jogosultságokkal, és jelentkezzen be az Azure-fiókjába. Ez a parancsmag bejelentkezési hitelesítő adatokat kér be. A bejelentkezés után letölti a fiók beállításait, hogy elérhetők legyenek az Azure PowerShell számára.

  ```powershell
  Login-AzureRmAccount
  ```
2. Olvassa be az Azure-előfizetések listáját.

  ```powershell
  Get-AzureRmSubscription
  ```
3. Válassza ki a használni kívánt előfizetést.

  ```powershell
  Select-AzureRmSubscription -SubscriptionName "Name of subscription"
  ```
4. Deklarálja a használni kívánt változókat. Használja a következő példát, és szükség szerint cserélje le az értékeket a sajátjaira.

  ```powershell
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $BESubName = "Backend"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "192.168.0.0/16"
  $VNetPrefix2 = "10.254.0.0/16"
  $FESubPrefix = "192.168.1.0/24"
  $BESubPrefix = "10.254.1.0/24"
  $GWSubPrefix = "192.168.200.0/26"
  $VPNClientAddressPool = "172.16.201.0/24"
  $RG = "TestRG"
  $Location = "East US"
  $DNS = "10.1.1.3"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWPIP"
  $GWIPconfName = "gwipconf"
  ```

## <a name="ConfigureVNet"></a>2. Virtuális hálózat konfigurálása

1. Hozzon létre egy erőforráscsoportot.

  ```powershell
  New-AzureRmResourceGroup -Name $RG -Location $Location
  ```
2. Hozza létre a virtuális hálózat alhálózatainak konfigurációit, névként a következő értékeket adja meg: *FrontEnd*, *BackEnd*, illetve *GatewaySubnet*. Ezek az előtagok a deklarált virtuális hálózati címtér részei kell, hogy legyenek.

  ```powershell
  $fesub = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
  $besub = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName -AddressPrefix $BESubPrefix
  $gwsub = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix
  ```
3. Hozza létre a virtuális hálózatot.

  Ebben a példában a DNS-kiszolgálót nem kötelező megadni. Az érték megadásával nem jön létre új DNS-kiszolgáló. A megadott DNS-kiszolgáló IP-címének olyan DNS-kiszolgálónak kell lennie, amely fel tudja oldani azoknak az erőforrásoknak a nevét, amelyekkel Ön kapcsolatot fog létesíteni. Ebben a példában egy magánhálózati IP-címet használtunk, de ez valószínűleg nem az Ön DNS-kiszolgálójának IP-címe. Ügyeljen arra, hogy a saját értékeit használja.

  ```powershell
  New-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG -Location $Location -AddressPrefix $VNetPrefix1,$VNetPrefix2 -Subnet $fesub, $besub, $gwsub -DnsServer $DNS
  ```
4. Adja meg a most létrehozott virtuális hálózat változóit.

  ```powershell
  $vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG
  $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
  ```
5. Egy VPN Gateway-nek rendelkeznie kell nyilvános IP-címmel. Először az IP-cím típusú erőforrást kell kérnie, majd hivatkoznia kell arra, amikor létrehozza a virtuális hálózati átjárót. Az IP-címet a rendszer dinamikusan rendeli hozzá az erőforráshoz a VPN Gateway létrehozásakor. A VPN Gateway jelenleg csak a *Dinamikus* nyilvános IP-cím lefoglalását támogatja. Nem kérheti statikus IP-cím hozzárendelését. Ez azonban nem jelenti azt, hogy az IP-cím módosul a VPN Gatewayhez való hozzárendelése után. A nyilvános IP-cím kizárólag abban az esetben változik, ha az átjárót törli, majd újra létrehozza. Nem módosul átméretezés, alaphelyzetbe állítás, illetve a VPN Gateway belső karbantartása/frissítése során.

  Kérjen egy dinamikusan hozzárendelt nyilvános IP-címet.

  ```powershell
  $pip = New-AzureRmPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
  $ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
  ```

## <a name="creategateway"></a>3. A VPN-átjáró létrehozása

Konfigurálja és hozza létre a virtuális hálózati átjárót a virtuális hálózat számára.

* A *-GatewayType* csak **Vpn** lehet, a *-VpnType* pedig csak **RouteBased** lehet.
* Egy VPN-átjáró létrehozása akár 45 percet is igénybe vehet a kiválasztott [átjáró termékváltozatától](vpn-gateway-about-vpn-gateway-settings.md) függően.

```powershell
New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
-VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 `
```

## <a name="addresspool"></a>4. A VPN-ügyfélcímkészlet hozzáadása

Miután befejeződött a VPN-átjáró létrehozása, hozzáadhatja a VPN-ügyfélcímkészletet. A VPN-ügyfélcímkészlet az a tartomány, amelyből a VPN-ügyfelek IP-címet kapnak csatlakozáskor. Olyan magánhálózati IP-címtartományt használjon, amely nincs átfedésben azzal a helyszíni hellyel, amelyről csatlakozik, vagy azzal a virtuális hálózattal, amelyhez csatlakozik. Ebben a példában a VPN-ügyfélcímkészlet [változóként](#declare) lett deklarálva az 1. lépésben.

```powershell
$Gateway = Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG -Name $GWName
Set-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
```

## <a name="Certificates"></a>5. Tanúsítványok előállítása

A tanúsítványokat az Azure használja a VPN-ügyfelek hitelesítésére a pont–hely VPN-kapcsolatokban. A főtanúsítvány nyilvánoskulcs-adatait feltölti az Azure-ba. A nyilvános kulcs ezután „megbízhatónak” tekinthető. Az ügyféltanúsítványokat a megbízható főtanúsítványból kell létrehozni, majd telepíteni kell az összes számítógépen a Certificates-Current User/Personal tanúsítványtárolóban. A tanúsítványt a rendszer az ügyfél hitelesítésére használja, amikor az a virtuális hálózathoz próbál csatlakozni. 

Önaláírt tanúsítványok használata esetén azokat megadott paraméterekkel kell létrehozni. Önaláírt tanúsítványt a [PowerShell és Windows 10](vpn-gateway-certificates-point-to-site.md), vagy ha nem rendelkezik Windows 10 rendszerrel, a [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) című cikkekben leírt utasítások alapján hozhat létre. Fontos, hogy az önaláírt legfelső szintű tanúsítványok és az ügyféltanúsítványok generálása során lépésről lépésre betartsa ezeket az utasításokat. Ellenkező esetben a generált tanúsítványok nem lesznek kompatibilisek a P2S-kapcsolatokkal, és hibaüzenetet eredményeznek kapcsolódáskor.

### <a name="cer"></a>1. A .cer fájl beszerzése a főtanúsítványhoz

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-rootcert-include.md)]


### <a name="generate"></a>2. Ügyféltanúsítvány létrehozása

[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-p2s-clientcert-include.md)]

## <a name="upload"></a>6. A főtanúsítvány nyilvánoskulcs-adatainak feltöltése

Ellenőrizze, hogy a VPN-átjáró létrehozása befejeződött-e. Ha befejeződött, töltse fel a megbízható főtanúsítványhoz tartozó .cer fájlt (amely a nyilvános kulcsot tartalmazza) az Azure-ba. Miután feltöltötte a .cer fájlt, az Azure felhasználhatja azt azon ügyfelek hitelesítéséhez, amelyeken telepítve lett egy, a megbízható főtanúsítványból létrehozott ügyféltanúsítvány. Szükség szerint később további megbízhatófőtanúsítvány-fájlokat is feltölthet (legfeljebb 20-at).

1. Deklarálja a tanúsítványnév változóját, és cserélje le az értékeket a saját értékeire.

  ```powershell
  $P2SRootCertName = "P2SRootCert.cer"
  ```
2. Helyettesítse a fájl elérési útját a sajátjával, majd futtassa a parancsmagokat.

  ```powershell
  $filePathForCert = "C:\cert\P2SRootCert.cer"
  $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
  $CertBase64 = [system.convert]::ToBase64String($cert.RawData)
  $p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64
  ```
3. Töltse fel a nyilvánoskulcs-adatokat az Azure-ba. A tanúsítványadatok feltöltését követően az Azure megbízható főtanúsítványnak tekinti ezt.

   ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $CertBase64
  ```

## <a name="clientconfig"></a>7. A VPN-ügyfél konfigurációs csomagjának letöltése

Ha pont–hely típusú VPN-kapcsolattal szeretne kapcsolódni egy virtuális hálózathoz, minden ügyfélen telepíteni kell egy VPN-ügyfélkonfigurációs csomagot, amely konfigurálja a natív VPN-ügyfelet a virtuális hálózathoz való csatlakozáshoz szükséges beállításokkal és fájlokkal. A VPN-ügyfélkonfigurációs csomag a natív Windows VPN-ügyfelet konfigurálja, nem telepít új vagy más VPN-ügyfelet. 

Használhatja a VPN-ügyfél azonos konfigurációs csomagját minden ügyfélszámítógépen, feltéve, hogy a verzió megfelel az ügyfél architektúrájának. A támogatott ügyfél operációs rendszerek listáját a cikk végén, a [Pont–hely kapcsolatok – gyakori kérdések](#faq) című szakaszban tekintheti meg.

1. Az átjáró létrehozása után előállíthatja és letöltheti az ügyfél-konfigurációs csomagot. Ebben a példában a 64 bites ügyfél csomag kerül letöltésre. Ha a 32 bites ügyfelet szeretné letölteni, akkor az ’AMD64’ sort cserélje le ’x86’-ra. A VPN-ügyfelet az Azure Portal használatával is letöltheti.

  ```powershell
  Get-AzureRmVpnClientPackage -ResourceGroupName $RG `
  -VirtualNetworkGatewayName $GWName -ProcessorArchitecture Amd64
  ```
2. A csomag letöltéséhez másolja és illessze be a visszaadott hivatkozást a böngészőbe, ügyelve arra, hogy eltávolítsa a hivatkozást körülvevő idézőjeleket. 
3. Töltse le és telepítse a csomagot az ügyfélszámítógépen. Ha megjelenik a SmartScreen egy előugró ablaka, kattintson a **További információ**, majd a **Futtatás mindenképpen** elemre. A csomagot mentheti is, így más ügyfélszámítógépekre is telepítheti.
4. Nyissa meg az ügyfélszámítógépen a **Hálózati beállítások** eszközt, és kattintson a **VPN** elemre. A VPN-kapcsolat megjeleníti annak a virtuális hálózatnak a nevét, amelyhez csatlakozott.

## <a name="clientcertificate"></a>8. Exportált ügyféltanúsítvány telepítése

Ha a tanúsítvány létrehozásához használttól eltérő ügyfélszámítógépről szeretne pont–hely kapcsolatot létesíteni, akkor telepítenie kell egy ügyféltanúsítványt. Az ügyféltanúsítvány telepítésekor szükség lesz az ügyféltanúsítvány exportálásakor létrehozott jelszóra. Ehhez általában annyit kell tennie, hogy duplán rákattint a tanúsítványra, és telepíti azt.

Győződjön meg arról, hogy az ügyféltanúsítványt .pfx fájlként exportálta a teljes tanúsítványlánccal együtt (ez az alapértelmezett beállítás). Egyéb esetben a főtanúsítvány adatai nem lesznek jelen az ügyfélszámítógépen, és az ügyfél nem fogja tudni megfelelően elvégezni a hitelesítést. További információkért lásd az [exportált ügyféltanúsítványok telepítését](vpn-gateway-certificates-point-to-site.md#install) ismertető cikket. 

## <a name="connect"></a>9. Csatlakozás az Azure szolgáltatáshoz

1. Csatlakozzon a virtuális hálózathoz. Ehhez navigáljon az ügyfélszámítógépen a VPN-kapcsolatokhoz, és keresse meg a létrehozott VPN-kapcsolatot. Ugyanaz a neve, mint a virtuális hálózatnak. Kattintson a **Connect** (Csatlakozás) gombra. Megjelenhet egy előugró üzenet, amely a tanúsítvány használatára utal. Kattintson a **Folytatás** gombra emelt szintű jogosultságok használatához. 
2. A csatlakozás megkezdéséhez a **Kapcsolat** állapotlapon kattintson a **Csatlakozás** gombra. Ha megjelenik a **Tanúsítvány kiválasztása** képernyő, ellenőrizze, hogy az a csatlakozáshoz használni kívánt ügyféltanúsítványt mutatja-e. Ha nem, kattintson a legördülő nyílra, válassza ki a helyes tanúsítványt, majd kattintson az **OK** gombra.

  ![A VPN-ügyfél az Azure-hoz csatlakozik](./media/vpn-gateway-howto-point-to-site-rm-ps/clientconnect.png)
3. A kapcsolat létrejött.

  ![A kapcsolat létrejött](./media/vpn-gateway-howto-point-to-site-rm-ps/connected.png)

#### <a name="troubleshooting-p2s-connections"></a>Pont–hely kapcsolatok hibaelhárítása

[!INCLUDE [client certificates](../../includes/vpn-gateway-certificates-verify-client-cert-include.md)]

## <a name="verify"></a>10. A kapcsolat ellenőrzése

1. Annak ellenőrzéséhez, hogy a VPN-kapcsolat aktív-e, nyisson meg egy rendszergazda jogú parancssort, és futtassa az *ipconfig/all* parancsot.
2. Tekintse meg az eredményeket. Figyelje meg, hogy a kapott IP-cím azok közül a címek közül való, amelyeket a pont–hely VPN-ügyfél konfigurációjának címkészletében megadott. Az eredmények az alábbi példában szereplőkhöz hasonlóak:

  ```
  PPP adapter VNet1:
      Connection-specific DNS Suffix .:
      Description.....................: VNet1
      Physical Address................:
      DHCP Enabled....................: No
      Autoconfiguration Enabled.......: Yes
      IPv4 Address....................: 172.16.201.3(Preferred)
      Subnet Mask.....................: 255.255.255.255
      Default Gateway.................:
      NetBIOS over Tcpip..............: Enabled
  ```

## <a name="connectVM"></a>Csatlakozás virtuális géphez

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-p2s-include.md)]

## <a name="addremovecert"></a>Főtanúsítvány hozzáadása vagy eltávolítása

A megbízható főtanúsítványokat felveheti vagy el is távolíthatja az Azure-ban. Főtanúsítvány eltávolításakor a főtanúsítványból létrehozott tanúsítvánnyal rendelkező ügyfelek nem fognak tudni hitelesítést végezni, így csatlakozni sem. Ha azt szeretné, hogy az ügyfelek hitelesítést végezhessenek és csatlakozni tudjanak, telepítenie kell egy olyan új ügyféltanúsítványt, amelyet az Azure által megbízhatónak tartott (feltöltött) főtanúsítványból hoztak létre.

### <a name="addtrustedroot"></a>Megbízható főtanúsítvány hozzáadása

Az Azure-ba legfeljebb 20 főtanúsítványt tölthet fel .cer fájl formájában. A következő lépések segítségével adhat hozzá főtanúsítványt:

#### <a name="certmethod1"></a>1. módszer:

Ez a főtanúsítvány feltöltésének leghatékonyabb módszere.

1. Készítse elő a .cer fájlt feltöltésre:

  ```powershell
  $filePathForCert = "C:\cert\P2SRootCert3.cer"
  $cert = new-object System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)
  $CertBase64_3 = [system.convert]::ToBase64String($cert.RawData)
  $p2srootcert = New-AzureRmVpnClientRootCertificate -Name $P2SRootCertName -PublicCertData $CertBase64_3
  ```
2. Töltse fel a fájlt. Egyszerre csak egy fájlt tölthet fel.

  ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $CertBase64_3
  ```

3. A tanúsítványfájl sikeres feltöltésének ellenőrzése:

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

#### <a name="certmethod2"></a>2. módszer:

Ez a módszer több lépésből áll, mint az 1. módszer, de az eredménye ugyanaz. Arra az esetre szerepel itt, ha meg kellene tekintenie a tanúsítványadatokat.

1. Hozza létre és készítse elő az Azure-ba felvenni kívánt új főtanúsítványt. Exportálja a nyilvános kulcsot Base-64-kódolású X.509 (.CER) formátumban, és nyissa meg egy egyszerű szövegszerkesztőben. Másolja a példányokat az alábbi példában látható módon.

  ![tanúsítvány](./media/vpn-gateway-howto-point-to-site-rm-ps/copycert.png)

  > [!NOTE]
  > A tanúsítványadatok másolásakor a szöveget egy folyamatos sorként másolja kocsivissza vagy új sor nélkül. A kocsivisszák és az új sorok megjelenítéséhez lehet, hogy módosítania kell a nézetet a szövegszerkesztőben a „Szimbólum megjelenítése/Minden karakter megjelenítése” beállításra.
  >
  >

2. Adja meg a tanúsítvány nevét és a kulcs adatait egy változóként. Helyettesítse az adatokat a saját adataival az alábbi példában látható módon:

  ```powershell
  $P2SRootCertName2 = "ARMP2SRootCert2.cer"
  $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
  ```
3. Adja hozzá a megbízható főtanúsítványt. Egyszerre csak egy tanúsítványt adhat hozzá.

  ```powershell
  Add-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayname "VNet1GW" -ResourceGroupName "TestRG" -PublicCertData $MyP2SCertPubKeyBase64_2
  ```
4. A következő példa használatával ellenőrizheti, hogy helyesen ment-e végbe a tanúsítvány hozzáadása:

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

### <a name="removerootcert"></a>Főtanúsítvány eltávolítása

1. Deklarálja a változókat.

  ```powershell
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  $P2SRootCertName2 = "ARMP2SRootCert2.cer"
  $MyP2SCertPubKeyBase64_2 = "MIIC/zCCAeugAwIBAgIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAMBgxFjAUBgNVBAMTDU15UDJTUm9vdENlcnQwHhcNMTUxMjE5MDI1MTIxWhcNMzkxMjMxMjM1OTU5WjAYMRYwFAYDVQQDEw1NeVAyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyjIXoWy8xE/GF1OSIvUaA0bxBjZ1PJfcXkMWsHPzvhWc2esOKrVQtgFgDz4ggAnOUFEkFaszjiHdnXv3mjzE2SpmAVIZPf2/yPWqkoHwkmrp6BpOvNVOpKxaGPOuK8+dql1xcL0eCkt69g4lxy0FGRFkBcSIgVTViS9wjuuS7LPo5+OXgyFkAY3pSDiMzQCkRGNFgw5WGMHRDAiruDQF1ciLNojAQCsDdLnI3pDYsvRW73HZEhmOqRRnJQe6VekvBYKLvnKaxUTKhFIYwuymHBB96nMFdRUKCZIiWRIy8Hc8+sQEsAML2EItAjQv4+fqgYiFdSWqnQCPf/7IZbotgQIDAQABo00wSzBJBgNVHQEEQjBAgBAkuVrWvFsCJAdK5pb/eoCNoRowGDEWMBQGA1UEAxMNTXlQMlNSb290Q2VydIIQKazxzFjMkp9JRiX+tkTfSzAJBgUrDgMCHQUAA4IBAQA223veAZEIar9N12ubNH2+HwZASNzDVNqspkPKD97TXfKHlPlIcS43TaYkTz38eVrwI6E0yDk4jAuPaKnPuPYFRj9w540SvY6PdOUwDoEqpIcAVp+b4VYwxPL6oyEQ8wnOYuoAK1hhh20lCbo8h9mMy9ofU+RP6HJ7lTqupLfXdID/XevI8tW6Dm+C/wCeV3EmIlO9KUoblD/e24zlo3YzOtbyXwTIh34T0fO/zQvUuBqZMcIPfM1cDvqcqiEFLWvWKoAnxbzckye2uk1gHO52d8AVL3mGiX8wBJkjc/pMdxrEvvCzJkltBmqxTM6XjDJALuVh16qFlqgTWCIcb7ju"
  ```
2. Távolítsa el a tanúsítványt.

  ```powershell
  Remove-AzureRmVpnClientRootCertificate -VpnClientRootCertificateName $P2SRootCertName2 -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -PublicCertData $MyP2SCertPubKeyBase64_2
  ```
3. A következő példával meggyőződhet arról, hogy sikeresen megtörtént a tanúsítvány eltávolítása.

  ```powershell
  Get-AzureRmVpnClientRootCertificate -ResourceGroupName "TestRG" `
  -VirtualNetworkGatewayName "VNet1GW"
  ```

## <a name="revoke"></a>Ügyféltanúsítvány visszavonása

Az ügyféltanúsítványokat vissza lehet vonni. A visszavont tanúsítványok listájával az egyes ügyféltanúsítványok alapján, szelektíven tagadhatja meg a pont–hely kapcsolódás lehetőségét. Ez a folyamat eltér a megbízható főtanúsítvány eltávolításától. Ha töröl egy .cer formátumú megbízható főtanúsítványt az Azure-ból, azzal megvonja a hozzáférést minden olyan ügyféltanúsítványtól, amelyet a visszavont főtanúsítvánnyal hoztak létre/írtak alá. A főtanúsítvány helyett az ügyféltanúsítvány visszavonása esetén a főtanúsítványból létrehozott többi tanúsítvány továbbra is használható hitelesítésre.

A szokásos gyakorlat az, hogy a főtanúsítvánnyal kezelik a hozzáférést a munkacsoport vagy a szervezet szintjén, az egyes felhasználókra vonatkozó részletesebb szabályozást pedig visszavont ügyféltanúsítványokkal oldják meg.

### <a name="revokeclientcert"></a>Ügyféltanúsítvány visszavonása

1. Kérje le az ügyféltanúsítvány ujjlenyomatát. További információkat [a tanúsítványok ujjlenyomatának lekérését ismertető útmutatóban](https://msdn.microsoft.com/library/ms734695.aspx) találhat.
2. Másolja át az adatokat egy szövegszerkesztőbe, és távolítsa el az összes szóközt, hogy egy folyamatos karakterláncot kapjon. Ez a karakterlánc a következő lépésben változóként van deklarálva.
3. Deklarálja a változókat. Győződjön meg róla, hogy az előző lépésben lekért ujjlenyomatot deklarálja.

  ```powershell
  $RevokedClientCert1 = "NameofCertificate"
  $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  ```
4. Vegye fel az ujjlenyomatot a visszavont tanúsítványok listájára. Az ujjlenyomat hozzáadása után a „Sikeres” üzenet jelenik meg.

  ```powershell
  Add-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
  -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG `
  -Thumbprint $RevokedThumbprint1
  ```
5. Ellenőrizze, hogy az ujjlenyomat bekerült-e a visszavont tanúsítványok listájába.

  ```powershell
  Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
  ```
6. Az ujjlenyomat hozzáadását követően a tanúsítvány már nem használható csatlakozáshoz. Azok az ügyfelek, akik ezzel a tanúsítvánnyal próbálnak csatlakozni, egy üzenetet kapnak majd arról, hogy a tanúsítvány már nem érvényes.

### <a name="reinstateclientcert"></a>Ügyféltanúsítvány érvényességének visszaállítása

Vissza is állíthatja az ügyféltanúsítványok érvényességét. Ehhez törölni kell az ujjlenyomatukat a visszavont ügyféltanúsítványok listájából.

1. Deklarálja a változókat. Győződjön meg arról, hogy a megfelelő, a visszaállítani kívánt tanúsítványhoz tartozó ujjlenyomatot deklarálta.

  ```powershell
  $RevokedClientCert1 = "NameofCertificate"
  $RevokedThumbprint1 = "‎51ab1edd8da4cfed77e20061c5eb6d2ef2f778c7"
  $GWName = "Name_of_virtual_network_gateway"
  $RG = "Name_of_resource_group"
  ```
2. Törölje a tanúsítvány ujjlenyomatát a visszavont tanúsítványok listájából.

  ```powershell
  Remove-AzureRmVpnClientRevokedCertificate -VpnClientRevokedCertificateName $RevokedClientCert1 `
  -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG -Thumbprint $RevokedThumbprint1
  ```
3. Ellenőrizze, hogy megtörtént-e az ujjlenyomat eltávolítása a listából.

  ```powershell
  Get-AzureRmVpnClientRevokedCertificate -VirtualNetworkGatewayName $GWName -ResourceGroupName $RG
  ```

## <a name="faq"></a>Pont–hely kapcsolatok – gyakori kérdések

[!INCLUDE [Point-to-Site FAQ](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## <a name="next-steps"></a>Következő lépések
Miután a kapcsolat létrejött, hozzáadhat virtuális gépeket a virtuális hálózataihoz. További információkért lásd: [Virtuális gépek](https://docs.microsoft.com/azure/#pivot=services&panel=Compute). A hálózatok és virtuális gépek ismertetését lásd az [Azure- és Linux-alapú virtuálisgép-hálózatok áttekintésében](../virtual-machines/linux/azure-vm-network-overview.md).
