---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 문서 생성 및 업데이트
  module: Module 4 - Implement Azure Cosmos DB SQL API point operations
ms.openlocfilehash: b4b167618243026dd3b2d9510da1b73555b1c136
ms.sourcegitcommit: 9e320ed456eaaab98e80324267c710628b557b1c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139039326"
---
# <a name="create-and-update-documents-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 문서 생성 및 업데이트

[Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 클래스에는 Azure Cosmos DB SQL API 컨테이너 내에서 항목을 만들고, 검색하고, 업데이트하고, 삭제하는 멤버 메서드 집합이 포함되어 있습니다. 이들 메서드는 SQL API 컨테이너 내의 다양한 항목에서 가장 일반적인 "CRUD" 작업 중 일부를 함께 수행합니다.

이 랩에서는 SDK를 사용하여 Azure Cosmos DB SQL API 컨테이너 내의 항목에 대해 일상적인 CRUD 작업을 수행합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [시작 설명서][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기

Azure Cosmos DB는 여러 API를 지원하는 클라우드 기반 NoSQL 데이터베이스 서비스입니다. Azure Cosmos DB 계정을 처음으로 프로비전할 때 계정을 지원할 API(예: **Mongo API** 또는 **SQL API**)를 선택합니다. Azure Cosmos DB SQL API 계정 프로비저닝이 완료되면 엔드포인트 및 키를 검색하고 이를 사용하여 .NET용 Azure SDK 또는 선택한 다른 SDK를 사용하여 Azure Cosmos DB SQL API 계정에 연결할 수 있습니다.

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

## <a name="connect-to-the-azure-cosmos-db-sql-api-account-from-the-sdk"></a>SDK에서 Azure Cosmos DB SQL API 계정에 연결

새로 만든 계정의 자격 증명을 사용하여 SDK 클래스에 연결하고 새 데이터베이스 및 컨테이너 인스턴스를 만듭니다. 그런 다음, 데이터 탐색기를 사용하여 인스턴스가 Azure Portal에 있는지 확인합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **06-sdk-crud** 폴더로 이동합니다.

1. **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **06-sdk-crud** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

1. 통합 터미널을 닫습니다.

1. **06-sdk-crud** 폴더 내에서 **script.cs** 코드 파일을 엽니다.

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** 라이브러리는 이미 NuGet에서 미리 가져왔습니다.

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

1. 에뮬레이터 내에서 만들려는 새 데이터베이스(**cosmicworks**)의 이름을 전달하고 결과를 **Database** 형식의 변수에 저장하는 **client** 변수의 CreateDatabaseIfNotExistsAsync 메서드를 비동기적으로 호출합니다.

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 새 컨테이너(**products**), 파티션 키 경로( **/categoryId**) 및 **cosmicworks** 데이터베이스 내에서 만들려는 처리량(**400**)을 전달하고 결과를 **Container** 형식의 변수에 저장하는 **database** 변수의 **CreateContainerIfNotExistsAsync** 메서드를 비동기적으로 호출합니다.
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="perform-create-and-read-point-operations-on-items-with-the-sdk"></a>SDK를 사용하여 항목에 대한 만들기 및 읽기 지점 작업 수행

이제 Microsoft.Azure.Cosmos.Container 클래스에서 비동기 메서드 집합을 사용하여 SQL API 컨테이너 내의 항목에 대한 일반적인 작업을 수행합니다. 이러한 작업은 모두 C#의 작업 비동기 프로그래밍 모델을 사용하여 수행됩니다.

1. **Visual Studio Code** 로 돌아갑니다. **06-sdk-crud** 폴더 내에서 **product.cs** 코드 파일을 엽니다.

    > &#128221; **script.cs** 파일의 편집기를 닫지 마세요.

1. 이 코드 파일 내에서 **Product** 클래스를 관찰합니다. 이 클래스는 이 컨테이너 내에 저장되고 조작될 제품 항목을 나타냅니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 속성을 사용하여 **Product** 라는 **saddle** 형식의 새 개체를 만듭니다.

    | 속성 | 값 |
    | ---: | :--- |
    | **id** | ``706cd7c6-db8b-41f9-aea2-0e0c7e8eb009`` |
    | **categoryId** | ``9603ca6c-9e28-4a02-9194-51cdb7fea816`` |
    | **name** | ``Road Saddle`` |
    | **price** | ``45.99d`` |
    | **태그** | ``{ tan, new, crisp }`` |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. 메서드 매개 변수로서 **saddle** 변수를 전달하고 제네릭 유형으로서 **Product** 를 사용하여 **container** 변수의 제네릭 [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] 메서드를 비동기적으로 호출합니다.

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 코드 줄을 삭제합니다.

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. 값이 **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** 인 **id** 라는 문자열 변수를 만듭니다.

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. 값이 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 인 **categoryId** 라는 문자열 변수를 만듭니다.

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. 생성자 매개 변수로서 **categoryId** 변수를 전달하는 **partitionKey** 라는 [PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] 형식의 변수를 만듭니다.

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. **id** 및 **partitionkey** 변수에서 메서드 매개 변수로 전달하고, **Product** 를 제네릭 형식으로 사용하고, **Product** 형식으로 이름이 **saddle** 인 변수에 결과를 저장하는 **container** 변수의 제네릭 [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] 메서드를 비동기적으로 호출합니다.

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하고 형식이 지정된 출력 문자열을 사용하여 saddle 개체를 출력합니다.

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 특히 항목의 ID, 이름 및 가격을 사용하여 서식이 지정된 출력 텍스트를 관찰합니다.

1. 통합 터미널을 닫습니다.

## <a name="perform-update-and-delete-point-operations-with-the-sdk"></a>SDK를 사용하여 업데이트 및 삭제 지점 작업 수행

SDK를 학습하는 동안 온라인 Azure Cosmos DB SDK 계정 또는 에뮬레이터를 사용하여 항목을 업데이트하고 작업을 수행할 때 데이터 탐색기와 선택한 IDE 사이를 왔다 갔다 하면서 변경 내용이 적용되었는지 확인하는 것은 드문 일이 아닙니다. 여기서는 SDK를 사용하여 항목을 업데이트하고 삭제하면 됩니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확장합니다.

1. **항목** 노드를 선택합니다. 컨테이너 내에서 유일한 항목을 선택한 다음, 항목의 **이름** 및 **가격** 속성 값을 관찰합니다.

    | **속성** | **값** |
    | ---: | :--- |
    | **이름** | Road Saddle |
    | **가격** | $45.99 |

    > &#128221; 이 때는 항목을 만들었기 때문에 이 값이 변경되지 않았을 것입니다. 이 연습에서는 이러한 값을 변경합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 로 돌아갑니다. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 코드 줄을 삭제합니다.

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 가격 속성 값을 **32.55** 로 설정하여 **saddle** 변수를 변경합니다.

    ```
    saddle.price = 32.55d;
    ```

1. **name** 속성 값을 **Road LL Saddle** 로 설정하여 **saddle** 변수를 다시 수정합니다.

    ```
    saddle.name = "Road LL Saddle";
    ```

1. 메서드 매개 변수로서 **saddle** 변수를 전달하고 제네릭 유형으로서 **Product** 를 사용하여 **container** 변수의 제네릭 [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] 메서드를 비동기적으로 호출합니다.

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확장합니다.

1. **항목** 노드를 선택합니다. 컨테이너 내에서 유일한 항목을 선택한 다음, 항목의 **이름** 및 **가격** 속성 값을 관찰합니다.

    | **속성** | **값** |
    | ---: | :--- |
    | **이름** | Road LL Saddle |
    | **가격** | $32.55 |

    > &#128221; 이 때는 항목을 관찰했기 때문에 이 값을 변경해야 합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 로 돌아갑니다. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 코드 줄을 삭제합니다.

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 메서드 매개 변수로서 **id** 및 **partitionkey** 변수를 전달하고 제네릭 유형으로서 **Product** 를 사용하여 **container** 변수의 제네릭 [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] 메서드를 비동기적으로 호출합니다.

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **06-sdk-crud** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확장합니다.

1. **항목** 노드를 선택합니다. 이제 항목 목록이 비어 있는지 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
