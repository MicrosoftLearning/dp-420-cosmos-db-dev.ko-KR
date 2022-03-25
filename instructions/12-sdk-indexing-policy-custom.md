---
lab:
  title: 포털에서 Azure Cosmos DB SQL API 컨테이너의 인덱스 정책 구성
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: 1d14cf0d3c98832cb46c06178845b56b9748b0d3
ms.sourcegitcommit: 694767b3c7933a8ee84beca79da880d5874486bc
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/17/2022
ms.locfileid: "139057392"
---
# <a name="configure-an-azure-cosmos-db-sql-api-containers-index-policy-using-the-sdk"></a>SDK를 사용하여 Azure Cosmos DB SQL API 컨테이너의 인덱스 정책 구성

인덱싱 정책은 Azure Cosmos DB SDK에서 관리할 수 있습니다. .NET SDK에는 특히 Azure Cosmos DB SQL API의 컨테이너에 새 인덱싱 정책을 설계하고 푸시하는데 사용할 수 있는 클래스 집합이 포함되어 있습니다.

이 랩에서는 .NET SDK를 사용하여 컨테이너에 대한 사용자 지정 인덱싱 정책을 만듭니다.

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

## <a name="create-a-new-indexing-policy-using-the-net-sdk"></a>.NET SDK를 사용하여 새 인덱싱 정책 만들기

.NET SDK에는 코드에서 새 인덱싱 정책을 빌드하기 위한 부모 [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] 클래스와 관련된 클래스 모음이 포함되어 있습니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **12-custom-index-policy** 폴더로 이동합니다.

1. **script.cs** 코드 파일을 엽니다.

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

1. 기본 빈 생성자를 사용하여 **policy** 라는 [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] 형식의 새 변수를 만듭니다.

    ```
    IndexingPolicy policy = new ();
    ```

1. **policy** 변수의 [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]속성을 [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields] 값으로 설정합니다.

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] 속성이 _ *policy** 변수에 있는 [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] 컬렉션 속성의 **/** _ 값으로 설정된 [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] 형식의 새 개체를 추가합니다.

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. [Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] 속성이 **/name/?** 의 값으로 설정된 [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] 형식의 새 개체를 추가합니다. **policy** 변수에 있는 [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] 컬렉션 속성에 대해 다음을 수행합니다.

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. ``products`` and ``/categoryId`` 값에서 생성자 매개 변수로 전달하는 **options** 라는 [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] 형식의 새 변수를 만듭니다.

    ```
    ContainerProperties options = new ("products", "/categoryId");
    ```

1. **options** 변수의 [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] 속성에 **policy** 변수를 할당합니다.

    ```
    options.IndexingPolicy = policy;
    ```

1. **options** 변수에서 생성자 매개 변수로 전달하고 **container** 라는 [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 형식의 변수에 결과를 저장하는 **database** 변수의 [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] 메서드를 비동기적으로 호출합니다.

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **만드 컨테이너** 라는 제목의 헤더와 함께 Container 클래스의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] 속성을 인쇄합니다.

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
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
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/categoryId");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **script.cs** 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **12-custom-index-policy** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 이제 스크립트가 새로 만든 컨테이너의 이름을 출력합니다.

    ```
    Container Created [products]
    ```

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

## <a name="observe-an-indexing-policy-created-by-the-net-sdk-using-the-data-explorer"></a>데이터 탐색기를 사용하여 .NET SDK에서 만든 인덱싱 정책 관찰

다른 인덱싱 정책과 마찬가지로 데이터 탐색기를 통해 .NET SDK를 사용하여 푸시한 정책을 볼 수 있습니다. 이제 포털을 사용하여 이 랩에서 코드를 사용하여 만든 정책을 검토합니다.

1. 웹 브라우저에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리의 **products** 컨테이너 노드 내에서 **스케일링 및 설정** 을 선택합니다.

1. **인덱싱 정책** 섹션 내에서 인덱싱 정책을 확인합니다.

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; 이 랩에서 .NET SDK를 사용하여 만든 인덱싱 정책의 JSON을 표시한 것입니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
