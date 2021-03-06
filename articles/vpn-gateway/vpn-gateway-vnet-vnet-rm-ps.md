---
title: "Azure-alapú virtuális hálózat csatlakoztatása másik VNethez: PowerShell | Microsoft Docs"
description: "Ez a cikk lépésről lépésre bemutatja, hogyan csatlakoztathatók a virtuális hálózatok egymáshoz az Azure Resource Manager és a PowerShell használatával."
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 0683c664-9c03-40a4-b198-a6529bf1ce8b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/02/2017
ms.author: cherylmc
ms.translationtype: HT
ms.sourcegitcommit: 79bebd10784ec74b4800e19576cbec253acf1be7
ms.openlocfilehash: 8c42c0046ccaa98c572134042fbbb7e883ef93c3
ms.contentlocale: hu-hu
ms.lasthandoff: 08/03/2017

---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-using-powershell"></a>Virtuális hálózatok közötti VPN Gateway-kapcsolat konfigurálása a PowerShell használatával

Ebből a cikkből megtudhatja, hogyan hozható létre VPN Gateway-kapcsolat virtuális hálózatok között. A virtuális hálózatok lehetnek azonos vagy eltérő régiókban, illetve azonos vagy eltérő előfizetésekben. Amikor különböző előfizetésekről csatlakoztat virtuális hálózatokat, az előfizetéseket nem kell társítani ugyanazzal az Active Directory-bérlővel. 

A cikkben ismertetett lépések a Resource Manager-alapú üzemi modellre vonatkoznak és a PowerShellt használják. Ezt a konfigurációt más üzembehelyezési eszközzel vagy üzemi modellel is létrehozhatja, ha egy másik lehetőséget választ az alábbi listáról:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [Azure CLI](vpn-gateway-howto-vnet-vnet-cli.md)
> * [(Klasszikus) Azure Portal](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Különböző üzemi modellek összekapcsolása – Azure Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Különböző üzemi modellek összekapcsolása – PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

Egy virtuális hálózat egy másikkal való összekapcsolása (a virtuális hálózatok közötti kapcsolat) nagyon hasonlít egy virtuális hálózat egy helyszíni helyhez való csatlakoztatásához. Mindkét kapcsolattípus egy VPN-átjárót használ a biztonságos alagút IPsec/IKE használatával való kialakításához. Ha a virtuális hálózatai azonos régióban találhatóak, fontolja meg az összekötésüket virtuális hálózatok közötti társviszony útján. A virtuális hálózatok közötti társviszony nem használ VPN-átjárót. További információ: [Társviszony létesítése virtuális hálózatok között](../virtual-network/virtual-network-peering-overview.md).

A virtuális hálózatok közötti kommunikáció kombinálható többhelyes konfigurációkkal. Így létrehozhat olyan hálózati topológiákat, amelyek a létesítmények közötti kapcsolatokat a virtuális hálózatok közötti kapcsolatokkal egyesítik, ahogyan azt a következő diagram mutatja:

![Tudnivalók a kapcsolatokról](./media/vpn-gateway-vnet-vnet-rm-ps/aboutconnections.png)

### <a name="why-connect-virtual-networks"></a>Miért érdemes összekapcsolni a virtuális hálózatokat?

A virtuális hálózatokat a következő okokból érdemes összekapcsolni:

* **Georedundancia és földrajzi jelenlét több régióban**

  * Beállíthatja a saját georeplikációját vagy szinkronizálását biztonságos kapcsolaton át, internetes végpontok használata nélkül.
  * Az Azure Traffic Manager és a Load Balancer segítségével magas rendelkezésre állású munkaterhelést állíthat be georedundanciával több Azure-régióban. Például beállíthat folyamatosan működő SQL-t több Azure-régióban található rendelkezésre állási csoportokkal.
* **Regionális többrétegű alkalmazások elkülönítéssel vagy felügyeleti határral**

  * Egy régión belül beállíthat többrétegű alkalmazásokat több, elkülönítéssel vagy felügyeleti követelményekkel összekapcsolt virtuális hálózatokkal.

A virtuális hálózatok közötti kapcsolatokról további információt a cikk végén, a [Virtuális hálózatok közötti kapcsolat – gyakori kérdések](#faq) című részben talál.

## <a name="which-set-of-steps-should-i-use"></a>Melyik eljárást használjam?

Ebben a cikkben kétféle lépéssorozatot láthat. Az egyik lépéssorozat az [azonos előfizetésben található virtuális hálózatokra](#samesub), a másik az [eltérő előfizetésekben található virtuális hálózatokra](#difsub) vonatkozik. A két sorozat közötti fő különbség az, hogy lehetséges-e egyetlen PowerShell-munkameneten belül konfigurálni az összes virtuális hálózati és átjáró-erőforrást.

A cikkben szereplő lépések az egyes szakaszok elején deklarált változókat használják. Ha már meglévő virtuális hálózatokat használ, módosítsa úgy a változókat, hogy azok a saját környezetének beállításait tükrözzék. Ha névfeloldással szeretné ellátni a virtuális hálózatokat, tekintse meg a [Névfeloldás](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md) című cikket.

## <a name="samesub"></a>Azonos előfizetésben található virtuális hálózatok összekapcsolása

![v2v ábra](./media/vpn-gateway-vnet-vnet-rm-ps/v2vrmps.png)

### <a name="before-you-begin"></a>Előkészületek

Mielőtt hozzálát, telepítenie kell az Azure Resource Manager PowerShell-parancsmagjainak legújabb verzióit, legalább a 4.0-s verziót. További információ a PowerShell-parancsmagok telepítéséről: [Az Azure PowerShell telepítése és konfigurálása](/powershell/azure/overview).

### <a name="Step1"></a>1. lépés – Az IP-címtartományok megtervezése

A következő lépések során két virtuális hálózatot fogunk létrehozni az azokhoz tartozó átjáróalhálózatokkal és konfigurációkkal. Ezután létrehozunk egy VPN-kapcsolatot a két virtuális hálózat között. Fontos, hogy megtervezze a hálózati konfiguráció IP-címtartományát. Ügyeljen arra, hogy a virtuális hálózata tartományai és a helyi hálózata tartományai ne legyenek átfedésben egymással. Ezekben a példákban nem szerepel DNS-kiszolgáló. Ha névfeloldással szeretné ellátni a virtuális hálózatokat, tekintse meg a [Névfeloldás](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md) című cikket.

A példákban a következő értékeket használjuk:

**Értékek a TestVNet1-hez:**

* Virtuális hálózat neve: TestVNet1
* Erőforráscsoport: TestRG1
* Hely: East US
* TestVNet1: 10.11.0.0/16 és 10.12.0.0/16
* Előtér: 10.11.0.0/24
* Háttér: 10.12.0.0/24
* Átjáróalhálózat: 10.12.255.0/27
* Átjáró neve: VNet1GW
* Nyilvános IP-cím: VNet1GWIP
* VPN típusa: RouteBased
* Kapcsolat (1–4): VNet1toVNet4
* Kapcsolat (1–5): VNet1toVNet5
* Kapcsolat típusa: VNet2VNet

**Értékek a TestVNet4-hez:**

* Virtuális hálózat neve: TestVNet4
* TestVNet2: 10.41.0.0/16 és 10.42.0.0/16
* Előtér: 10.41.0.0/24
* Háttér: 10.42.0.0/24
* Átjáróalhálózat: 10.42.255.0/27
* Erőforráscsoport: TestRG4
* Hely: West US
* Átjáró neve: VNet4GW
* Nyilvános IP-cím: VNet4GWIP
* VPN típusa: RouteBased
* Kapcsolat: VNet4toVNet1
* Kapcsolat típusa: VNet2VNet


### <a name="Step2"></a>2. lépés – A TestVNet1 létrehozása és konfigurálása

1. Deklarálja a változókat. Ez a példa a gyakorlathoz használt értékekkel deklarálja a változókat. A legtöbb esetben ezeket az értékeket a saját értékeire kell lecserélnie. Ezeket a változókat akkor használhatja, ha azért hajtja végre a lépéseket, hogy megismerje ezt a konfigurációtípust. Ha szükséges, módosítsa a változókat, majd másolja és illessze be őket a PowerShell-konzolra.

  ```powershell
  $Sub1 = "Replace_With_Your_Subcription_Name"
  $RG1 = "TestRG1"
  $Location1 = "East US"
  $VNetName1 = "TestVNet1"
  $FESubName1 = "FrontEnd"
  $BESubName1 = "Backend"
  $GWSubName1 = "GatewaySubnet"
  $VNetPrefix11 = "10.11.0.0/16"
  $VNetPrefix12 = "10.12.0.0/16"
  $FESubPrefix1 = "10.11.0.0/24"
  $BESubPrefix1 = "10.12.0.0/24"
  $GWSubPrefix1 = "10.12.255.0/27"
  $GWName1 = "VNet1GW"
  $GWIPName1 = "VNet1GWIP"
  $GWIPconfName1 = "gwipconf1"
  $Connection14 = "VNet1toVNet4"
  $Connection15 = "VNet1toVNet5"
  ```

2. Csatlakozás a fiókhoz. A következő példa segít a kapcsolódásban:

  ```powershell
  Login-AzureRmAccount
  ```

  Keresse meg a fiókot az előfizetésekben.

  ```powershell
  Get-AzureRmSubscription
  ```

  Válassza ki a használni kívánt előfizetést.

  ```powershell
  Select-AzureRmSubscription -SubscriptionName $Sub1
  ```
3. Hozzon létre egy új erőforráscsoportot.

  ```powershell
  New-AzureRmResourceGroup -Name $RG1 -Location $Location1
  ```
4. Hozza létre a TestVNet1 alhálózat-konfigurációit. Ez a példa létrehoz egy TestVNet1 nevű virtuális hálózatot és három alhálózatot, amelyek neve a következő: GatewaySubnet, FrontEnd és Backend. Az értékek behelyettesítésekor fontos, hogy az átjáróalhálózat neve mindenképp GatewaySubnet legyen. Ha ezt másként nevezi el, az átjáró létrehozása meghiúsul.

  A következő példa a korábban beállított változókat használja. A példában az átjáróalhálózat /27-es alhálózatot használ. Bár lehetséges akár /29-es átjáróalhálózatot is létrehozni, javasolt egy ennél nagyobb, több címmel rendelkező alhálózatot létrehozni: legalább /28-asat vagy /27-eset. Ez elegendő címet biztosít az esetleges későbbi konfigurációkhoz.

  ```powershell
  $fesub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
  $besub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
  $gwsub1 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1
  ```
5. Hozza létre a TestVNet1-et.

  ```powershell
  New-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 `
  -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
  ```
6. Kérje egy nyilvános IP-cím kiosztását a virtuális hálózat számára létrehozni kívánt átjáróhoz. Vegye figyelembe, hogy az AllocationMethod értéke Dynamic. A használni kívánt IP-címet nem adhatja meg. Ennek kiosztása az átjáró számára dinamikusan történik. 

  ```powershell
  $gwpip1 = New-AzureRmPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
  ```
7. Hozza létre az átjáró konfigurációját. Az átjáró konfigurációja meghatározza az alhálózatot és a használandó nyilvános IP-címet. A példa használatával hozza létre az átjáró konfigurációját.

  ```powershell
  $vnet1 = Get-AzureRmVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
  $subnet1 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```
8. Hozza létre a TestVNet1 átjáróját. Ebben a lépésben a TestVNet1 virtuális hálózati átjáróját fogja létrehozni. A virtuális hálózatok közötti konfigurációkhoz a VpnType paraméternek RouteBased értékűnek kell lennie. Az átjáró létrehozása akár 45 percet vagy hosszabb időt is igénybe vehet a választott átjáró-termékváltozattól függően.

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
  -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-3---create-and-configure-testvnet4"></a>3. lépés – A TestVNet4 létrehozása és konfigurálása

A TestVNet1 konfigurálása után a hozza létre a TestVNet4 virtuális hálózatot. Kövesse az alábbi lépéseket, az értékeket a saját értékeire cserélve. Ez a lépés elvégezhető ugyanebben a PowerShell-munkamenetben, mert a virtuális hálózat azonos előfizetésben található.

1. Deklarálja a változókat. Ne felejtse el az értékeket olyanokra cserélni, amelyeket a saját konfigurációjához kíván használni.

  ```powershell
  $RG4 = "TestRG4"
  $Location4 = "West US"
  $VnetName4 = "TestVNet4"
  $FESubName4 = "FrontEnd"
  $BESubName4 = "Backend"
  $GWSubName4 = "GatewaySubnet"
  $VnetPrefix41 = "10.41.0.0/16"
  $VnetPrefix42 = "10.42.0.0/16"
  $FESubPrefix4 = "10.41.0.0/24"
  $BESubPrefix4 = "10.42.0.0/24"
  $GWSubPrefix4 = "10.42.255.0/27"
  $GWName4 = "VNet4GW"
  $GWIPName4 = "VNet4GWIP"
  $GWIPconfName4 = "gwipconf4"
  $Connection41 = "VNet4toVNet1"
  ```
2. Hozzon létre egy új erőforráscsoportot.

  ```powershell
  New-AzureRmResourceGroup -Name $RG4 -Location $Location4
  ```
3. Hozza létre a TestVNet4 alhálózat-konfigurációit.

  ```powershell
  $fesub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName4 -AddressPrefix $FESubPrefix4
  $besub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName4 -AddressPrefix $BESubPrefix4
  $gwsub4 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName4 -AddressPrefix $GWSubPrefix4
  ```
4. Hozza létre a TestVNet4-et.

  ```powershell
  New-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4 `
  -Location $Location4 -AddressPrefix $VnetPrefix41,$VnetPrefix42 -Subnet $fesub4,$besub4,$gwsub4
  ```
5. Kérjen egy nyilvános IP-címet.

  ```powershell
  $gwpip4 = New-AzureRmPublicIpAddress -Name $GWIPName4 -ResourceGroupName $RG4 `
  -Location $Location4 -AllocationMethod Dynamic
  ```
6. Hozza létre az átjáró konfigurációját.

  ```powershell
  $vnet4 = Get-AzureRmVirtualNetwork -Name $VnetName4 -ResourceGroupName $RG4
  $subnet4 = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet4
  $gwipconf4 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName4 -Subnet $subnet4 -PublicIpAddress $gwpip4
  ```
7. A TestVNet4 átjárójának létrehozása Az átjáró létrehozása akár 45 percet vagy hosszabb időt is igénybe vehet a választott átjáró-termékváltozattól függően.

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4 `
  -Location $Location4 -IpConfigurations $gwipconf4 -GatewayType Vpn `
  -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-4---create-the-connections"></a>4. lépés – A kapcsolatok létrehozása

1. Hozza létre a virtuális hálózati átjárókat. Ha mindkét átjáró ugyanabban az előfizetésben szerepel, ahogy a példában is látható, a lépést ugyanabban a PowerShell-munkamenetben hajthatja végre.

  ```powershell
  $vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
  $vnet4gw = Get-AzureRmVirtualNetworkGateway -Name $GWName4 -ResourceGroupName $RG4
  ```
2. Hozza létre a TestVNet1–TestVNet4 kapcsolatot. Ebben a lépésben a TestVNet1 felől a TestVNet4 felé irányuló kapcsolatot hozza létre. A példák egy megosztott kulcsra is hivatkoznak. A megosztott kulcshoz használhatja a saját értékeit. Fontos, hogy a megosztott kulcs azonos legyen mindkét kapcsolathoz. A kapcsolat létrehozása egy kis időt vehet igénybe.

  ```powershell
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection14 -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet4gw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
3. Hozza létre a TestVNet4–TestVNet1 kapcsolatot. Ez a lépés az előzőhöz hasonló. A különbség annyi, hogy ezúttal a TestVNet4 felől a TestVNet1 felé irányuló kapcsolatot hozza létre. Ügyeljen arra, hogy a megosztott kulcsok megegyezzenek. A kapcsolat néhány perc alatt létrejön.

  ```powershell
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection41 -ResourceGroupName $RG4 `
  -VirtualNetworkGateway1 $vnet4gw -VirtualNetworkGateway2 $vnet1gw -Location $Location4 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
4. Ellenőrizze a kapcsolatot. Tekintse meg [A kapcsolat ellenőrzése](#verify) című szakaszt.

## <a name="difsub"></a>Különböző előfizetésben található virtuális hálózatok összekapcsolása

![v2v ábra](./media/vpn-gateway-vnet-vnet-rm-ps/v2vdiffsub.png)

Ebben a forgatókönyvben csatlakoztatjuk a TestVNet1 és a TestVNet5 virtuális hálózatot. A TestVNet1 és a TestVNet5 eltérő előfizetésekben található. Az előfizetéseket nem kell társítani ugyanazzal az Active Directory bérlővel. A jelen és korábbi lépések közötti különbség abban áll, hogy a konfigurációs lépések egy részét külön PowerShell-munkamenetben kell elvégezni a második előfizetés környezetében. Ez különösen akkor van így, ha a két előfizetés különböző szervezetekhez tartozik.

### <a name="step-5---create-and-configure-testvnet1"></a>5. lépés – A TestVNet1 létrehozása és konfigurálása

Az előző szakaszban található [1. lépés](#Step1) és a [2. lépés](#Step2) elvégzésével hozza létre és konfigurálja a TestVNet1-et, valamint a TestVNet1 VPN-átjáróját. Ehhez a konfigurációhoz nem kell létrehoznia az előző szakaszban a TestVNet4-et, azonban ha már megtette, az sem akadályozza az alábbi lépések végrehajtását. Miután elvégezte az 1. lépést és a 2. lépést, folytassa az 6. lépéssel a TestVNet5 létrehozásához. 

### <a name="step-6---verify-the-ip-address-ranges"></a>6. lépés – Az IP-címtartományok ellenőrzése

Fontos ügyelni arra, hogy az új virtuális hálózat (TestVNet5) IP-címtere ne legyen átfedésben a többi virtuális hálózati vagy hálózati átjárói tartománnyal. Ebben a példában a virtuális hálózatok különböző szervezetekhez is tartozhatnak. A gyakorlatban a TestVNet5 hálózathoz a következő értékeket használhatja:

**Értékek a TestVNet5-höz:**

* Virtuális hálózat neve: TestVNet5
* Erőforráscsoport: TestRG5
* Hely: Japan East
* TestVNet5: 10.51.0.0/16 és 10.52.0.0/16
* Előtér: 10.51.0.0/24
* Háttér: 10.52.0.0/24
* Átjáróalhálózat: 10.52.255.0.0/27
* Átjáró neve: VNet5GW
* Nyilvános IP-cím: VNet5GWIP
* VPN típusa: RouteBased
* Kapcsolat: VNet5toVNet1
* Kapcsolat típusa: VNet2VNet

### <a name="step-7---create-and-configure-testvnet5"></a>7. lépés – A TestVNet5 létrehozása és konfigurálása

Ezt a lépést az új előfizetés környezetében kell elvégezni. Ezt a részt azon másik szervezet rendszergazdájának kell elvégeznie, amely az előfizetés tulajdonosa.

1. Deklarálja a változókat. Ne felejtse el az értékeket olyanokra cserélni, amelyeket a saját konfigurációjához kíván használni.

  ```powershell
  $Sub5 = "Replace_With_the_New_Subcription_Name"
  $RG5 = "TestRG5"
  $Location5 = "Japan East"
  $VnetName5 = "TestVNet5"
  $FESubName5 = "FrontEnd"
  $BESubName5 = "Backend"
  $GWSubName5 = "GatewaySubnet"
  $VnetPrefix51 = "10.51.0.0/16"
  $VnetPrefix52 = "10.52.0.0/16"
  $FESubPrefix5 = "10.51.0.0/24"
  $BESubPrefix5 = "10.52.0.0/24"
  $GWSubPrefix5 = "10.52.255.0/27"
  $GWName5 = "VNet5GW"
  $GWIPName5 = "VNet5GWIP"
  $GWIPconfName5 = "gwipconf5"
  $Connection51 = "VNet5toVNet1"
  ```
2. Csatlakozzon az 5. előfizetéshez. Nyissa meg a PowerShell konzolt, és csatlakozzon a fiókjához. A következő minta segíthet a kapcsolódásban:

  ```powershell
  Login-AzureRmAccount
  ```

  Keresse meg a fiókot az előfizetésekben.

  ```powershell
  Get-AzureRmSubscription
  ```

  Válassza ki a használni kívánt előfizetést.

  ```powershell
  Select-AzureRmSubscription -SubscriptionName $Sub5
  ```
3. Hozzon létre egy új erőforráscsoportot.

  ```powershell
  New-AzureRmResourceGroup -Name $RG5 -Location $Location5
  ```
4. Hozza létre a TestVNet5 alhálózat-konfigurációit.

  ```powershell
  $fesub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $FESubName5 -AddressPrefix $FESubPrefix5
  $besub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $BESubName5 -AddressPrefix $BESubPrefix5
  $gwsub5 = New-AzureRmVirtualNetworkSubnetConfig -Name $GWSubName5 -AddressPrefix $GWSubPrefix5
  ```
5. Hozza létre a TestVNet5-öt.

  ```powershell
  New-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5 -Location $Location5 `
  -AddressPrefix $VnetPrefix51,$VnetPrefix52 -Subnet $fesub5,$besub5,$gwsub5
  ```
6. Kérjen egy nyilvános IP-címet.

  ```powershell
  $gwpip5 = New-AzureRmPublicIpAddress -Name $GWIPName5 -ResourceGroupName $RG5 `
  -Location $Location5 -AllocationMethod Dynamic
  ```
7. Hozza létre az átjáró konfigurációját.

  ```powershell
  $vnet5 = Get-AzureRmVirtualNetwork -Name $VnetName5 -ResourceGroupName $RG5
  $subnet5  = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet5
  $gwipconf5 = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName5 -Subnet $subnet5 -PublicIpAddress $gwpip5
  ```
8. Hozza létre a TestVNet5 átjáróját.

  ```powershell
  New-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5 -Location $Location5 `
  -IpConfigurations $gwipconf5 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1
  ```

### <a name="step-8---create-the-connections"></a>8. lépés – A kapcsolatok létrehozása

Ebben a példában, mivel az átjárók különböző előfizetésekben találhatóak, a lépést felosztottuk két PowerShell-munkamenetre, amelyek jelölése [1. előfizetés] és [5. előfizetés].

1. **[1. előfizetés]** Szerezze be az 1. előfizetés virtuális hálózati átjáróját. Jelentkezzen be és csatlakozzon az 1. előfizetéshez az alábbi példa futtatása előtt:

  ```powershell
  $vnet1gw = Get-AzureRmVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
  ```

  Másolja a következő elemek kimenetét, és küldje el a másolatokat az 5. előfizetés rendszergazdájának e-mailben vagy egyéb módon.

  ```powershell
  $vnet1gw.Name
  $vnet1gw.Id
  ```

  Ez a két elem hasonló értékekkel fog rendelkezni, mint a következő kimeneti példában látható értékek:

  ```
  PS D:\> $vnet1gw.Name
  VNet1GW
  PS D:\> $vnet1gw.Id
  /subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroupsTestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW
  ```
2. **[5. előfizetés]** Szerezze be az 5. előfizetés virtuális hálózati átjáróját. Jelentkezzen be és csatlakozzon az 5. előfizetéshez az alábbi példa futtatása előtt:

  ```powershell
  $vnet5gw = Get-AzureRmVirtualNetworkGateway -Name $GWName5 -ResourceGroupName $RG5
  ```

  Másolja a következő elemek kimenetét, és küldje el a másolatokat az 1. előfizetés rendszergazdájának e-mailben vagy egyéb módon.

  ```powershell
  $vnet5gw.Name
  $vnet5gw.Id
  ```

  Ez a két elem hasonló értékekkel fog rendelkezni, mint a következő kimeneti példában látható értékek:

  ```
  PS C:\> $vnet5gw.Name
  VNet5GW
  PS C:\> $vnet5gw.Id
  /subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW
  ```
3. **[1. előfizetés]** Hozza létre a TestVNet1–TestVNet5 kapcsolatot. Ebben a lépésben a TestVNet1 felől a TestVNet5 felé irányuló kapcsolatot hozza létre. A különbség itt az, hogy a $vnet5gw nem szerezhető be közvetlenül, mert másik előfizetésben található. Ehhez létre kell hoznia egy új PowerShell-objektumot a fenti lépésekben az 1. előfizetésből átküldött értékekkel. Használja az alábbi példát. Cserélje le a nevet, az azonosítót és a megosztott kulcsot a saját értékeire. Fontos, hogy a megosztott kulcs azonos legyen mindkét kapcsolathoz. A kapcsolat létrehozása egy kis időt vehet igénybe.

  Csatlakozzon az 1. előfizetéshez az alábbi példa futtatása előtt:

  ```powershell
  $vnet5gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
  $vnet5gw.Name = "VNet5GW"
  $vnet5gw.Id   = "/subscriptions/66c8e4f1-ecd6-47ed-9de7-7e530de23994/resourceGroups/TestRG5/providers/Microsoft.Network/virtualNetworkGateways/VNet5GW"
  $Connection15 = "VNet1toVNet5"
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet5gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```
4. **[5. előfizetés]** Hozza létre a TestVNet5–TestVNet1 kapcsolatot. Ez a lépés az előzőhöz hasonló. A különbség annyi, hogy ezúttal a TestVNet5 felől a TestVNet1 felé irányuló kapcsolatot hozza létre. Itt ugyanúgy egy PowerShell-objektumot kell létrehozni az 1. előfizetésből szerzett értékek alapján. Ügyeljen arra, hogy a megosztott kulcsok megegyezzenek ebben a lépésben.

  Csatlakozzon az 5. előfizetéshez az alábbi példa futtatása előtt:

  ```powershell
  $vnet1gw = New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
  $vnet1gw.Name = "VNet1GW"
  $vnet1gw.Id = "/subscriptions/b636ca99-6f88-4df4-a7c3-2f8dc4545509/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW "
  New-AzureRmVirtualNetworkGatewayConnection -Name $Connection51 -ResourceGroupName $RG5 -VirtualNetworkGateway1 $vnet5gw -VirtualNetworkGateway2 $vnet1gw -Location $Location5 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```

## <a name="verify"></a>A kapcsolat ellenőrzése

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

[!INCLUDE [verify connections powershell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="faq"></a>Virtuális hálózatok közötti kapcsolat – gyakori kérdések

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Következő lépések

* Miután a kapcsolat létrejött, hozzáadhat virtuális gépeket a virtuális hálózataihoz. További információkért tekintse meg a [Virtual Machines-dokumentációt](https://docs.microsoft.com/azure/#pivot=services&panel=Compute).
* Információk a BGP-ről: [A BGP áttekintése](vpn-gateway-bgp-overview.md) és [A BGP konfigurálása](vpn-gateway-bgp-resource-manager-ps.md).
