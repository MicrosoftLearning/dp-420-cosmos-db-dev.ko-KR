---
lab:
  title: Azure Portal을 사용하여 저장 프로시저 만들기
  module: Module 13 - Create server-side programming constructs in Azure Cosmos DB SQL API
ms.openlocfilehash: 0af86a04b0a5cc9a9786e88063e7f5590baa7089
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025055"
---
# <a name="create-a-stored-procedure-with-the-azure-portal"></a>Azure Portal을 사용하여 저장 프로시저 만들기

저장 프로시저는 Azure Cosmos DB에서 비즈니스 논리 서버 쪽을 실행할 수 있는 방법 중 하나입니다. 저장 프로시저를 사용하면 단일 트랜잭션 범위 내의 여러 문서에 컨테이너를 사용하여 기본 CRUD(만들기, 읽기, 업데이트, 삭제) 작업을 수행할 수 있습니다.

이 랩에서는 컨테이너 내에서 문서를 만드는 저장 프로시저를 작성합니다. 그런 다음, SQL 쿼리를 사용하여 저장 프로시저의 결과의 유효성을 검사합니다.

## <a name="author-a-stored-procedure"></a>저장 프로시저 작성

저장 프로시저는 언어 통합 JavaScript로 작성되며 데이터베이스 엔진 내에서 기본 CRUD 작업의 실행을 지원합니다. 데이터베이스 엔진 내에서 실행되는 JavaScript는 Azure Cosmos DB용 서버 쪽 JavaScript SDK 및 일련의 도우미 메서드를 사용하여 가능합니다.

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

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **새 컨테이너** 를 선택한 후, 다음 설정으로 새 컨테이너를 만들고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **데이터베이스 ID** | *새로 만들기* &vert; *cosmicworks* |
    | **컨테이너 간에 처리량 공유** | *이 옵션을 선택합니다.* |
    | **데이터베이스 처리량** | *수동* &vert; *400* |
    | **컨테이너 ID** | *products* |
    | **인덱싱** | *자동* |
    | **파티션 키** | */categoryId* |

1. 계속해서 **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 선택합니다.

1. **새 저장 프로시저** 를 선택합니다.

1. **저장 프로시저 ID** 필드에 **createDoc** 값을 입력합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. 입력 매개 변수 없이 **createDoc** 라는 새 JavaScript 함수를 만듭니다.

    ```
    function createDoc() {
        
    }
    ```

1. **createDoc** 함수 내에서 기본 제공 [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] 메서드를 호출하고 결과를 **context** 라는 변수에 저장합니다.

    ```
    var context = getContext();
    ```

1. 컨텍스트 개체의 [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] 메서드를 호출하고 결과를 **container** 라는 변수에 저장합니다.

    ```
    var container = context.getCollection();
    ```

1. 다음 두 가지 속성을 사용하여 **doc** 라는 새 개체를 만듭니다.

    | **속성** | **값** |
    | ---: | :--- |
    | **이름** | *first document* |
    | **범주 ID** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. 컨테이너 개체의 **getSelfLink** 메서드 및 새 문서를 매개 변수로서 호출한 결과를 전달하는 컨테이너 개체의 **createDocument** 메서드를 호출합니다.

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. 완료되면 저장 프로시저 코드에 다음이 포함됩니다.

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. **저장** 을 선택하여 저장 프로시저의 변경 내용을 유지합니다.

1. **실행** 을 선택한 후 다음 입력 매개 변수를 사용하여 저장 프로시저를 실행합니다.

    | **설정** | **Key** | **값** |
    | ---: | :--- | :--- |
    | **파티션 키 값** | *String* | *demo* |

1. 빈 결과를 관찰합니다. 저장 프로시저가 성공적으로 실행되는 동안 JavaScript 코드는 사람이 읽을 수 있는 응답을 반환하지 않았습니다.

## <a name="implement-best-practices-for-a-stored-procedure"></a>저장 프로시저에 대한 모범 사례 구현

이 랩의 앞부분에서 작성한 저장 프로시저에는 기본 기능이 있지만 모든 저장 프로시저에서 구현해야 하는 몇 가지 일반적인 오류 처리 기술도 누락되었습니다. 첫째, 저장 프로시저는 항상 작업을 완료할 시간이 있다고 가정하고 충분한 시간이 있는지 확인하기 위해 **createDocument** 메서드의 반환 값을 확인하지 않습니다. 둘째, 저장 프로시저는 잠재적인 오류 메시지를 확인하거나 throw하지 않고 모든 문서가 성공적으로 삽입되었다고 가정합니다. 마지막으로 저장 프로시저는 새로 만든 문서를 원래 저장 프로시저를 호출한 요청에 대한 HTTP 응답으로 반환하지 않습니다. 이러한 세 가지 저장 프로시저를 변경하여 일반적인 모범 사례를 구현합니다.

1. **createDoc** 저장 프로시저의 편집기로 돌아갑니다.

1. **createDoc** 함수를 정의하는 코드에서 줄 1을 찾아

    ```
    function createDoc() {
    ```

    **title** 이라는 매개 변수를 포함하도록 코드 줄을 업데이트합니다.

    ```
    function createDoc(title) {
    ```

1. **doc** 개체의 **name** 속성을 설정하는 코드에서 줄 5를 찾아

    ```
    name: 'first document',
    ```

    **title** 매개 변수의 값을 사용하도록 코드 줄을 업데이트합니다.

    ```
    name: title,
    ```

1. **createDocument** 메서드를 호출하는 코드에서 줄 8을 찾아

    ```
    container.createDocument(
    ```

    **accepted** 라는 변수에 메서드 호출 결과를 저장하도록 코드 줄을 업데이트합니다.

    ```
    var accepted = container.createDocument(
    ```

1. **createDocument** 메서드 호출 후 새 코드 줄을 추가하여 **accepted** 변수의 값을 확인하고 true가 아닌 경우 메서드를 반환합니다.

    ```
    if (!accepted) return;
    ```

1. 마지막으로, **error** 및 **newDoc** 이라는 두 매개 변수를 사용하는 함수인 **createDocument** 메서드 호출에 세 번째 매개 변수를 추가하고, 오류가 null인지 확인한 다음, newDoc를 저장 프로시저의 응답 본문으로 설정합니다.

    ```
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. 완료되면 저장 프로시저 코드에 다음이 포함됩니다.

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. **저장** 을 선택하여 저장 프로시저의 변경 내용을 유지합니다.

1. **실행** 을 선택한 후 다음 입력 매개 변수를 사용하여 저장 프로시저를 실행합니다.

    | **설정** | **Key** | **값** |
    | ---: | :--- | :--- |
    | **파티션 키 값** | *String* | *demo* |
    | **입력 매개 변수** | *String* | *두 번째 문서* |

1. JSON 결과를 관찰합니다. 저장 프로시저가 성공적으로 실행된 후 새로 만든 문서가 원래 HTTP 요청에 대한 응답으로 반환되었습니다.

## <a name="query-documents"></a>문서 쿼리

작업을 마무리하려면 데이터 탐색기를 사용하여 이 랩에서 만든 두 문서를 반환하는 SQL 쿼리를 실행합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택합니다.

1. **새 SQL 쿼리** 를 선택합니다.

1. 편집기 영역의 콘텐츠를 삭제합니다.

1. **categoryId** 가 **demo** 와 동일한 모든 문서를 반환하는 새 SQL 쿼리를 만듭니다.

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. **쿼리 실행** 을 선택합니다.

1. 이 쿼리를 실행한 결과로 이 랩에서 만든 두 문서를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[azure.github.io/azure-cosmosdb-js-server/context.html]: https://azure.github.io/azure-cosmosdb-js-server/Context.html
[azure.github.io/azure-cosmosdb-js-server/global.html]: https://azure.github.io/azure-cosmosdb-js-server/global.html
