---
lab:
  title: SDK를 사용하여 UDF 구현 및 사용
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB SQL API
ms.openlocfilehash: 638c9b822f2ed8f1eae1e6ac1be36585804abf1c
ms.sourcegitcommit: 694767b3c7933a8ee84beca79da880d5874486bc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139057399"
---
# <a name="implement-and-then-use-a-udf-using-the-sdk"></a>SDK를 사용하여 UDF 구현 및 사용

Azure Cosmos DB SQL API용 .NET SDK를 사용하여 컨테이너에서 직접 서버 쪽 프로그래밍 구문을 관리하고 호출할 수 있습니다. 새 컨테이너를 준비할 때 데이터 탐색기를 사용하여 수동으로 작업을 수행하는 대신 .NET SDK를 사용하여 UDF를 컨테이너에 직접 게시하는 것이 합리적일 수 있습니다.

이 랩에서는 .NET SDK를 사용하여 새 UDF를 만든 다음, 데이터 탐색기를 사용하여 UDF가 올바르게 작동하는지 확인합니다.

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

1. 머신에서 전역으로 사용할 수 있는 [cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구를 설치합니다.

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

1. 통합 터미널을 닫습니다.

## <a name="create-a-user-defined-function-udf-using-the-net-sdk"></a>.NET SDK를 사용하여 UDF(user-defined function) 만들기

.NET SDK의 [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 클래스에는 SDK에서 직접 저장 프로시저, UDF 및 트리거에 대해 CRUD 작업을 수행하는 데 사용되는 [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] 속성이 포함되어 있습니다. 이 속성을 사용하여 새 UDF를 만든 다음, 해당 UDF를 Azure Cosmos DB SQL API 컨테이너에 푸시합니다. SDK를 사용하여 만들 UDF는 세금을 포함하여 제품의 가격을 계산하므로, 세금이 포함된 가격을 사용하여 제품에 대한 SQL 쿼리를 실행할 수 있습니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **33-create-use-udf-sdk** 폴더로 이동합니다.

1. **script.cs** 코드 파일을 엽니다.

1. [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] 네임스페이스에 using 블록을 추가합니다.

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

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

1. 기본 빈 생성자를 사용하여 props라는 [UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] 형식의 새 변수를 만듭니다.

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. **props** 변수의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] 속성을 **tax** 값으로 설정합니다.

    ```
    props.Id = "tax";
    ```

1. **props** 변수의 [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] 속성을 **props.Body = "function tax(i) { return i * 1.25; }";** 값으로 설정합니다.

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. **container** 변수의 [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] 메서드를 비동기적으로 호출하여 **props** 변수를 매개 변수로 전달하고 결과를 [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse] 형식의 **udf** 라는 변수에 저장합니다.

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **만든 UDF** 라는 헤더와 함께 UserDefinedFunctionResponse 클래스의 [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] 속성을 인쇄합니다.

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClientOptions clientoptions = new CosmosClientOptions()
    {
        RequestTimeout = new TimeSpan(0,0,90)
        , OpenTcpConnectionTimeout = new TimeSpan (0,0,90)
    };

    CosmosClient client = new CosmosClient(endpoint, key, clientoptions);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **script.cs** 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **33-create-use-udf-sdk** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 이제 스크립트가 새로 만든 UDF의 이름을 출력합니다.

    ```
    Created UDF [tax]
    ```

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

## <a name="test-the-udf-using-the-data-explorer"></a>데이터 탐색기를 사용하여 UDF 테스트

이제 Azure Cosmos DB 컨테이너에 새 UDF가 만들어졌으므로 데이터 탐색기를 사용하여 UDF가 예상대로 작동하는지 확인합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 쿼리 탭에서 필터 없이 모든 항목을 선택하는 표준 쿼리를 보려면 **쿼리 실행** 을 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 두 개의 가격 값이 예상되는 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다. 첫 번째 값은 컨테이너의 원시 가격 값이고 두 번째 값은 UDF에서 계산한 가격 값입니다.

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. **쿼리 실행** 을 선택합니다.

1. 문서를 관찰하고 **price** 및 **priceWithTax** 필드를 비교합니다.

    > &#128221; **priceWithTax** 필드에는 **price** 필드보다 25% 더 큰 값이 있어야 합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
