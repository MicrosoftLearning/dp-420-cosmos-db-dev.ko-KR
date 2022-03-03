---
lab:
  title: 포털 및 Azure Cosmos DB SQL API SDK에서 일관성 모델 구성
  module: Module 9 - Design and implement a replication strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 280f43ff34be1d12ff9767531d6909743678d53e
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025028"
---
# <a name="configure-consistency-models-in-the-portal-and-the-azure-cosmos-db-sql-api-sdk"></a>포털 및 Azure Cosmos DB SQL API SDK에서 일관성 모델 구성

새 Azure Cosmos DB SQL API 계정의 기본 일관성 수준은 세션 일관성입니다. 이 기본 설정은 향후 모든 요청에서 수정할 수 있습니다. 개별 요청 수준에서 한 단계 더 나아가 해당 특정 요청의 일관성 수준을 완화할 수 있습니다.

이 랩에서는 Azure Cosmos DB SQL API 계정의 기본 일관성 수준을 구성한 다음, SDK를 사용하여 개별 작업의 일관성 수준을 구성합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [시작 설명서][code.visualstudio.com/docs/getstarted]를 검토하세요.

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
    | **전역 배포** &vert; **지역 중복** | 사용 |
    | **무료 계층 할인 적용** | *적용 안 함* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **전역적으로 데이터 복제** 창으로 이동합니다.

1. **전역으로 데이터 복제** 창에서 계정에 두 개의 읽기 영역을 더 추가한 다음, 변경 내용을 **저장** 합니다.

1. 복제 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

    > &#128221; 이 작업은 약 5~10분 정도 걸릴 수 있으며 **기본 일관성** 창으로 이동합니다.

1. 리소스 블레이드에서 **기본 일관성** 창으로 이동합니다.

1. **기본 일관성** 창에서 **강력** 옵션을 선택한 다음, 변경 내용을 **저장** 합니다.

1. 이 작업을 계속하기 전에 기본 일관성 수준의 변경 내용이 유지되기를 기다립니다.

1. 리소스 블레이드에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 새 &vert; cosmicworks 만들기  |
    | **컨테이너 간에 처리량 공유** | 선택 안 함 |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |
    | **컨테이너 처리량** | 수동 &vert; *400*  |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **products** 컨테이너 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 확장한 다음, **항목** 을 관찰합니다.

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

새로 만든 계정의 자격 증명을 사용하여 SDK 클래스에 연결하고 새 데이터베이스 및 컨테이너 인스턴스를 만듭니다. 그런 다음, 데이터 탐색기를 사용하여 인스턴스가 Azure Portal에 있는지 확인합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **21-sdk-consistency-model** 폴더로 이동합니다.

1. **21-sdk-consistency-model** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **21-sdk-consistency-model** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

    > &#128221; **endpoint** 및 **key** 변수가 현재 사용되지 않는다는 컴파일러 경고가 표시될 수 있습니다. 이 작업에서 이러한 변수를 사용하게 되므로 이 경고를 무시해도 됩니다.

1. 통합 터미널을 닫습니다.

1. **product.cs** 코드 파일을 엽니다.

1. **Product** 레코드 및 해당 속성을 관찰합니다. 특히 이 랩에서는 **ID**, **이름** 및 **categoryId** 속성을 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창으로 돌아가서 **script.cs** 코드 파일을 엽니다.

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

## <a name="configure-consistency-level-for-a-point-operation"></a>지점 작업의 일관성 수준 구성

**ItemRequestOptions** 클래스에는 요청별로 구성 속성이 포함되어 있습니다. 이 클래스를 사용하면 현재 기본값인 강력에서 최종 일관성으로 일관성 수준을 완화합니다.

1. 값이 **7d9273d9-5d91-404c-bb2d-126abb6e4833** 인 **id** 라는 문자열 변수를 만듭니다.

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. 값이 **78d204a2-7d64-4f4a-ac29-9bfc437ae959** 인 **categoryId** 라는 문자열 변수를 만듭니다.

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. **categoryId** 변수에서 생성자 매개 변수로 전달하는 **partitionKey** 라는 이름의 **PartitionKey** 변수 형식을 만듭니다.

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. **id** 및 **partitionkey** 변수에서 메서드 매개 변수로 전달하고, **Product** 를 제네릭 형식으로 사용하고, **ItemResponse\<Product\>** 형식으로 이름이 **response** 인 변수에 결과를 저장하는 **container** 변수의 제네릭 **ReadItemAsync\<\>** 메서드를 비동기적으로 호출합니다.

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하고 형식이 지정된 출력 문자열을 사용하여 요청 요금을 출력합니다.

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    using CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **21-sdk-consistency-model** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 요청 요금(RU)을 콘솔에 인쇄해야 합니다.

    > &#128221; 현재 요청 요금은 **2RU** 여야 합니다. 이는 최신 쓰기가 있는지 확인하기 위해 둘 이상의 복제본에서 읽기가 필요한 강력한 일관성 때문입니다.

1. 통합 터미널을 닫습니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 코드 줄을 삭제합니다.

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    
    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. [ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] 속성을 [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] 열거형 값으로 설정하는 [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] 형식의 이름이 **options** 인 새 변수를 만듭니다.

    ```
    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    ```

1. **id**, **partitionkey** 및 **options** 변수에서 메서드 매개 변수로 전달하고, **Product** 를 제네릭 형식으로 사용하고, **ItemResponse\<Product\>** 형식으로 이름이 **response** 인 변수에 결과를 저장하는 **container** 변수의 제네릭 **ReadItemAsync\<\>** 메서드를 비동기적으로 호출합니다.

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하고 형식이 지정된 출력 문자열을 사용하여 요청 요금을 출력합니다.

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    using CosmosClient client = new CosmosClient(endpoint, key);
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    { 
        ConsistencyLevel = ConsistencyLevel.Eventual 
    };
    
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **21-sdk-consistency-model** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 요청 요금(RU)을 콘솔에 인쇄해야 합니다.

    > &#128221; 현재 요청 요금은 **1RU** 여야 합니다. 이는 단일 복제본에서 읽기만 필요한 최종 일관성 때문입니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
