---
lab:
  title: 쓰기 작업에 대한 Azure Cosmos DB SQL API 컨테이너 인덱싱 정책 최적화
  module: Module 10 - Optimize query performance in Azure Cosmos DB SQL API
ms.openlocfilehash: d83af5c3783a7502a2f2e220b3b2ae566c0338fe
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025082"
---
# <a name="optimize-an-azure-cosmos-db-sql-api-containers-indexing-policy-for-write-operations"></a>쓰기 작업에 대한 Azure Cosmos DB SQL API 컨테이너의 인덱싱 정책 최적화

쓰기가 많은 워크로드 또는 대규모 JSON 개체가 있는 워크로드의 경우 인덱싱 정책을 최적화하여 쿼리에서 사용할 속성만 인덱싱하는 것이 유리할 수 있습니다.

이 랩에서는 테스트 .NET 애플리케이션을 사용하여 기본 인덱싱 정책을 사용한 다음 약간 조정된 인덱싱 정책을 사용하여 Azure Cosmos DB SQL API 컨테이너에 큰 JSON 항목을 삽입합니다.

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
    | **용량 모드** | *서버를 사용하지 않음* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 페이지에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | *새로 만들기* &vert; *cosmicworks* |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **제품** 컨테이너 노드를 관찰합니다.

1. 리소스 블레이드에서 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="run-the-test-net-application-using-the-default-indexing-policy"></a>기본 인덱싱 정책을 사용하여 테스트 .NET 애플리케이션 실행

이 랩에는 큰 JSON 개체를 사용하고 Azure Cosmos DB SQL API 컨테이너에 새 항목을 만드는 미리 빌드된 테스트 .NET 애플리케이션이 있습니다. 단일 쓰기 작업이 완료되면 애플리케이션은 항목의 고유 식별자 및 RU 요금을 콘솔 창에 출력합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **23-index-optimization** 폴더로 이동합니다.

1. **23-index-optimization** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **23-index-optimization** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] 명령을 사용하여 프로젝트를 빌드합니다.

    ```
    dotnet build
    ```

    > &#128221; **endpoint** 및 **key** 변수가 현재 사용되지 않는다는 컴파일러 경고가 표시될 수 있습니다. 이 작업에서 이러한 변수를 사용하게 되므로 이 경고를 무시해도 됩니다.

1. 통합 터미널을 닫습니다.

1. **script.cs** 코드 파일을 엽니다.

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

1. **Visual Studio Code** 에서 **23-index-optimization** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 터미널의 출력을 관찰합니다. 항목의 고유 식별자 및 작업의 요청 요금(RU)을 콘솔에 인쇄해야 합니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 두 번 이상 빌드하고 실행합니다. 콘솔 출력에서 RU 요금을 관찰합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 열어 둡니다.

    > &#128221; 이 연습의 뒷부분에서 이 터미널을 다시 사용합니다. 원래 RU 요금과 업데이트된 RU 요금을 비교할 수 있도록 터미널을 열어 두는 것이 중요합니다.

## <a name="update-the-indexing-policy-and-rerun-the-net-application"></a>인덱싱 정책 업데이트 및 .NET 애플리케이션 다시 실행

이 랩 시나리오에서는 향후 쿼리가 주로 이름 및 categoryName 속성에 초점을 맞춘다고 가정합니다. 큰 JSON 항목에 최적화하려면 모든 경로를 제외하여 시작하는 인덱싱 정책을 만들어 인덱스에서 다른 모든 필드를 제외합니다. 그런 다음, 정책에 특정 경로가 선택적으로 포함됩니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 확장한 다음, **설정** 을 관찰합니다.

1. **설정** 탭에서 **인덱싱 정책** 섹션으로 이동합니다.

1. 기본 인덱싱 정책을 관찰합니다.

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }    
    ```

1. 인덱싱 정책을 수정된 JSON 개체로 바꾼 다음, 변경 내용을 **저장** 합니다.

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        },
        {
          "path": "/categoryName/?"
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

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 로 돌아갑니다. 열린 터미널로 돌아갑니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 두 번 이상 빌드하고 실행합니다. 콘솔 출력에서 새 RU 요금을 관찰합니다. 이 요금은 원래 요금보다 훨씬 적어야 합니다.

    ```
    dotnet run
    ```

    > &#128221; 업데이트된 RU 요금이 표시되지 않는 경우 몇 분 정도 기다려야 할 수 있습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 확장한 다음, **설정** 을 관찰합니다.

1. **설정** 탭에서 **인덱싱 정책** 섹션으로 이동합니다.

1. 인덱싱 정책을 수정된 JSON 개체로 바꾼 다음, 변경 내용을 **저장** 합니다.

    ```
    {
      "indexingMode": "none"
    }
    ```

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 로 돌아갑니다. 열린 터미널로 돌아갑니다.

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** 명령을 사용하여 프로젝트를 두 번 이상 빌드하고 실행합니다. 콘솔 출력에서 새 RU 요금을 관찰합니다. 이 요금은 원래 요금보다 약간 적어야 합니다.

    ```
    dotnet run
    ```

    > &#128221; 업데이트된 RU 요금이 표시되지 않는 경우 몇 분 정도 기다려야 할 수 있습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
