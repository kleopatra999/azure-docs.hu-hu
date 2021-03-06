---
title: "Az Azure Data Lake Store használatának első lépései az Azure 2.0-s verziójú parancssori felületével | Microsoft Docs"
description: "Data Lake Store-fiók létrehozása és alapszintű műveletek végrehajtása az Azure 2.0-s verziójú, platformfüggetlen parancssorával"
services: data-lake-store
documentationcenter: 
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: 4ffa0f4a-1cca-46ac-803d-1fc8538c685b
ms.service: data-lake-store
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/29/2017
ms.author: nitinme
ms.translationtype: Human Translation
ms.sourcegitcommit: 6efa2cca46c2d8e4c00150ff964f8af02397ef99
ms.openlocfilehash: ed78d25f2bac0a9996f1796ee503f31a36940977
ms.contentlocale: hu-hu
ms.lasthandoff: 07/01/2017


---
<a id="get-started-with-azure-data-lake-store-using-azure-cli-20" class="xliff"></a>

# Az Azure Data Lake Store használatának első lépései az Azure CLI 2.0 használatával
> [!div class="op_single_selector"]
> * [Portal](data-lake-store-get-started-portal.md)
> * [PowerShell](data-lake-store-get-started-powershell.md)
> * [.NET SDK](data-lake-store-get-started-net-sdk.md)
> * [Java SDK](data-lake-store-get-started-java-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Azure CLI 2.0](data-lake-store-get-started-cli-2.0.md)
> * [Node.js](data-lake-store-manage-use-nodejs.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

Ismerje meg, hogyan hozhat létre Azure Data Lake Store-fiókot az Azure CLI 2.0 használatával, illetve hogyan végezhet el olyan alapvető műveleteket, mint például mappák létrehozása, adatfájlok le- és feltöltése, fiók törlése stb. További információk a Data Lake Store-ról: [Overview of Data Lake Store](data-lake-store-overview.md) (A Data Lake Store áttekintése).

Az Azure CLI 2.0 az Azure új parancssori felülete, amely Azure-erőforrások felügyeletére szolgál. A szolgáltatás macOS, Linux és Windows rendszereken használható. További információért lásd: [Az Azure CLI 2.0 áttekintése](https://docs.microsoft.com/cli/azure/overview). A parancsok és a szintaxis teljes listája az [Azure Data Lake Store CLI 2.0 dokumentációjában](https://docs.microsoft.com/cli/azure/dls) található.


<a id="prerequisites" class="xliff"></a>

## Előfeltételek
A cikk elkezdéséhez az alábbiakkal kell rendelkeznie:

* **Azure-előfizetés**. Lásd: [Ingyenes Azure-fiók létrehozása](https://azure.microsoft.com/pricing/free-trial/).

* **Azure CLI 2.0** – az utasításokért lásd: [Az Azure CLI 2.0 telepítése](https://docs.microsoft.com/cli/azure/install-azure-cli).

<a id="authentication" class="xliff"></a>

## Authentication

Ez a cikk egy egyszerűbb hitelesítési módszert használ a Data Lake Store-ral, ahol Ön végfelhasználóként jelentkezik be. Ezután a Data Lake Store-fiókhoz és a fájlrendszerhez való hozzáférés szintje a bejelentkezett felhasználó hozzáférési szintjétől függ. Azonban a Data Lake Store-ral más módokon is lehet hitelesíteni. Ezek a következők: **végfelhasználói hitelesítés** vagy **szolgáltatások közötti hitelesítés**. Útmutatás a hitelesítéshez és további tudnivalók a [Végfelhasználói hitelesítés](data-lake-store-end-user-authenticate-using-active-directory.md) vagy a [Szolgáltatások közötti hitelesítés](data-lake-store-authenticate-using-active-directory.md) című témakörben.


<a id="log-in-to-your-azure-subscription" class="xliff"></a>

## Bejelentkezés az Azure-előfizetésbe

1. Jelentkezzen be az Azure-előfizetésébe.

    ```azurecli
    az login
    ```

    Kap egy kódot a következő lépésben való használatra. A webböngészővel nyissa meg a https://aka.ms/devicelogin oldalt, és a hitelesítéshez adja meg a kódot. A rendszer kéri a hitelesítési adatokkal való bejelentkezést.

2. Bejelentkezés után az ablakban megjelenő listában találhatók a fiókhoz társított Azure-előfizetések. Az alábbi paranccsal használhat egy adott előfizetést.
   
    ```azurecli
    az account set --subscription <subscription id> 
    ```

<a id="create-an-azure-data-lake-store-account" class="xliff"></a>

## Azure Data Lake Store-fiók létrehozása

1. Hozzon létre egy új erőforráscsoportot. Az alábbi parancsban adja meg a használni kívánt paraméterértékeket. Ha a hely neve tartalmaz szóközöket, használjon idézőjeleket. Például: „USA 2. keleti régiója”. 
   
    ```azurecli
    az group create --location "East US 2" --name myresourcegroup
    ```

2. Hozza létre a Data Lake Store-fiókot.
   
    ```azurecli
    az dls account create --account mydatalakestore --resource-group myresourcegroup
    ```

<a id="create-folders-in-a-data-lake-store-account" class="xliff"></a>

## Mappák létrehozása Data Lake Store-fiókban

Mappákat hozhat létre az Azure Data Lake Store-fiókjában az adatok kezelése és tárolása céljából. Az alábbi parancs segítségével hozzon létre egy **mynewfolder** nevű mappát a Data Lake Store gyökérmappájában.

```azurecli
az dls fs create --account mydatalakestore --path /mynewfolder --folder
```

> [!NOTE]
> A `--folder` paraméter gondoskodik arról, hogy a parancs egy mappát hozzon létre. Ha a paraméter hiányzik, a parancs egy mynewfolder nevű üres fájlt hoz létre a Data Lake Store-fiók gyökérmappájában.
> 
>

<a id="upload-data-to-a-data-lake-store-account" class="xliff"></a>

## Adatok feltöltése a Data Lake Store-fiókba

Az adatokat közvetlenül a Data Lake Store gyökérmappájába vagy a fiókban létrehozott egyik mappába töltheti fel. Az alábbi kódtöredékek bemutatják, hogyan tölthet fel néhány adatot az előző szakaszban létrehozott mappába (**mynewfolder**).

Ha feltölthető mintaadatokra van szüksége, használhatja az [Azure Data Lake Git-tárában](https://github.com/MicrosoftBigData/usql/tree/master/Examples/Samples/Data/AmbulanceData) található **Ambulance Data** mappát. Töltse le a fájlt, és tárolja a számítógépén egy helyi könyvtárban (pl. C:\sampledata).

```azurecli
az dls fs upload --account mydatalakestore --source-path "C:\SampleData\AmbulanceData\vehicle1_09142014.csv" --destination-path "/mynewfolder/vehicle1_09142014.csv"
```

> [!NOTE]
> Célként adja meg a teljes elérési utat, beleértve a fájlnevet is.
> 
>


<a id="list-files-in-a-data-lake-store-account" class="xliff"></a>

## A Data Lake Store-fiók fájljainak listázása

Az alábbi parancs segítségével kilistázhatja a Data Lake Store-fiók fájljait.

```azurecli
az dls fs list --account mydatalakestore --path /mynewfolder
```

A kimenet az alábbihoz hasonló lesz:

    [
        {
            "accessTime": 1491323529542,
            "aclBit": false,
            "blockSize": 268435456,
            "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
            "length": 1589881,
            "modificationTime": 1491323531638,
            "msExpirationTime": 0,
            "name": "mynewfolder/vehicle1_09142014.csv",
            "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
            "pathSuffix": "vehicle1_09142014.csv",
            "permission": "770",
            "replication": 1,
            "type": "FILE"
        }
    ]

<a id="rename-download-and-delete-data-from-a-data-lake-store-account" class="xliff"></a>

## A Data Lake Store-fiókban lévő adatok átnevezése, letöltése és törlése 

* **Fájlok átnevezéséhez** használja az alábbi parancsot:
  
    ```azurecli
    az dls fs move --account mydatalakestore --source-path /mynewfolder/vehicle1_09142014.csv --destination-path /mynewfolder/vehicle1_09142014_copy.csv
    ```

* **Fájlok letöltéséhez** használja az alábbi parancsot: Ügyeljen arra, hogy a megadott cél elérési útja egy létező hely legyen.
  
    ```azurecli     
    az dls fs download --account mydatalakestore --source-path /mynewfolder/vehicle1_09142014_copy.csv --destination-path "C:\mysampledata\vehicle1_09142014_copy.csv"
    ```

    > [!NOTE]
    > A parancs létrehozza a célmappát, ha az nem létezik.
    > 
    >

* **Fájlok törléséhez** használja az alábbi parancsot:
  
    ```azurecli
    az dls fs delete --account mydatalakestore --path /mynewfolder/vehicle1_09142014_copy.csv
    ```

    Ha egyetlen paranccsal szeretné törölni a **mynewfolder** nevű mappát és a **vehicle1_09142014_copy.csv** nevű fájlt, használja a --recurse paramétert

    ```azurecli
    az dls fs delete --account mydatalakestore --path /mynewfolder --recurse
    ```

<a id="work-with-permissions-and-acls-for-a-data-lake-store-account" class="xliff"></a>

## A Data Lake Store-fiókhoz tartozó engedélyek és a hozzáférés-vezérlési listák használata

Ebben a szakaszban a hozzáférés-vezérlési listák és az engedélyek Azure CLI 2.0-beli felügyeletét ismerheti meg. A hozzáférés-vezérlési listák Azure Data Lake Store-beli használatának részletes leírásáért lásd: [Az Azure Data Lake Store szolgáltatásban található hozzáférés-vezérlés](data-lake-store-access-control.md).

* **Egy fájl/mappa tulajdonosának frissítését** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access set-owner --account mydatalakestore --path /mynewfolder/vehicle1_09142014.csv --group 80a3ed5f-959e-4696-ba3c-d3c8b2db6766 --owner 6361e05d-c381-4275-a932-5535806bb323
    ```

* **Egy fájl/mappa engedélyeinek frissítését** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access set-permission --account mydatalakestore --path /mynewfolder/vehicle1_09142014.csv --permission 777
    ```
    
* **Adott elérési úthoz tartozó hozzáférés-vezérlési listák beolvasását** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access show --account mydatalakestore --path /mynewfolder/vehicle1_09142014.csv
    ```

    A kimenet az alábbihoz hasonló lesz:

        {
            "entries": [
            "user::rwx",
            "group::rwx",
            "other::---"
          ],
          "group": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
          "owner": "1808bd5f-62af-45f4-89d8-03c5e81bac20",
          "permission": "770",
          "stickyBit": false
        }

* **Egy hozzáférés-vezérlési listához tartozó bejegyzés beállítását** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access set-entry --account mydatalakestore --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323:-w-
    ```

* **Egy hozzáférés-vezérlési listához tartozó bejegyzés eltávolítását** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access remove-entry --account mydatalakestore --path /mynewfolder --acl-spec user:6360e05d-c381-4275-a932-5535806bb323
    ```

* **Egy alapértelmezett teljes hozzáférés-vezérlési lista eltávolítását** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access remove-all --account mydatalakestore --path /mynewfolder --default-acl
    ```

* **Egy nem alapértelmezett teljes hozzáférés-vezérlési lista eltávolítását** az alábbi paranccsal végezheti el:

    ```azurecli
    az dls fs access remove-all --account mydatalakestore --path /mynewfolder
    ```
    
<a id="delete-a-data-lake-store-account" class="xliff"></a>

## Data Lake Store-fiók törlése
Az alábbi parancs segítségével törölheti a Data Lake Store-fiókját.

```azurecli
az dls account delete --account mydatalakestore
```

Ha a rendszer rákérdez, írja be az **Y** karaktert a fiók törléséhez.

<a id="next-steps" class="xliff"></a>

## Következő lépések

* [Az Azure Data Lake Store CLI 2.0 dokumentációja](https://docs.microsoft.com/cli/azure/dls)
* [Biztonságos adattárolás a Data Lake Store-ban](data-lake-store-secure-data.md)
* [Az Azure Data Lake Analytics használata a Data Lake Store-ral](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Az Azure HDInsight használata a Data Lake Store-ral](data-lake-store-hdinsight-hadoop-use-portal.md)

[azure-command-line-tools]: ../xplat-cli-install.md

