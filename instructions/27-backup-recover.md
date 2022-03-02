---
Lab:
  title: 복구 지점에서 데이터베이스 또는 컨테이너 복구
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 1e934b994735c51ad3d699474186e2255ba1f057
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025166"
---
# <a name="recover-a-database-or-container-from-a-recovery-point"></a>복구 지점에서 데이터베이스 또는 컨테이너 복구 

Azure는 데이터의 암호화된 백업을 자동으로 수행합니다. 이러한 백업은 **주기적** 백업 모드와 **지속적인** 백업 모드의 두 가지 모드로 수행됩니다.

이 랩에서는 지속적인 백업 모드를 사용하여 `backup` 및 `restores`를 수행합니다. 먼저 Azure Cosmos DB 계정을 만듭니다. 그런 다음 두 개의 컨테이너를 만들고 몇 가지 문서를 추가합니다. 다음으로 해당 컨테이너의 문서 몇 가지를 업데이트합니다. 마지막으로 각 삭제 이전의 지점까지 계정의 복원을 만듭니다.

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기

Azure Cosmos DB는 여러 API를 지원하는 클라우드 기반 NoSQL 데이터베이스 서비스입니다. Azure Cosmos DB 계정을 처음으로 프로비저닝할 때 계정을 지원할 API(예: **Mongo API** 또는 **SQL API**)를 선택합니다. Azure Cosmos DB SQL API 계정이 프로비저닝되면 엔드포인트 및 키를 검색할 수 있습니다. 엔드포인트와 키를 사용하여 프로그래밍 방식으로 Azure Cosmos DB SQL API 계정에 연결합니다. .NET용 Azure SDK 또는 다른 SDK의 연결 문자열에서 엔드포인트 및 키를 사용합니다.

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

    > &#128221; Azure Cosmos DB 계정을 만드는 동안 **백업 정책** 탭에서 선택하여 **지속적인** 모드를 사용하도록 설정할 수 있습니다. 이 랩에서는 계정을 만드는 동안 또는 아래 선택적 섹션에서 계정을 만든 후에 이 기능을 사용하도록 설정할 수 있습니다. **계정을 만든 후 기능을 사용하도록 설정하는 데 5분 이상 걸릴 수 있습니다.**  

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

## <a name="add-a-database-and-two-containers-to-the-account"></a>계정에 데이터베이스 및 컨테이너 2개 추가

데이터베이스와 몇 개의 컨테이너를 만들어 보겠습니다.

1. Azure Portal에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. **데이터 탐색기** 에서 다음 설정을 사용하여 새 데이터베이스를 추가합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터베이스 ID** | *`Sales`* |
    | **컨테이너 간에 처리량 공유** | 선택 안 함 |

1. **데이터 탐색기** 에서 다음 설정을 사용하여 새 컨테이너를 추가합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터베이스 ID** | 기존 이름 사용: *Sales*  |
    | **컨테이너 ID** | *`customer`* |
    | **파티션 키** | *`/id`* |
    | **컨테이너 처리량(400 - 무제한 RU/s)** | 수동 처리량: *400* |

1. **데이터 탐색기** 에서 다음 설정을 사용하여 새 컨테이너를 추가합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터베이스 ID** | 기존 이름 사용: *Sales*  |
    | **컨테이너 ID** | *`salesOrder`* |
    | **파티션 키** | *`/id`* |
    | **컨테이너 처리량(400 - 무제한 RU/s)** | 수동 처리량: *400* |

## <a name="add-items-to-the-containers"></a>컨테이너에 항목 추가

해당 컨테이너에 일부 문서를 추가해 보겠습니다.

1. Azure Portal에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. **데이터 탐색기** 에서 **고객** 컨테이너에 다음 두 문서를 추가합니다.

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
1. **데이터 탐색기** 에서 **salesOrder** 컨테이너에 다음 세 개의 문서를 추가합니다.

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

## <a name="change-the-default-backup-mode-to-continuous-optional-if-feature-not-enabled-during-the-account-creation"></a>기본 백업 모드를 지속적으로 변경합니다(계정을 만드는 동안 기능을 사용하지 않는 경우 선택 사항)

Azure Cosmos DB 계정을 만드는 동안 이 기능을 활성화하지 않았다면 지금 활성화해야 합니다.  백업 모드를 변경하는 것은 간단합니다. 하나의 설정을 **켜기** 로 변경하기만 하면 됩니다. 이제 변경해 보겠습니다.

1. Azure Portal에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. **설정** 섹션에서 **기능** 을 선택합니다.

1. 기능을 활성화하려면 **지속적인 백업** 옵션을 선택합니다. 이 옵션을 선택하여 창이 나타나면 **사용** 단추를 선택합니다.  이 기능을 사용하려면 5분 이상이 걸릴 수 있습니다.

## <a name="delete-one-of-the-salesorder-documents"></a>salesOrder 문서 중 하나 삭제

1. **데이터 탐색기** 에서 다음 쿼리를 실행하여 현재 날짜 및 시간을 가져옵니다. 해당 타임스탬프를 메모장에 복사합니다. 이 타임스탬프는 UTC여야 합니다.

```
SELECT GetCurrentDateTime ()
```

1. **데이터 탐색기** 에서 **ID** `0019092E-BD25-48F5-8050-7051B2655BC5`가 있는 **salesOrder** 문서를 찾습니다. 문서를 삭제하고 문서가 더 이상 존재하지 않는지 확인합니다.

## <a name="restore-the-database-to-the-point-before-you-deleted-the-salesorder-document"></a>salesOrder 문서를 삭제하기 전의 지점으로 데이터베이스 복원

1. Azure Portal에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. 설정 섹션에서 **특정 시점 복원** 을 선택합니다. 다음 설정을 사용합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **복원 시점(UTC)** | 날짜 및 시간을 적절하게 변환합니다. 시간은 AM/PM 형식이어야 합니다.|
    | **위치** | 사용 가능한 위치 선택 |
    | **복원하려는 리소스 선택** | 선택한 데이터베이스/컨테이너 |
    | **복원 리소스** | *salesOrder* |
    | **복원 대상 계정** |  ***새** _ _Azure Cosmos DB 계정 이름* 선택 |

    > &#128221; Azure Cosmos DB 복원의 경우 *기존 계정 _위에 복원하지_ **않고*** 항상 새로운 Azure Cosmos DB 계정을 만들어야 합니다.

    > &#128221; 전체 데이터베이스 또는 전체 계정을 복원하도록 선택할 수도 있지만 실제 프로덕션 환경에서는 데이터베이스가 매우 커질 수 있습니다. 많은 시나리오에서 필요한 컨테이너 또는 데이터베이스만 복원하는 것이 더 빠를 수 있습니다.

1. 이 복원은 15분 이상 걸릴 수 있습니다. 다음 섹션으로 이동하여 이 복원을 백그라운드에서 실행한 상태로 둡니다.

## <a name="delete-the-customer-container"></a>고객 컨테이너 삭제

1. **데이터 탐색기** 에서 다음 쿼리를 실행하여 현재 날짜 및 시간을 가져옵니다. 해당 타임스탬프를 메모장에 복사합니다.

```
SELECT GetCurrentDateTime ()
```

1. **고객** 컨테이너를 삭제합니다.

## <a name="restore-the-database-to-the-point-before-you-deleted-the-salesorder-document"></a>salesOrder 문서를 삭제하기 전의 지점으로 데이터베이스 복원

1. Azure Portal에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. 설정 섹션에서 **특정 시점 복원** 을 선택합니다. 다음 설정을 사용합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **위치** | 사용 가능한 위치 선택 |
    | **복원 시점(UTC)** | 날짜 및 시간을 적절하게 변환합니다. 시간은 AM/PM 형식이어야 합니다.|
    | **복원하려는 리소스 선택** | 선택한 데이터베이스/컨테이너 |
    | **복원 리소스** | *`customer`* |
    | **복원 대상 계정** |  ***새** _ _Azure Cosmos DB 계정 이름* 선택 |

    > &#128221; Azure Cosmos DB 복원의 경우 *기존 계정 _위에 복원하지_ **않고*** 항상 새로운 Azure Cosmos DB 계정을 만들어야 합니다.

    > &#128221; 전체 데이터베이스 또는 전체 계정을 복원하도록 선택할 수도 있지만 실제 프로덕션 환경에서는 데이터베이스가 매우 커질 수 있습니다. 많은 시나리오에서 필요한 컨테이너 또는 데이터베이스만 복원하는 것이 더 빠를 수 있습니다.

1. 이 복원은 15분 이상 걸릴 수 있습니다. 다음 섹션으로 이동하여 이 복원을 백그라운드에서 실행한 상태로 둡니다.

## <a name="review-the-data-restored"></a>복원된 데이터 검토

복원은 데이터베이스 크기 및 기타 요인에 따라 시간이 오래 걸릴 수도 있습니다. Azure Cosmos DB 계정 복원이 완료되면 다음을 수행합니다.

1. 첫 번째 복원의 경우 세 번째 문서가 복구되었는지 확인합니다.

1. 두 번째 복원의 경우 고객 테이블을 복원해야 합니다.

## <a name="cleanup"></a>정리

1. 계정 복원에서 만든 두 개의 새 Azure Cosmos DB 계정을 삭제합니다.

1. 판매 데이터베이스를 삭제하고 필요한 경우 원래 Azure Cosmos DB 계정을 삭제합니다.


