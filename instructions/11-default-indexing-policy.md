---
lab:
  title: 포털에서 Azure Cosmos DB SQL API 컨테이너에 대한 기본 인덱스 정책 검토
  module: Module 6 - Define and implement an indexing strategy for Azure Cosmos DB SQL API
ms.openlocfilehash: a5918d41746f82da08d66c486aa53f782d9e8e17
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025087"
---
# <a name="review-the-default-index-policy-for-an-azure-cosmos-db-sql-api-container-with-the-portal"></a>포털에서 Azure Cosmos DB SQL API 컨테이너에 대한 기본 인덱스 정책 검토

Azure Cosmos DB의 모든 컨테이너에는 컨테이너 내에서 항목을 인덱싱하는 방법에 대한 서비스를 안내하는 인덱싱 정책이 있습니다. 기본적으로 이 인덱싱 정책은 모든 항목의 모든 속성을 인덱싱합니다. 기본 인덱싱 정책을 사용하면 프로젝트를 시작할 때 인덱싱, 성능, 관리에 대해 생각할 필요 없이 Azure Cosmos DB를 빠르고 쉽게 시작할 수 있습니다.

이 랩에서는 데이터 탐색기를 사용하여 일부 컨테이너에 대한 기본 인덱스 정책을 관찰하고 조작합니다.

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

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="seed-the-azure-cosmos-db-sql-api-account-with-data"></a>Azure Cosmos DB SQL API 계정을 데이터와 함께 시드

[cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구는 샘플 데이터를 Azure Cosmos DB SQL API 계정에 배포합니다. 이 도구는 오픈 소스이며 NuGet를 통해 구할 수 있습니다. 이 도구를 Azure Cloud Shell에 설치한 다음, 이를 사용하여 데이터베이스를 시드합니다.

1. **Visual Studio Code** 를 시작합니다.

1. **Visual Studio Code** 에서 **터미널** 메뉴를 연 다음, **새 터미널** 을 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

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

1. **Visual Studio Code** 를 닫습니다.

## <a name="view-and-manipulate-the-default-indexing-policy"></a>기본 인덱싱 정책 보기 및 조작

코드, 포털 또는 도구로 컨테이너를 만들 때 인덱싱 정책을 별도로 지정하지 않으면 지능형 기본값으로 설정됩니다. 기본 인덱싱 정책을 관찰하고 정책을 변경합니다.

1. 웹 브라우저에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, 새 **SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **이름** 이 **HL Headset** 과 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리 결과를 관찰합니다.

1. **쿼리** 탭에서 **쿼리 통계** 를 선택합니다.

1. **쿼리** 탭에서 **쿼리 통계** 섹션 내의 **요청 요금** 필드 값을 확인합니다.

    > &#128221; 모든 경로가 현재 인덱싱되어 있으므로 이 쿼리는 비교적 효율적이어야 합니다.

1. **SQL API** 탐색 트리의 **products** 컨테이너 노드 내에서 **스케일링 및 설정** 을 선택합니다.

1. **인덱싱 정책** 섹션 내에서 기본 인덱싱 정책을 관찰합니다.

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

    > &#128221; 이 기본 정책은 **_etag** 를 제외하고 가능한 모든 경로를 인덱싱합니다.

1. 편집기 내에서 인덱싱 정책의 콘텐츠를 **/price** 경로만 인덱싱하도록 바꿉니다.

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. **저장** 을 선택하여 변경 내용을 유지합니다.

1. **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **이름** 이 **HL Headset** 과 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리 결과를 관찰합니다.

1. **쿼리** 탭에서 **쿼리 통계** 를 선택합니다.

1. **쿼리** 탭에서 **쿼리 통계** 섹션 내의 **요청 요금** 필드 값을 확인합니다.

    > &#128221; **name** 속성이 인덱싱되지 않아 요청 요금이 증가했습니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **가격** 이 **$3,000** 보다 큰 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리 결과를 관찰합니다.

1. **쿼리** 탭에서 **쿼리 통계** 를 선택합니다.

1. **쿼리** 탭에서 **쿼리 통계** 섹션 내의 **요청 요금** 필드 값을 확인합니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
