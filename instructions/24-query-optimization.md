---
lab:
  title: 쿼리에 대한 Azure Cosmos DB SQL API 컨테이너 인덱싱 정책 최적화
  module: Module 10 - Optimize query performance in Azure Cosmos DB SQL API
ms.openlocfilehash: 3556b5f9b8a3129d92dcccf65a54d6adf1ccbf32
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025165"
---
# <a name="optimize-an-azure-cosmos-db-sql-api-containers-indexing-policy-for-a-query"></a>쿼리에 대한 Azure Cosmos DB SQL API 컨테이너의 인덱싱 정책 최적화

Azure Cosmos DB SQL API 계정을 계획할 때 가장 많이 사용되는 쿼리를 알고 있으면 가능한 한 성능을 발휘하도록 인덱싱 정책을 조정하는 데 도움이 될 수 있습니다.

이 랩에서는 데이터 탐색기를 사용하여 기본 인덱싱 정책 및 복합 인덱스가 포함된 인덱싱 정책으로 SQL 쿼리를 테스트합니다.

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
    | **용량 모드** | *서버를 사용하지 않음* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 새 &vert; cosmicworks 만들기  |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **products** 컨테이너 노드를 관찰합니다.

1. 리소스 블레이드에서 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>샘플 데이터를 사용하여 Azure Cosmos DB SQL API 계정 시드

**Cosmicworks** 데이터베이스 및 **products** 컨테이너를 만드는 명령줄 유틸리티를 사용합니다. 그런 다음 도구는 터미널 창에서 실행되는 변경 피드 프로세서를 사용하여 관찰할 항목 집합을 만듭니다.

1. **Visual Studio Code** 에서 **터미널** 메뉴를 연 다음, **분할 터미널** 을 선택하여 기존 인스턴스와 나란히 새 터미널을 엽니다.

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

## <a name="execute-sql-queries-and-measure-their-request-unit-charge"></a>SQL 쿼리를 실행하고 요청 단위 요금 측정

인덱싱 정책을 수정하기 전에 먼저 몇 가지 샘플 SQL 쿼리를 실행하여 RU로 표현된 기준 요청 단위 요금을 가져옵니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. **쿼리 실행** 을 선택하여 기본 쿼리를 실행합니다.

    ```
    SELECT * FROM c
    ```

1. 쿼리 결과를 관찰합니다. **쿼리 통계** 를 선택하여 RU에서 요청 단위 요금을 확인합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **이름** 이 **HL Headset** 과 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p    
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리의 결과 및 통계를 관찰합니다. 요청 단위 요금은 첫 번째 쿼리와 거의 동일합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **이름** 이 **HL Headset** 과 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리의 결과 및 통계를 관찰합니다. **ORDER BY** 절로 인해 요청 단위 요금이 증가했습니다.

## <a name="create-a-composite-index-in-the-indexing-policy"></a>인덱싱 정책에서 복합 인덱스 만들기

이제 여러 속성을 사용하여 항목을 정렬하는 경우 복합 인덱스 만들어야 합니다. 이 작업에서는 categoryName으로 항목을 정렬한 다음 실제 이름으로 항목을 정렬하는 복합 인덱스를 만듭니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 먼저 **categoryName** 에 의한 결과를 내림차순으로 정렬한 다음, **price** 에 의한 결과를 오름차순으로 정렬하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리는 **쿼리별 순서에 따라 처리할 수 있는 해당 복합 인덱스가 없습니다** 라는 오류로 인해 실패하게 됩니다.

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
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 먼저 **categoryName** 에 의한 결과를 내림차순으로 정렬한 다음, **price** 에 의한 결과를 오름차순으로 정렬하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리의 결과 및 통계를 관찰합니다. 복합 인덱스가 포함되었으므로 요청 단위 요금은 더 작아야 합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 먼저 **categoryName** 에 의한 결과를 내림차순으로 정렬하고 **name** 에 의한 결과를 오름차순으로 정렬한 다음, 마지막으로 **price** 에 의한 결과를 오름차순으로 정렬하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리의 결과 및 통계를 관찰합니다. 쿼리의 복잡성 및 지원 복합 인덱스의 부족으로 인해 요청 단위 요금이 다시 더 높아집니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 확장한 다음, **설정** 을 다시 선택합니다.

1. **설정** 탭에서 **인덱싱 정책** 섹션으로 이동합니다.

1. 인덱싱 정책을 수정된 JSON 개체로 바꾼 다음, 변경 내용을 **저장** 합니다.

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 먼저 **categoryName** 에 의한 결과를 내림차순으로 정렬하고 **name** 에 의한 결과를 오름차순으로 정렬한 다음, 마지막으로 **price** 에 의한 결과를 오름차순으로 정렬하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리의 결과 및 통계를 관찰합니다. 복합 인덱스가 포함되었으므로 요청 단위 요금은 더 낮아야 합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
