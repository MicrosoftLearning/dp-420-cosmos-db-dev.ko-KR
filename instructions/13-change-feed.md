---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 변경 피드 이벤트 처리
  module: Module 7 - Integrate Azure Cosmos DB SQL API with Azure services
ms.openlocfilehash: 6dbff97f3a587513714610617007080ccd371778
ms.sourcegitcommit: 694767b3c7933a8ee84beca79da880d5874486bc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139057407"
---
# <a name="process-change-feed-events-using-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 변경 피드 이벤트 처리

Azure Cosmos DB SQL API 변경 피드는 플랫폼의 이벤트에 의해 구동되는 추가 애플리케이션을 만드는 데 핵심적인 요소입니다. Azure Cosmos DB SQL API용 .NET SDK는 변경 피드와 통합되는 애플리케이션을 빌드하고 컨테이너 내 작업에 대한 알림을 수신 대기하는 클래스 제품군과 함께 제공됩니다.

이 랩에서는 .NET SDK의 변경 피드 프로세서 기능을 사용하여 지정된 컨테이너의 항목에서 만들기 또는 업데이트 작업이 수행된다는 알림을 받은 애플리케이션을 만듭니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

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
    | **용량 모드** | *서버를 사용하지 않음* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히 다음 사항에 주의하세요.

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 리소스 메뉴에서 **데이터 탐색기** 를 선택합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 확장한 다음, **새 데이터베이스** 를 선택합니다.

1. **새 데이터베이스** 팝업에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | *cosmicworks* |

1. **데이터 탐색기** 창으로 돌아가서 계층 내의 **cosmicworks** 데이터베이스 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 기존 항목 사용 &vert; *cosmicworks*  |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **products** 컨테이너 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 다시 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 기존 항목 사용 &vert; *cosmicworks*  |
    | **컨테이너 ID** | *productslease* |
    | **파티션 키** | */partitionKey* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **productslease** 컨테이너 노드를 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="implement-the-change-feed-processor-in-the-net-sdk"></a>.NET SDK에서 변경 피드 프로세서 구현

**Microsoft.Azure.Cosmos.Container** 클래스는 변경 피드 프로세서를 원활하게 빌드하는 일련의 메서드와 함께 제공합니다. 시작하려면 모니터링되는 컨테이너, 임대 컨테이너 및 C\#의 대리자(각 변경 내용에 대한 일괄 처리)에 대한 참조가 필요합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **13-change-feed** 폴더로 이동합니다.

1. **product.cs** 코드 파일을 엽니다.

1. **Product** 클래스 및 해당 속성을 관찰합니다. 특히 이 랩에서는 **id** 및 **name** 속성을 사용합니다.

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

1. **client** 변수의 **GetContainer** 메서드를 사용하여 데이터베이스 이름(*cosmicworks*) 및 컨테이너 이름(*products*)을 사용하여 기존 컨테이너를 검색하고 결과를 **Container** 형식의 **sourceContainer** 라는 변수에 저장합니다.

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. **client** 변수의 **GetContainer** 메서드를 사용하여 데이터베이스 이름(*cosmicworks*) 및 컨테이너 이름(*productslease*)을 사용하여 기존 컨테이너를 검색하고 결과를 **Container** 형식의 **leaseContainer** 라는 변수에 저장합니다.

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. 두 개의 입력 매개 변수가 있는 빈 비동기 익명 함수를 사용하여 [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] 형식의 **handleChanges** 라는 새 대리자 변수를 만듭니다.

    1. **IReadOnlyCollection\<Product\>** 형식의 **changes** 라는 매개 변수입니다.

    1. **CancellationToken** 형식의 **cancellationToken** 이라는 매개 변수입니다.

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. 익명 함수 내에서 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 원시 문자열 **START\tHandling batch of changes...** 를 인쇄합니다.

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. 계속해서 익명 함수 내에서 **product** 변수를 사용하여 **changes** 변수를 반복하는 foreach 루프를 만들어 **Product** 형식의 인스턴스를 나타냅니다.

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. 익명 함수의 foreach 루프 내에서 기본 제공 비동기 **Console.WriteLineAsync** 정적 메서드를 사용하여 **product** 변수의 **id** 및 **name** 속성을 인쇄합니다.

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. foreach 루프 및 익명 함수 외부에서 다음 매개변수를 사용하여 **sourceContainer** 변수에서 [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] 호출 결과를 저장하는 **builder** 라는 새 변수를 만듭니다.

    | **매개 변수** | **값** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. 매개 변수가 **consoleApp** 인 [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] 메서드, 매개 변수가 **leaseContainer** 인 [WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] 메서드 및 [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] 형식의 **processor** 라는 변수에 결과를 저장하는 **builder** 변수에서 [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] 메서드를 원활하게 호출합니다.

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. **processor** 변수의 [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]를 비동기적으로 호출합니다.

    ```
    await processor.StartAsync();
    ```

1. 기본 제공 **Console.WriteLine** 및 **Console.ReadKey** 정적 메서드를 사용하여 콘솔에 출력을 인쇄하고 애플리케이션이 키 누름을 기다리도록 합니다.

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. **processor** 변수의 [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]를 비동기적으로 호출합니다.

    ```
    await processor.StopAsync();
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **script.cs** 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **13-change-feed** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. **Visual Studio Code** 와 터미널을 모두 열어 둡니다.

    > &#128221; 다른 도구를 사용하여 Azure Cosmos DB SQL API 컨테이너에서 항목을 생성합니다. 항목을 생성하면 이 터미널로 돌아가 출력을 확인합니다. 터미널을 너무 일찍 닫지 마세요.

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>샘플 데이터를 사용하여 Azure Cosmos DB SQL API 계정 시드

**Cosmicworks** 데이터베이스 및 **products** 컨테이너를 만드는 명령줄 유틸리티를 사용합니다. 그런 다음, 도구는 터미널 창에서 실행되는 변경 피드 프로세서를 사용하여 관찰할 항목 집합을 만듭니다.

1. **Visual Studio Code** 에서 **터미널** 메뉴를 연 다음, **분할 터미널** 을 선택하여 기존 인스턴스와 나란히 새 터미널을 엽니다.

1. 컴퓨터에서 전역으로 사용할 수 있는 [cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구를 설치합니다.

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; 이 명령을 완료하는 데 몇 분 정도 걸릴 수 있습니다. 이 명령은 과거에 이 도구의 최신 버전을 이미 설치한 경우 경고 메시지(*’cosmicworks’ 도구는 이미 설치되어 있습니다')를 출력합니다.

1. cosmicworks를 실행하여 다음 명령줄 옵션을 사용하여 Azure Cosmos DB 계정을 시드합니다.

    | **옵션** | **값** |
    | ---: | :--- |
    | **--endpoint** | 이 랩에서 이전에 복사한 엔드포인트 값 |
    | **--key** | 이 랩에서 이전에 복사한 키 값 |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 이고 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 명령은 ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``가 됩니다.

1. **cosmicworks** 명령이 데이터베이스, 컨테이너 및 항목으로 계정을 채울 때까지 기다립니다.

1. .NET 애플리케이션에서 터미널 출력을 관찰합니다. 터미널에는 변경 피드를 사용하여 전송된 각 변경 내용에 대해 **검색된 작업** 이라는 메시지가 출력됩니다.

1. 두 통합 터미널을 모두 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
