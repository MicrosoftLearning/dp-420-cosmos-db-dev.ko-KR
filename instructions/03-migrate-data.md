---
lab:
  title: Azure Data Factory를 사용하여 기존 데이터 마이그레이션
  module: Module 2 - Plan and implement Azure Cosmos DB SQL API
ms.openlocfilehash: 0ea95a45f5e20ef089d712939fde0c3d8767601e
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025039"
---
# <a name="migrate-existing-data-using-azure-data-factory"></a>Azure Data Factory를 사용하여 기존 데이터 마이그레이션

Azure Data Factory에서 Azure Cosmos DB는 데이터 수집 원본 및 데이터 출력의 대상(싱크)으로 지원됩니다.

이 랩에서는 유용한 명령줄 유틸리티를 사용하여 Azure Cosmos DB를 채운 다음, Azure Data Factory를 사용하여 한 컨테이너에서 다른 컨테이너로 데이터 하위 집합을 이동합니다.

## <a name="create-and-seed-your-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기 및 시드

초당 **4,000** 개 요청 단위(RU/s)로 **cosmicworks** 데이터베이스 및 **products** 컨테이너를 만드는 명령줄 유틸리티를 사용합니다. 만든 후에는 처리량을 400RU/s로 조정합니다.

제품 컨테이너와 함께 사용하려면 이 랩이 끝날 때 ETL 변환 및 로드 작업의 대상이 될 **flatproducts** 컨테이너를 수동으로 만듭니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **+ 리소스 만들기** 를 선택하고 *Cosmos DB* 를 검색한 다음, 다음 설정을 사용하여 새 **Azure Cosmos DB SQL API** 계정 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | *기존 Azure 구독* |
    | **리소스 그룹** | *기존 리소스 그룹을 선택하거나 새로 만듭니다.* |
    | **계정 이름** | *전역적으로 고유한 이름을 입력합니다.* |
    | **위치** | *사용 가능한 지역을 선택합니다.* |
    | **용량 모드** | *프로비전된 처리량* |
    | **무료 계층 할인 적용** | *적용 안 함* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. **Visual Studio Code** 시작

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. **Visual Studio Code** 에서 **터미널** 메뉴를 연 다음, **새 터미널** 을 선택하여 새 터미널 인스턴스를 엽니다.

1. 컴퓨터에서 전역으로 사용할 수 있는 [cosmicworks][nuget.org/packages/cosmicworks] 명령줄 도구를 설치합니다.

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; 이 명령을 완료하는 데 몇 분 정도 걸릴 수 있습니다. 이 명령은 과거에 이 도구의 최신 버전을 이미 설치한 경우 경고 메시지(*'cosmicworks' 도구는 이미 설치되어 있습니다')를 출력합니다.

1. cosmicworks를 실행하여 다음 명령줄 옵션을 사용하여 Azure Cosmos DB 계정을 시드합니다.

    | **옵션** | **값** |
    | ---: | :--- |
    | **--endpoint** | *이 랩에서 이전에 복사한 엔드포인트 값* |
    | **--key** | *이 랩에서 이전에 복사한 키 값* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 이고 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 명령은 ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``가 됩니다.

1. **cosmicworks** 명령이 데이터베이스, 컨테이너 및 항목으로 계정을 채울 때까지 기다립니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **products** 컨테이너 노드를 확장한 다음, **항목** 을 관찰합니다.

1. **products** 컨테이너에서 다양한 JSON 항목을 관찰하고 선택합니다. 이전 단계에서 사용된 명령줄 도구로 만든 항목입니다.

1. **크기 조정 및 설정** 노드를 선택합니다. **크기 조정 및 설정** 탭에서 **수동** 을 선택하고 **필수 처리량** 설정을 **4000RU/s** 에서 **400RU/s** 로 업데이트한 다음, 변경 내용을 **저장** 합니다**.

1. **데이터 탐색기** 페이지에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | *기존 항목 사용* &vert; *cosmicworks* |
    | **컨테이너 ID** | *flatproducts* |
    | **파티션 키** | */category* |
    | **컨테이너 처리량(자동 크기 조정)** | *수동* |
    | **RU/초** | *400* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **flatproducts** 컨테이너 노드를 관찰합니다.

1. Azure Portal의 **홈** 으로 돌아갑니다.

## <a name="create-azure-data-factory-resource"></a>Azure Data Factory 리소스 만들기

이제 Azure Cosmos DB SQL API 리소스가 설치되었으므로 Azure Data Factory 리소스를 만들고 필요한 모든 구성 요소 및 연결을 구성하여 한 SQL API 컨테이너에서 다른 SQL API 컨테이너로 일회성 데이터 이동을 수행하여 데이터를 추출하고, 변환하고, 다른 SQL API 컨테이너로 로드합니다.

1. **+ 리소스 만들기** 를 선택하고 *Data Factory* 를 검색한 다음, 다음 설정을 사용하여 새 **Azure Data Factory** 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | *기존 Azure 구독* |
    | **리소스 그룹** | *기존 리소스 그룹을 선택하거나 새로 만듭니다.* |
    | **이름** | *전역적으로 고유한 이름을 입력합니다.* |
    | **지역** | *사용 가능한 지역을 선택합니다.* |
    | **버전** | *V2* |
    | **Git 구성** | *나중에 Git 구성* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Data Factory** 리소스로 이동하여 **Azure Data Factory Studio 열기** 를 선택합니다.

    > &#128161; 또는 (``adf.azure.com/home``)로 이동하여 새로 만든 Data Factory 리소스를 선택한 다음, 홈 아이콘을 선택할 수 있습니다.

1. 홈 화면에서. **수집** 옵션을 선택하여 빠른 마법사를 시작하여 대규모 작업에서 일회성 복사 데이터를 수행하고 마법사의 **속성** 단계로 이동합니다.

1. 마법사의 **속성** 단계부터 시작하여 **작업 유형** 섹션에서 **기본 제공 복사 작업** 을 선택합니다.

1. **작업 주기 또는 작업 일정** 섹션에서 **지금 한 번 실행** 을 선택한 후 **다음** 을 선택하여 마법사의 **소스** 단계로 이동합니다.

1. 마법사의 **소스** 단계에 있는 **소스 유형** 목록에서 **Azure Cosmos DB(SQL API)** 를 선택합니다.

1. **연결** 섹션에서 **+ 새 연결** 을 선택합니다.

1. **새 연결(Azure Cosmos DB(SQL API))** 팝업에서 다음 값으로 새 연결을 구성한 다음, **만들기** 를 선택합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **이름** | *CosmosSqlConn* |
    | **통합 런타임을 통해 연결** | *AutoResolveIntegrationRuntime* |
    | **인증 방법** | *기본 키* &vert; *연결 문자열* |
    | **계정 선택 방법** | *Azure 구독에서* |
    | **Azure 구독** | *기존 Azure 구독* |
    | **Azure Cosmos DB 계정 이름** | *이 랩의 앞부분에서 선택한 기존 Azure Cosmos DB 계정 이름* |
    | **데이터베이스 이름** | *cosmicworks* |

1. **원본 데이터 저장소** 섹션으로 돌아가서 **원본 테이블** 섹션 내에서 **쿼리 사용** 을 선택합니다.

1. **테이블 이름** 목록에서 **products** 을 선택합니다.

1. **쿼리** 편집기에서 기존 콘텐츠를 삭제하고 다음 쿼리를 입력합니다.

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. **미리 보기 데이터** 를 선택하여 쿼리의 유효성을 테스트합니다. **다음** 을 선택하여 마법사의 **대상** 단계로 이동합니다.

1. 마법사의 **대상** 단계에 있는 **대상 유형** 목록에서 **Azure Cosmos DB(SQL API)** 를 선택합니다.

1. **연결** 목록에서 **CosmosSqlConn** 을 선택합니다.

1. **대상** 목록에서 **flatproducts** 를 선택한 다음, **다음** 을 선택하여 마법사의 **설정** 단계로 이동합니다.

1. 마법사의 **설정** 단계에서 **작업 이름** 필드에 **FlattenAndMoveData** 를 입력합니다.

1. 나머지 모든 필드를 기본 빈 값으로 두고 **다음** 을 선택하여 마법사의 마지막 단계로 이동합니다.

1. 마법사에서 선택한 단계의 **요약** 을 검토한 다음, **다음** 을 선택합니다.

1. 배포의 다양한 단계를 관찰합니다. 배포가 완료되면 **마침** 을 선택합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장하고 **flatproducts** 컨테이너 노드를 선택한 다음, **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **이름** 이 **HL Headset** 과 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        products p
    WHERE
        p.name = 'HL Headset'
    ```

1. **쿼리 실행** 을 선택합니다.

1. 쿼리 결과를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
