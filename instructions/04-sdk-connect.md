---
lab:
  title: SDK를 사용하여 Azure Cosmos DB SQL API에 연결
  module: Module 3 - Connect to Azure Cosmos DB SQL API with the SDK
ms.openlocfilehash: df1c9fd90b00e59edbe9e078f8d99ccc043ec209
ms.sourcegitcommit: 70795561eb9e26234c0e0ce614c2e8be120135ac
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 05/28/2022
ms.locfileid: "145919971"
---
# <a name="connect-to-azure-cosmos-db-sql-api-with-the-sdk"></a>SDK를 사용하여 Azure Cosmos DB SQL API에 연결

.NET용 Azure SDK는 많은 Azure 서비스와 상호 작용하는 일관된 개발자 인터페이스를 제공하는 라이브러리 모음입니다. .NET용 Azure SDK는 .NET Standard 2.0 사양을 기반으로 하여 .NET Framework(4.6.1 이상), .NET Core(2.1 이상) 및 .NET(5 이상) 애플리케이션에서 사용할 수 있습니다.

이 랩에서는 .NET용 Azure SDK를 사용하여 Azure Cosmos DB SQL API 계정에 연결합니다.

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
    | **이 계정에서 프로비전할 수 있는 총 처리량 제한** | *선택 취소* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히 다음 사항에 주의하세요.

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="view-the-microsoftazurecosmos-library-on-nuget"></a>NuGet에서 Microsoft.Azure.Cosmos 라이브러리 보기

NuGet 웹 사이트에는 .NET 애플리케이션으로 가져올 수 있는 검색 가능한 패키지 인덱스가 포함되어 있습니다. **Microsoft.Azure.Cosmos** 와 같은 시험판 패키지를 가져오려면 NuGet 웹 사이트를 사용하여 적절한 버전 및 명령을 얻음 다음, 패키지를 애플리케이션으로 가져오면 됩니다.

1. 웹 브라우저에서 NuGet 웹 사이트(``nuget.org``)로 이동합니다.

1. NuGet, .NET용 패키지 관리자 및 해당 기능에 대한 설명을 검토합니다.

1. NuGet.org에서 **Microsoft.Azure.Cosmos** 라이브러리를 검색합니다.

1. **.NET CLI** 탭을 선택하여 이 라이브러리의 최신 버전을 .NET 프로젝트로 가져오는 데 필요한 명령을 확인합니다.

    > &#128161; 이 명령은 기록할 필요가 없습니다. 이 연습의 뒷부분에서 특정 버전의 라이브러리를 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="import-the-microsoftazurecosmos-library-into-a-net-project"></a>Microsoft.Azure.Cosmos 라이브러리를 .NET 프로젝트로 가져오기

.NET CLI에는 미리 구성된 패키지 피드에서 패키지를 가져오는 [패키지 추가][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 명령이 포함되어 있습니다. .NET 설치는 NuGet을 기본 패키지 피드로 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **04-sdk-connect** 폴더로 이동합니다.

1. **04-sdk-connect** 폴더의 컨텍스트 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **04-sdk-connect** 폴더로 이미 설정된 터미널을 엽니다.

1. 다음 명령을 사용하여 NuGet에서 [ Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 통합 터미널을 닫습니다.

## <a name="use-the-microsoftazurecosmos-library"></a>Microsoft.Azure.Cosmos 라이브러리 사용

.NET용 Azure SDK의 Azure Cosmos DB 라이브러리를 가져온 후에는 [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 네임스페이스 내에서 해당 클래스를 즉시 사용하여 Azure Cosmos DB SQL API 계정에 연결할 수 있습니다. [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] 클래스는 Azure Cosmos DB SQL API 계정에 대한 초기 연결을 설정하는 데 사용되는 핵심 클래스입니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **04-sdk-connect** 폴더로 이동합니다.

1. 빈 **script.cs** 코드 파일을 엽니다.

1. 기본 제공 **System** 및 **System.Linq** 네임스페이스에 using 블록을 추가합니다.

    ```
    using System;
    using System.Linq;
    ```

1. [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 네임스페이스에 using 블록을 추가합니다.

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. 값이 이전에 만든 Azure Cosmos DB 계정의 **endpoint** 로 설정된 **endpoint** 라는 **string** 변수를 추가합니다.
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 인 경우 C# 문은 **문자열 엔드포인트 = "https&shy;://dp420.documents.azure.com:443/"** 이 됩니다.

1. 값이 이전에 만든 Azure Cosmos DB 계정의 **key** 로 설정된 **key** 라는 **string** 변수를 추가합니다.

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; 예를 들어 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 C# 문은 **문자열 키 = "fDR2ci9QgkdkvERTQ=="** 가 됩니다.

1. 생성자의 **endpoint** 및 **key** 변수를 사용하여 [CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]형식의 **client** 라는 새 변수를 추가합니다.
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. **client** 변수의 [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] 메서드를 호출한 비동기 결과를 사용하여 [AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] 형식의 **account** 라는 새 변수를 추가합니다.

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **계정 이름** 이라는 제목의 헤더와 함께 AccountProperties 클래스의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] 속성을 인쇄합니다.

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 AccountProperties 클래스의 [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] 속성을 쿼리한 다음, 첫 번째 결과의 [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] 속성을 **기본 지역** 이라는 헤더와 함께 인쇄합니다.

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

## <a name="test-the-script"></a>스크립트 테스트

Azure Cosmos DB SQL API 계정에 연결하는 .NET 코드가 완료되었으므로 스크립트를 테스트할 수 있습니다. 이 스크립트는 계정 이름과 쓰기 가능한 첫 번째 지역의 이름을 출력합니다. 계정을 만들 때 위치를 지정했다면 이 스크립트의 결과로 동일한 위치 값이 인쇄되어야 합니다.

1. **Visual Studio Code** 에서 **04-sdk-connect** 폴더의 컨텍스트 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 이제 스크립트는 계정 이름과 쓰기 가능한 첫 번째 지역을 출력합니다. 예를 들어 계정 이름을 **dp420** 으로 지정하고 첫 번째 쓰기 가능 지역이 **미국 서부 2** 인 경우 스크립트는 다음을 출력합니다.

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
