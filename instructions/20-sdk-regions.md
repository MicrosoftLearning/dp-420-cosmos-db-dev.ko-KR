---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 다중 지역에 연결
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 8d6439943bc1baac6e5c1cdb8e40f98cf6665119
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025086"
---
# <a name="connect-to-different-regions-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 다중 지역에 연결

Azure Cosmos DB SQL API 계정에 대해 지역 중복성을 사용하도록 설정하면 SDK를 사용하여 구성한 순서대로 지역에서 데이터를 읽을 수 있습니다. 이 기법은 사용 가능한 모든 읽기 영역에 읽기 요청을 배포할 때 유용합니다.

이 앱에서는 수동으로 구성한 대체 순서로 읽기 영역에 연결하도록 CosmosClient 클래스를 구성합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 시작

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [시작 설명서][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기

Azure Cosmos DB는 여러 API를 지원하는 클라우드 기반 NoSQL 데이터베이스 서비스입니다. Azure Cosmos DB 계정을 처음으로 프로비전할 때 계정을 지원할 API(예: **Mongo API** 또는 **SQL API**)를 선택합니다. Azure Cosmos DB SQL API 계정 프로비전이 완료되면 엔드포인트 및 키를 검색하고 이를 사용하여 .NET용 Azure SDK 또는 선택한 다른 SDK를 사용하여 Azure Cosmos DB SQL API 계정에 연결할 수 있습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **+ 리소스 만들기** 를 선택하고 *Cosmos DB* 를 검색한 다음, 다음 설정을 사용하여 새 **Azure Cosmos DB SQL API** 계정 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | *기존 Azure 구독* |
    | **리소스 그룹** | *기존 리소스 그룹을 선택하거나 새로 만듭니다.* |
    | **계정 이름** | *전역적으로 고유한 이름을 입력합니다.* |
    | **위치** | *사용 가능한 지역을 선택합니다.* |
    | **용량 모드** | *프로비전된 처리량* |
    | **무료 계층 할인 적용** | *적용 안 함* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **전역적으로 데이터 복제** 창으로 이동합니다.

1. **전역으로 데이터 복제** 창에서 계정에 두 개의 읽기 영역을 더 추가한 다음, 변경 내용을 **저장** 합니다.

1. 복제 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

    > &#128221; 이 작업은 약 5~10분이 걸릴 수 있습니다.

1. **쓰기**(기본) 지역 및 두 **읽기** 지역의 이름을 기록합니다. 이 연습의 뒷부분에서 이러한 지역 이름을 사용합니다.

    > &#128221; 예를 들어 주 지역이 **북유럽** 이고 두 개의 읽기 보조 지역이 **미국 동부 2** 이고 **남아프리카 북부** 인 경우 이러한 세 이름을 모두 있는 그대로 기록합니다.

1. 리소스 블레이드에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 페이지에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | *새로 만들기* &vert; *cosmicworks* |
    | **컨테이너 간에 처리량 공유** | *선택 안 함* |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |
    | **컨테이너 처리량** | *수동* &vert; *400* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **제품** 컨테이너 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **cosmicworks** 데이터베이스 노드를 확장하고 **제품** 컨테이너 노드를 확장한 다음, **항목** 을 관찰합니다.

1. 계속해서 **데이터 탐색기** 창의 명령 모음에서 **새 항목** 을 선택합니다. 편집기에서 자리 표시자 JSON 항목을 다음 콘텐츠로 바꿉니다.

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. 명령 모음에서 **저장** 을 선택하여 JSON 항목을 추가합니다.

1. **항목** 탭의 **항목** 창에서 새 항목을 확인합니다.

1. 리소스 블레이드에서 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK에서 Azure Cosmos DB SQL API 계정에 연결

새로 만든 계정의 자격 증명을 사용하여 SDK 클래스에 연결하고 다른 지역에서 데이터베이스 및 컨테이너 인스턴스에 연결합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **20-sdk-regions** 폴더로 이동합니다.

1. **20-sdk-regions** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **20-sdk-regions** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

    > &#128221; **endpoint** 및 **key** 변수가 현재 사용되지 않는다는 컴파일러 경고가 표시될 수 있습니다. 이 작업에서 이러한 변수를 사용하게 되므로 이 경고를 무시해도 됩니다.

1. 통합 터미널을 닫습니다.

1. **20-sdk-regions** 폴더 내에서 **script.cs** 코드 파일을 엽니다.

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

## <a name="configure-the-net-sdk-with-a-preferred-region-list"></a>기본 설정 지역 목록을 사용하여 .NET SDK 구성

**CosmosClientOptions** 클래스에는 SDK로 연결하려는 지역 목록을 구성하는 속성이 포함되어 있습니다. 목록은 장애 조치(failover) 우선 순위에 따라 정렬되며 구성한 순서대로 각 지역에 연결을 시도합니다.

1. 세 번째 지역에서 시작하여 첫 번째(기본) 지역으로 끝나는 계정으로 구성한 지역 목록이 포함된 일반 유형 **목록\<string\>** 의 새 변수를 만듭니다. 예를 들어 **미국 서부** 지역에서 Azure Cosmos DB SQL API 계정을 만든 다음, **남아프리카 공화국 북부** 를 추가하고 마지막으로 **동아시아** 를 추가한 경우 목록 변수에는 다음이 포함됩니다.

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > &#128161; 대신 여러 Azure 지역에 대한 기본 제공 문자열 속성을 포함하는 [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 정적 클래스를 사용할 수 있습니다.

1. [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] 속성이 **regions** 변수로 설정된 **CosmosClientOptions** 라는 **options** 클래스의 새 인스턴스를 만듭니다.

    ```
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    ```

1. 생성자 매개 변수로서 **endpoint**, **key** 및 **options** 변수를 전달하는 **CosmosClient** 라는 **client** 클래스의 새 인스턴스를 만듭니다.

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. 데이터베이스 이름(*cosmicworks*) 및 컨테이너 이름(*products*)을 사용하여 기존 컨테이너를 검색하려면 **client** 변수의 **GetContainer** 메서드를 사용합니다.

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. 서버에서 특정 항목을 검색하고 nullable [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] 형식의 **response** 라는 변수에 결과를 저장하려면 **container** 변수의 [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] 메서드를 사용합니다.

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하여 현재 항목 식별자와 JSON 진단 데이터를 인쇄합니다.

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };
    
    CosmosClientOptions options = new () 
    { 
        ApplicationPreferredRegions = regions
    };
    
    using CosmosClient client = new(endpoint, key, options);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **20-sdk-regions** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 컨테이너의 이름 및 JSON 진단 데이터는 콘솔 출력에 인쇄되어야 합니다.

1. JSON 진단 데이터를 검토합니다. **HttpResponseStats** 라는 속성과 **RequestUri** 라는 자식 속성을 검색합니다. 이 속성의 값은 이 랩에서 앞부분에서 구성한 이름과 지역을 포함하는 URI여야 합니다.

    > &#128221; 예를 들어 계정 이름이 **dp420** 이고 구성한 첫 번째 지역이 **동아시아** 인 경우 JSON 속성의 값은 **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products** 가 됩니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
