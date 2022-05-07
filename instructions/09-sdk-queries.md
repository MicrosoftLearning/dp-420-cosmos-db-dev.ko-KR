---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 쿼리 실행
  module: Module 5 - Execute queries in Azure Cosmos DB SQL API
ms.openlocfilehash: cc0f5c470747dca5ea494b29eeb1b2397223ffd8
ms.sourcegitcommit: b86b01443b8043b4cfefd2cf6bf6b5104e2ff514
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/05/2022
ms.locfileid: "144773636"
---
# <a name="execute-a-query-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 쿼리 실행

최신 버전의 Azure Cosmos DB SQL API용 .NET SDK를 사용하면 그 어느 때보다 쉽게 컨테이너를 쿼리하고 최신 모범 사례 및 C#의 언어 기능을 사용하여 결과 집합을 비동기적으로 반복할 수 있습니다.

> &#128161; 이 랩에서는 NuGet에 있는 [Azure.Cosmos][nuget.org/packages/azure.cosmos/4.0.0-preview3]라이브러리의 *4.0.0-preview3* 릴리스를 사용합니다. 이 라이브러리에는 [비동기 스트림][docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]을 사용하여 Azure Cosmos DB를 더 쉽게 쿼리할 수 있는 특별한 기능이 있습니다.

이 랩에서는 비동기 스트림을 사용하여 Azure Cosmos DB SQL API에서 반환된 대규모 결과 집합을 반복합니다. .NET SDK를 사용하여 결과를 쿼리하고 반복합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기

Azure Cosmos DB는 여러 API를 지원하는 클라우드 기반 NoSQL 데이터베이스 서비스입니다. Azure Cosmos DB 계정을 처음으로 프로비저닝할 때 계정을 지원할 API(예: **Mongo API** 또는 **SQL API**)를 선택합니다. Azure Cosmos DB SQL API 계정 프로비저닝이 완료되면 엔드포인트 및 키를 검색하고 이를 사용하여 .NET용 Azure SDK 또는 선택한 다른 SDK를 사용하여 Azure Cosmos DB SQL API 계정에 연결할 수 있습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **+ 리소스 만들기** 를 선택하고 *Cosmos DB* 를 검색한 다음, 다음 설정을 사용하여 새 **Azure Cosmos DB SQL API** 계정 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | 기존 Azure 구독 |
    | **리소스 그룹** | 기존 리소스 그룹을 선택하거나 새로 만듭니다. |
    | **계정 이름** | 전역적으로 고유한 이름을 입력합니다. |
    | **위치** | 사용 가능한 지역을 선택합니다. |
    | **용량 모드** | *프로비전된 처리량* |
    | **무료 계층 할인 적용** | *적용 안 함* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히 다음 사항에 주의하세요.

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="seed-the-azure-cosmos-db-sql-api-account-with-data"></a>Azure Cosmos DB SQL API 계정을 데이터와 함께 시드

[cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구는 샘플 데이터를 Azure Cosmos DB SQL API 계정에 배포합니다. 이 도구는 오픈 소스이며 NuGet를 통해 구할 수 있습니다. 이 도구를 Azure Cloud Shell에 설치한 다음, 이를 사용하여 데이터베이스를 시드합니다.

1. **Visual Studio Code** 에서 **터미널** 메뉴를 연 다음, **새 터미널** 을 선택하여 새 터미널 인스턴스를 엽니다.

1. 컴퓨터에서 전역으로 사용할 수 있는 [cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구를 설치합니다.

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; 이 명령을 완료하는 데 몇 분 정도 걸릴 수 있습니다. 이 도구의 최신 버전이 이미 설치된 경우 경고 메시지(*'cosmicworks' 도구는 이미 설치되어 있습니다')가 출력됩니다.

1. cosmicworks를 실행하여 다음 명령줄 옵션을 사용하여 Azure Cosmos DB 계정을 시드합니다.

    | **옵션** | **값** |
    | ---: | :--- |
    | **--endpoint** | *이 랩에서 이전에 복사한 엔드포인트 값* |
    | **--key** | *이 랩에서 이전에 복사한 키 값* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 이고 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 명령은 ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``가 됩니다.

1. **cosmicworks** 명령이 데이터베이스, 컨테이너 및 항목으로 계정을 채울 때까지 기다립니다.

1. 통합 터미널을 닫습니다.

## <a name="iterate-over-the-results-of-a-sql-query-using-the-sdk"></a>SDK를 사용하여 SQL 쿼리의 결과 반복

이제 비동기 스트림을 사용하여 Azure Cosmos DB에서 페이지를 매긴 결과를 통해 이해하기 쉬운 foreach 루프를 만듭니다. 내부적으로 SDK는 피드 반복기를 관리하고 후속 요청이 올바르게 호출되도록 합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **09-execute-query-sdk** 폴더로 이동합니다.

1. **product.cs** 코드 파일을 엽니다.

1. **Product** 클래스 및 해당 속성을 관찰합니다. 특히 이 랩에서는 **ID**, **이름** 및 **price** 속성을 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창으로 돌아가서 **script.cs** 코드 파일을 엽니다.

1. **endpoint** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **endpoint** 로 설정된 값으로 업데이트합니다.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 인 경우 C# 문은 **문자열 엔드포인트 = "https&shy;://dp420.documents.azure.com:443/"** 이 됩니다.

1. **키** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **키** 로 설정된 값으로 업데이트합니다.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 예를 들어 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 C# 문은 **문자열 키 = "fDR2ci9QgkdkvERTQ=="** 가 됩니다.

1. **SELECT * FROM products p** 값으로 문자열 형식의 **sql** 이라는 새 변수를 만듭니다.

    ```
    string sql = "SELECT * FROM products p";
    ```

1. **sql** 변수를 생성자에 매개 변수로 전달하는 [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] 형식의 새 변수를 만듭니다.

    ```
    QueryDefinition query = new (sql);
    ```

1. **query** 변수에서 매개 변수로 전달하는 [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] 클래스의 제네릭 [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] 메서드를 호출한 다음, **Product** 형식의 인스턴스를 나타내기 위해 **product** 변수를 사용하여 결과를 비동기적으로 반복하는 방식으로 새 **await foreach** 루프를 만듭니다.

    ```
    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
    }
    ```

1. **await foreach** 루프 내에서 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **product** 변수의 **id**, **name** 및 **price** 속성의 서식을 지정하고 인쇄합니다.

    ```
    Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    CosmosDatabase database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    CosmosContainer container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. **script.cs** 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **09-execute-query-sdk** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 이제 스크립트가 컨테이너의 모든 제품을 출력합니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]: https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/azure.cosmos/4.0.0-preview3]: https://www.nuget.org/packages/azure.cosmos/4.0.0-preview3
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
