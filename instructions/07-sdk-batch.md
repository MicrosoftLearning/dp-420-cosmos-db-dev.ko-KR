---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 여러 지점 작업을 대량 처리
  module: Module 4 - Access and manage data with the Azure Cosmos DB SQL API SDKs
ms.openlocfilehash: 12d88545282991a14b758d05d27625b633edc7a3
ms.sourcegitcommit: 9e320ed456eaaab98e80324267c710628b557b1c
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139039332"
---
# <a name="batch-multiple-point-operations-together-with-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 여러 지점 작업을 대량 처리

[TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] 및 [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] 클래스는 작업을 단일 논리 단계로 구성하고 분해하는 데 핵심적인 요소입니다. 이러한 클래스를 사용하여 코드를 작성하여 여러 작업을 수행한 다음, 서버 쪽에서 성공적으로 완료되었는지 확인할 수 있습니다.

이 랩에서는 SDK를 사용하여 두 개의 항목을 단일 논리 단위로 만들려고 시도하는 두 개의 이중 항목 작업을 수행합니다.

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

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **07-sdk-batch** 폴더로 이동합니다.

1. **07-sdk-batch** 폴더 내에서 **script.cs** 코드 파일을 엽니다.

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

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **07-sdk-batch** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **07-sdk-batch** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

1. 통합 터미널을 닫습니다.

## <a name="creating-a-transactional-batch"></a>트랜잭션 일괄 처리 만들기

먼저 두 개의 가상 제품을 생성하는 간단한 트랜잭션 일괄 처리를 만들어 보겠습니다. 이 일괄 처리는 동일한 "중고 액세서리" 범주 식별자를 사용하여 낡은 안장과 녹슨 핸들바를 컨테이너에 삽입합니다. 두 항목 모두 동일한 논리 파티션 키를 가지므로 성공적인 일괄 처리 작업을 수행할 수 있습니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 고유 식별자가 **0120**, 이름이 **Worn Saddle**, 범주 식별자가 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 인 **saddle** 이라는 **Product** 변수를 만듭니다.

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 고유 식별자가 **012A**, 이름이 **Rusty Handlebar**, 범주 식별자가 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 인 **handlebar** 라는 **Product** 변수를 만듭니다.

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 생성자 매개 변수로서 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 을 전달하는 **partitionKey** 라는 **PartitionKey** 형식의 변수를 만듭니다.

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 메서드 매개 변수로서 **partitionkey** 변수를 전달하고 항목으로서 **saddle** 및 **handlebar** 변수를 전달하는 [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] 제네릭 메서드를 호출하는 흐름 구문을 사용하는 **container** 변수의 [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] 메서드를 호출하여 개별 작업에서 만들고 결과를 **TransactionalBatch** 형식의 **batch** 라는 변수에 저장합니다.

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. using 문 내에서 **batch** 변수의 **ExecuteAsync** 메서드를 비동기적으로 호출하고 결과를 **response** 라는 **TransactionalBatchResponse** 형식의 변수에 저장합니다.

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하여 **response** 변수의 **StatusCode** 속성 값을 출력합니다.

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
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

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **07-sdk-batch** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 상태 코드는 HTTP 200 **OK** 여야 합니다.

1. 통합 터미널을 닫습니다.

## <a name="creating-an-errant-transactional-batch"></a>잘못된 트랜잭션 일괄 처리 만들기

이제 의도적으로 오류를 발생시키는 트랜잭션 일괄 처리를 만들어 보겠습니다. 이 일괄 처리는 논리 파티션 키가 다른 두 항목을 삽입하려고 시도합니다. "중고 액세서리" 범주에 깜박이는 스트로브 조명과 "새 액세서리" 범주에 새로운 헬멧을 만들 것입니다. 정의에 따르면 이것은 잘못된 요청이어야 하며 이 트랜잭션을 수행할 때 오류를 반환해야 합니다.

1. **script.cs** 코드 파일의 편집기 탭으로 돌아갑니다.

1. 다음 코드 줄을 삭제합니다.

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 고유 식별자가 **012B**, 이름이 **Flickering Strobe Light**, 범주 식별자가 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 인 **light** 라는 **Product** 변수를 만듭니다.

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 고유 식별자가 **012C**, 이름이 **New Helmet**, 범주 식별자가 **0feee2e4-687a-4d69-b64e-be36afc33e74** 인 **helmet** 이라는 **Product** 변수를 만듭니다.

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. 생성자 매개 변수로서 **9603ca6c-9e28-4a02-9194-51cdb7fea816** 을 전달하는 **partitionKey** 라는 **PartitionKey** 형식의 변수를 만듭니다.

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 메서드 매개 변수로서 **partitionkey** 변수를 전달하고 항목으로서 **light** 및 **helmet** 변수를 전달하는 **CreateItem<>** 제네릭 메서드를 호출하는 흐름 구문을 사용하는 **container** 변수의 **CreateTransactionalBatch** 메서드를 호출하여 개별 작업에서 만들고 결과를 **TransactionalBatch** 형식의 **batch** 라는 변수에 저장합니다.

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. using 문 내에서 **batch** 변수의 **ExecuteAsync** 메서드를 비동기적으로 호출하고 결과를 **response** 라는 **TransactionalBatchResponse** 형식의 변수에 저장합니다.

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 정적 **Console.WriteLine** 메서드를 호출하여 **response** 변수의 **StatusCode** 속성 값을 출력합니다.

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
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

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **07-sdk-batch** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 상태 코드는 HTTP 400 **Bad Request** 또는 409 **Conflict** 이어야 합니다. 트랜잭션 내의 모든 항목이 트랜잭션 일괄 처리와 동일한 파티션 키 값을 공유하지 않았기 때문에 이 문제가 발생했습니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
