---
lab:
  title: Azure Cognitive Search 및 Azure Cosmos DB SQL API를 사용하여 데이터 검색
  module: Module 7 - Integrate Azure Cosmos DB SQL API with Azure services
ms.openlocfilehash: e61608396e31d7892168cbf29086cb16be525087
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025034"
---
# <a name="search-data-using-azure-cognitive-search-and-azure-cosmos-db-sql-api"></a>Azure Cognitive Search 및 Azure Cosmos DB SQL API를 사용하여 데이터 검색

Azure Cognitive Search는 검색 인덱스의 정보를 보강하기 위해 AI 기능과 긴밀하게 통합하여 Search Engine as a Service를 결합합니다.

이 랩에서는 Azure Cosmos DB SQL API 컨테이너의 데이터를 자동으로 인덱싱하고 Azure Cognitive Services 번역기 기능을 사용하여 데이터를 보강하는 Azure Cognitive Search 인덱스를 빌드합니다.

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

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

    1. **기본 연결 문자열** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **연결 문자열** 값을 사용합니다.

1. 리소스 메뉴에서 **데이터 탐색기** 를 선택합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 새 &vert; cosmicworks 만들기  |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **products** 컨테이너 노드를 관찰합니다.

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

## <a name="create-azure-cognitive-search-resource"></a>Azure Cognitive Search 리소스 만들기

이 연습을 계속하기 전에, 먼저 새 Azure Cognitive Search 인스턴스를 만들어야 합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **+ 리소스 만들기** 를 선택하고 *Cognitive Search* 를 검색한 다음, 다음 설정을 사용하여 새 **Azure Cognitive Search** 계정 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | 기존 Azure 구독 |
    | **리소스 그룹** | 기존 리소스 그룹을 선택하거나 새로 만듭니다. |
    | **이름** | 전역적으로 고유한 이름을 입력합니다. |
    | **위치** | 사용 가능한 지역을 선택합니다. |
    | **가격 책정 계층** | *Free* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cognitive Search** 계정 리소스로 이동합니다.

## <a name="build-indexer-and-index-for-azure-cosmos-db-sql-api-data"></a>Azure Cosmos DB SQL API 데이터에 대한 인덱서 및 인덱스 빌드

특정 Azure Cosmos DB SQL API 컨테이너에서 데이터 하위 집합을 시간 단위로 인덱싱하는 인덱서를 만듭니다.

1. **Azure Cognitive Search** 리소스 블레이드에서 **데이터 가져오기** 를 선택합니다.

1. **데이터 가져오기** 마법사의 **데이터에 연결**, **데이터 원본** 목록에서 **Azure Cosmos DB** 를 선택합니다.

1. 다음 설정을 사용하여 데이터 원본을 구성하고 나머지 모든 설정은 기본값으로 유지합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터 원본 이름** | *products-cosmossql-source* |
    | **연결 문자열** | 이전에 만든 Azure Cosmos DB SQL API 계정의 연결 문자열 |
    | **데이터베이스** | *cosmicworks* |
    | **컬렉션** | *products* |

1. **쿼리** 필드에 다음 SQL 쿼리를 입력하여 컨테이너에 있는 데이터 하위 집합에 대한 구체화된 뷰를 만듭니다.

    ```
    SELECT 
        p.id, 
        p.categoryId, 
        p.name, 
        p.price,
        p._ts
    FROM 
        products p 
    WHERE 
        p._ts > @HighWaterMark 
    ORDER BY 
        p._ts
    ```

1. **_ts를 기준으로 정렬된 쿼리 결과** 확인란을 선택합니다.

    > &#128221; 이 확인란을 통해 Azure Cognitive Search는 쿼리가 **_ts** 필드를 기준으로 결과를 정렬한다는 것을 알 수 있습니다. 이런 정렬 형식을 사용하면 증분 진행률을 추적할 수 있습니다. 결과가 타임스탬프를 기준으로 정렬되므로 인덱서가 실패하면 동일한 **_ts** 값에서 바로 백업을 선택할 수 있습니다.

1. **다음: 인식 기술 추가** 를 선택합니다.

1. **다음: 대상 인덱스 사용자 지정** 을 선택합니다.

1. 마법사의 **대상 인덱스 사용자 지정** 단계에서 다음 설정으로 인덱스를 구성하고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **인덱스 이름** | *products-index* |
    | **키** | *id* |

1. 필드 테이블에서 다음 표를 사용하여 각 필드의 **조회 가능**, **필터링 가능**, **정렬 가능**, **패싯 가능** 및 **검색 가능** 옵션을 구성합니다.

    | **필드** | **조회 가능** | **필터 가능** | **정렬 가능** | **패싯 가능** | **검색 가능** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **categoryId** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004;(영어 - Microsoft) |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. **다음: 인덱서 만들기** 를 선택합니다.

1. 마법사의 **인덱서 만들기** 단계에서 다음 설정으로 인덱서를 구성하고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **이름** | *products-cosmosdb-indexer* |
    | **일정** | *시간별* |

1. **제출** 을 선택하여 데이터 원본, 인덱스 및 인덱서를 만듭니다.

    > &#128221; 첫 번째 인덱서가 만들어지면 설문 조사 팝업을 해제해야 할 수 있습니다.

1. **Azure Cognitive Search** 리소스 블레이드에서 **인덱서** 탭으로 이동하여 첫 번째 인덱싱 작업의 결과를 확인합니다.

1. **products-cosmosdb-indexer** 인덱서의 상태가 **성공** 이 될 때까지 기다린 후 이 작업을 계속 진행합니다.

    > &#128221; 블레이드가 자동으로 업데이트되지 않는 경우 **새로 고침** 옵션을 사용하여 블레이드를 업데이트해야 할 수 있습니다.

1. **인덱스** 탭으로 이동한 다음, **products-index** 인덱스를 선택합니다.

## <a name="validate-index-with-example-search-queries"></a>예제 검색 쿼리를 사용하여 인덱스 유효성 검사

이제 Azure Cosmos DB SQL API 데이터의 구체화된 뷰가 검색 인덱스에 있으므로 Azure Cognitive Search 기능을 활용하는 몇 가지 기본 쿼리를 수행할 수 있습니다.

> &#128221; 이 랩은 Azure Cognitive Search 구문을 학습하기 위한 것이 아닙니다. 이러한 쿼리는 검색 인덱스 및 엔진에서 사용할 수 있는 몇 가지 기능을 보여 주기 위해 큐레이팅되었습니다.

1. **products-index** &vert; **인덱스** 창에서 **검색** 을 선택한 후 **\*** (와일드카드) 연산자를 사용하여 가능한 모든 결과를 반환하는 기본 검색 쿼리를 실행합니다.

1. 이 검색 쿼리가 가능한 모든 결과를 반환하는지 확인합니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    touring 3000
    ```

1. 이 검색 쿼리가 두 용어를 모두 포함하는 결과에 더 높은 점수를 주는 **touring** 이라는 용어나 **3000** 을 포함하는 결과를 반환하는지 확인합니다. 그러면 결과는 **@search.score** 필드를 기준으로 내림차순으로 정렬됩니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    red&$count=true
    ```

1. 이 검색 쿼리가 **red** 라는 용어로 결과를 반환하지만, 동일한 페이지에 모두 포함되지 않더라도 결과의 총 수를 나타내는 메타데이터 필드도 포함하는지 확인합니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    blue&$count=true&$top=6
    ```

1. 이 검색 쿼리가 서버 쪽과 일치하는 항목이 더 있더라도 한 번에 6개의 결과 집합만 반환하는지 확인합니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    mountain&$count=true&$top=25&$skip=50
    ```

1. 이 검색 쿼리가 처음 50개의 결과를 건너뛰고 25개의 결과 집합을 반환하는지 확인합니다. 클라이언트 쪽 애플리케이션에서 페이지를 매긴 뷰인 경우 결과의 세 번째 “페이지”가 될 것이라고 유추할 수 있습니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    touring&$count=true&$filter=price lt 500
    ```

1. 이 검색 쿼리가 숫자 가격 필드의 값이 500보다 작은 결과만 반환하는지 확인합니다.

1. **쿼리 문자열** 편집기에서 다음 쿼리를 입력한 다음 **검색** 을 선택합니다.

    ```
    road&$count=true&$top=15&facet=price,interval:500
    ```

1. 이 검색 쿼리가 현재 결과 페이지에 모두 없는 경우에도 각 범주에 속하는 항목 수를 나타내는 패싯 데이터 컬렉션을 반환하는지 확인합니다. 이 예제에서 일치하는 항목은 500 간격의 숫자 가격 범주로 세분화됩니다. 일반적으로 클라이언트 쪽 애플리케이션에서 필터 및 탐색 보조 기능을 채우는 데 사용됩니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
