---
title: "Csatlakozás az Azure SQL Data Warehouse-hoz az sqlcmd használatával | Microsoft Docs"
description: "Az [sqlcmd][sqlcmd] parancssori segédprogramot az Azure SQL Data Warehouse lekérdezéséhez és az ahhoz való csatlakozáshoz használhatja."
services: sql-data-warehouse
documentationcenter: NA
author: antvgski
manager: jhubbard
editor: 
ms.assetid: 6e2b69e5-4806-4e91-9ea1-e2b63bf28c46
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: get-started-article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.custom: connect
ms.date: 10/31/2016
ms.author: anvang;barbkess
ms.translationtype: Human Translation
ms.sourcegitcommit: eeb56316b337c90cc83455be11917674eba898a3
ms.openlocfilehash: 3508cdf6dcfa3d7122e1e3b635f3cd37863dbf62
ms.contentlocale: hu-hu
ms.lasthandoff: 04/03/2017


---
# <a name="connect-to-sql-data-warehouse-with-sqlcmd"></a>Csatlakozás az SQL Data Warehouse-hoz az sqlcmd használatával
> [!div class="op_single_selector"]
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Azure Machine Learning](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

Az [sqlcmd][sqlcmd] parancssori segédprogramot az Azure SQL Data Warehouse lekérdezéséhez és az ahhoz való csatlakozáshoz használhatja.  

## <a name="1-connect"></a>1. Kapcsolódás
Az [sqlcmd][sqlcmd] használatának megkezdéséhez nyissa meg a parancssort, és írja be az **sqlcmd** kifejezést, majd a saját SQL Data Warehouse-adatbázisának kapcsolati karakterláncát. A kapcsolati karakterláncban a következő paraméterekre van szükség:

* **Server (-S):** A kiszolgáló neve `<`kiszolgálónév`>`.database.windows.net formátumban.
* **Database (-d):** Az adatbázis neve.
* **Enable Quoted Identifiers (-I):** Az SQL Data Warehouse-példányokhoz való csatlakozáshoz engedélyezni kell a határolójeles azonosítókat.

Az SQL Server-hitelesítés használatához meg kell adnia a felhasználónév/jelszó paramétereit:

* **User (-U):** A kiszolgálói felhasználó neve `<`felhasználó`>` formátumban.
* **Password (-P):** A felhasználóhoz tartozó jelszó.

A kapcsolati karakterlánc például a következőképpen nézhet ki:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

Az Azure Active Directory beépített hitelesítés használatához meg kell adnia az Azure Active Directory paramétereit:

* **Azure Active Directory Authentication (-G):** az Azure Active Directory használata a hitelesítéshez

A kapcsolati karakterlánc például a következőképpen nézhet ki:

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> Az Active Directory használatával történő hitelesítéshez [engedélyeznie kell az Azure Active Directory-hitelesítést](sql-data-warehouse-authentication.md).
> 
> 

## <a name="2-query"></a>2. Lekérdezés
A kapcsolódás után kiadhatók a példányon a támogatott Transact-SQL utasítások.  Ebben a példában a lekérdezések elküldése interaktív módban történik.

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

A következő példák bemutatják, hogyan lehet a lekérdezéseket kötegelt módban futtatni a -Q kapcsoló használatával vagy az SQL átirányításával az sqlcmd-re.

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

## <a name="next-steps"></a>Következő lépések
Az sqlcmd-ben elérhető további lehetőségek részleteit az [sqlcmd dokumentációjában][sqlcmd] tekintheti meg.

<!--Image references-->

<!--Article references-->

<!--MSDN references--> 
[sqlcmd]: https://msdn.microsoft.com/library/ms162773.aspx
[Azure portal]: https://portal.azure.com

<!--Other Web references-->

