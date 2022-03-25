---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 여러 문서를 대량으로 이동
  module: Module 4 - Access and manage data with the Azure Cosmos DB SQL API SDKs
ms.openlocfilehash: a602f5e827c5183cfb2af8220490b7b672ee2638
ms.sourcegitcommit: 9e320ed456eaaab98e80324267c710628b557b1c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139039325"
---
# <a name="move-multiple-documents-in-bulk-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 여러 문서를 대량으로 이동

대량 작업을 수행하는 방법을 알아보는 가장 쉬운 방법은 클라우드의 Azure Cosmos DB SQL API 계정에 많은 문서를 푸시하는 것입니다. SDK의 대량 기능을 사용하면 [System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] 네임스페이스를 활용하여 이 작업을 수행할 수 있습니다.

이 랩에서는 NuGet [Bogus][nuget.org/packages/bogus/33.1.1] 라이브러리를 사용하여 가상의 데이터를 생성하고 Azure Cosmos DB 계정에 배치합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [시작 설명서][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-an-azure-cosmos-db-sql-api-account-and-configure-the-sdk-project"></a>Azure Cosmos DB SQL API 계정 만들기 및 SDK 프로젝트 구성

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

1. 계속해서 새로 만든 **Azure Cosmos DB** 계정 리소스에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **새 컨테이너** 를 선택한 후, 다음 설정으로 새 컨테이너를 만들고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터베이스 ID** | 새 &vert; cosmicworks 만들기  |
    | **컨테이너 간에 처리량 공유** | 선택 안 함 |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |
    | **컨테이너 처리량** | *자동 크기 조정* &vert; *4000* |

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **08-sdk-bulk** 폴더로 이동합니다.

1. **08-sdk-bulk** 폴더 내에서 **script.cs** 코드 파일을 엽니다.

    > &#128221; **[Microsoft.Azure.Cosmos] [nuget.org/packages/microsoft.azure.cosmos/3.22.1]** 라이브러리는 이미 NuGet에서 미리 가져왔습니다.

1. **endpoint** 라는 **string** 변수를 찾습니다. 값을 이전에 만든 Azure Cosmos DB 계정의 **엔드포인트** 로 설정합니다.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 인 경우 C# 문은 **문자열 엔드포인트 = "https&shy;://dp420.documents.azure.com:443/"** 이 됩니다.

1. **key** 라는 **string** 변수를 찾습니다. 값을 이전에 만든 Azure Cosmos DB 계정의 **키** 로 설정합니다.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 예를 들어 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 C# 문은 **문자열 키 = "fDR2ci9QgkdkvERTQ=="** 가 됩니다.

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **08-sdk-bulk** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **08-sdk-bulk** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

1. 통합 터미널을 닫습니다.

## <a name="bulk-inserting-a-twenty-five-thousand-documents"></a>2만 5천 개의 문서 대량 삽입

이제 "본격적으로" 이 작동 방식을 확인하기 위해 많은 문서를 삽입해 보겠습니다. 내부 테스트에서 랩 가상 머신과 Azure Cosmos DB SQL API 계정이 지리적으로 서로 가까운 경우 이 작업은 약 1~2분 정도 걸릴 수 있습니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. **AllowBulkExecution** 속성이 **true** 값으로 설정된 [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]라는 **options** 클래스의 새 인스턴스를 만듭니다.

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
        , RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };
    ```

1. 생성자 매개 변수로서 **endpoint**, **key** 및 **options** 변수를 전달하는 **CosmosClient** 라는 **client** 클래스의 새 인스턴스를 만듭니다.

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. 데이터베이스 이름(*cosmicworks*) 및 컨테이너 이름(*products*)을 사용하여 기존 컨테이너를 검색하려면 **client** 변수의 [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] 메서드를 사용합니다.

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 이 특수 샘플 코드를 사용하여 NuGet에서 가져온 Bogus 라이브러리의 **Faker** 클래스를 사용하여 **25,000** 개의 가상 제품을 생성합니다.

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; [Bogus][nuget.org/packages/bogus/33.1.1] 라이브러리는 가상의 데이터를 디자인하여 사용자 인터페이스 애플리케이션을 테스트하는 데 사용되는 오픈 소스 라이브러리이며 대량 가져오기/내보내기 애플리케이션을 개발하는 방법을 학습하는 데 적합합니다.

1. **concurrentTasks** 라는 **Task** 형식의 새 제네릭 **List<>** 를 만듭니다.

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. 이 애플리케이션의 앞부분에서 생성된 제품 목록을 반복하는 foreach 루프를 만듭니다.

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. foreach 루프 내에서 Azure Cosmos DB SQL API에 제품을 비동기적으로 삽입하는 **Task** 를 만들어 파티션 키를 명시적으로 지정하고 **concurrentTasks** 라는 작업 목록에 작업을 추가합니다.

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. foreach 루프 후에는 비동기적으로 **concurrentTasks** 변수에 대한 **Task.WhenAll** 결과를 기다립니다.

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **대량 작업 완료** 의 정적 메시지를 콘솔에 출력합니다.

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
        , RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **08-sdk-bulk** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 애플리케이션은 자동으로 실행되어야 하며, 자동으로 완료되기 전에 실행하는 데 약 1~2분이 걸립니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

## <a name="observe-the-results"></a>결과 관찰

Azure Cosmos DB에 25,000개의 항목을 보냈으므로 이제 데이터 탐색기를 살펴보겠습니다.

1. 웹 브라우저에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 확인합니다.

1. **products** 노드를 확장한 다음, **Items** 노드를 선택합니다. 컨테이너 내의 항목 목록을 관찰합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 대량 작업을 사용하여 만든 모든 문서의 수를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT COUNT(1) FROM items
    ```

1. **쿼리 실행** 을 선택합니다.

1. 컨테이너의 항목 수를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
