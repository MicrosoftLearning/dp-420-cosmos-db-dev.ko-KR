---
lab:
  title: Azure Functions를 사용하여 Azure Cosmos DB SQL API 데이터 처리
  module: Module 7 - Integrate Azure Cosmos DB SQL API with Azure services
ms.openlocfilehash: 78a13ba0cb150925083d449bb9abae1169af8e71
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025035"
---
# <a name="process-azure-cosmos-db-sql-api-data-using-azure-functions"></a>Azure Functions를 사용하여 Azure Cosmos DB SQL API 데이터 처리

Azure Functions용 Azure Cosmos DB 트리거는 변경 피드 프로세서를 사용하여 구현됩니다. 이 정보를 사용하여 Azure Cosmos DB SQL API 컨테이너에서 작업을 만들고 업데이트하는데 응답하는 함수를 만들 수 있습니다. 변경 피드 프로세서를 수동으로 구현한 경우 Azure Functions 설정도 비슷합니다.

이 랩에서는 다음을 수행합니다.

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

1. 리소스 메뉴에서 **데이터 탐색기** 를 선택합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 확장한 다음, **새 데이터베이스** 를 선택합니다.

1. **새 데이터베이스** 팝업에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | *cosmicworks* |

1. **데이터 탐색기** 창으로 돌아가서 계층 내의 **cosmicworks** 데이터베이스 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 기존 항목 사용 &vert; *cosmicworks*  |
    | **컨테이너 ID** | *products* |
    | **파티션 키** | */categoryId* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **products** 컨테이너 노드를 관찰합니다.

1. **데이터 탐색기** 창에서 **새 컨테이너** 를 다시 선택합니다.

1. **새 컨테이너** 창에서 각 설정에 대해 다음 값을 입력한 다음, **확인** 을 선택합니다.

    | **설정** | **값** |
    | --: | :-- |
    | **데이터베이스 ID** | 기존 항목 사용 &vert; *cosmicworks*  |
    | **컨테이너 ID** | *productslease* |
    | **파티션 키** | */id* |

1. **데이터 탐색기** 창으로 돌아가서 **cosmicworks** 데이터베이스 노드를 확장한 다음, 계층 내의 **productslease** 컨테이너 노드를 관찰합니다.

1. Azure 포털의 **홈** 으로 돌아갑니다.

## <a name="create-an-azure-function-app-and-azure-cosmos-db-triggered-function"></a>Azure Function 앱 및 Azure Cosmos DB 트리거 함수 만들기

코드 작성을 시작하기 전에 만들기 마법사를 사용하여 Azure Functions 리소스 및 해당 종속 리소스(Application Insights, Storage)를 만들어야 합니다.

1. **+ 리소스 만들기** 를 선택하고 *Functions* 를 검색한 다음, 다음 설정을 사용하여 새 **함수 앱** 계정 리소스를 만들고, 나머지 모든 설정을 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | 기존 Azure 구독 |
    | **리소스 그룹** | 기존 리소스 그룹을 선택하거나 새로 만듭니다. |
    | **이름** | 전역적으로 고유한 이름을 입력합니다. |
    | **게시** | *코드* |
    | **런타임 스택** | *.NET* |
    | **버전** | *6* |
    | **지역** | 사용 가능한 지역을 선택합니다. |
    | **스토리지 계정** | *새 스토리지 계정 만들기* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Functions** 계정 리소스로 이동하여 **함수** 창으로 이동합니다.

1. **함수** 창에서 **+ 만들기** 를 선택합니다.

1. **함수 만들기** 팝업에서 다음 설정을 사용하여 새 함수를 만들고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **개발 환경** | *포털에서 개발* |
    | **템플릿 선택** | Azure Cosmos DB 트리거 |
    | **새 함수** | *ItemsListener* |
    | **Cosmos DB 계정 연결** | 새로 만들기 선택 &vert; Azure Cosmos DB 계정 &vert; 앞에서 만든 Azure Cosmos DB 계정 선택   |
    | **데이터베이스 이름** | *cosmicworks* |
    | **컬렉션 이름** | *products* |
    | **임대 컬렉션 이름** | *productslease* |
    | **임대 컬렉션이 없는 경우 만들기** | *아니요* |

## <a name="implement-function-code-in-net"></a>.NET에서 함수 코드 구현

이전에 만든 함수는 포털에서 편집되는 C# 스크립트입니다. 이제 포털에서 짧은 함수를 작성하여 컨테이너에서 삽입되거나 업데이트된 항목의 고유 식별자를 출력할 수 있습니다.

1. **ItemsListener** &vert; **함수** 창에서 **코드 + 테스트** 창으로 이동합니다.

1. **run.csx** 스크립트의 편집기에서 편집기 영역의 콘텐츠를 삭제합니다.

1. 편집기 영역에서 **Microsoft.Azure.DocumentDB.Core** 라이브러리를 참조합니다.

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. **System**, **System.Collections.Generic** 및 [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] 네임스페이스에 Using 블록을 추가합니다.

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. 두 개의 매개 변수가 있는 **Run** 이라는 새 정적 메서드를 만듭니다.

    1. 제네릭 형식의 [문서][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]이고 **IReadOnlyList\<\>** 형식의 **input** 이라는 매개 변수입니다.

    1. **ILogger** 형식의 **log** 라는 매개 변수입니다.

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. **Run** 메서드 내에서 현재 일괄 처리의 항목 수를 계산하는 문자열을 전달하는 **log** 변수의 **LogInformation** 메서드를 호출합니다.

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. **Run** 메서드 내에서 **item** 변수를 사용하여 **문서** 형식의 인스턴스를 나타내는 **input** 변수를 반복하는 foreach 루프를 만듭니다.

    ```
    foreach(Document item in input)
    {
    }
    ```

1. **Run** 메서드의 foreach 루프 내에서 **item** 변수의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] 속성을 인쇄하는 문자열을 전달하는 **log** 변수의 **LogInformation** 메서드를 호출합니다.

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    
        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

1. **로그** 섹션을 확장하여 현재 함수의 스트리밍 로그에 연결합니다.

    > &#128161; 스트리밍 로그 서비스에 연결하는 데 몇 초 정도 걸릴 수 있습니다. 연결되면 로그 출력에 메시지가 표시됩니다.

1. 현재 함수 코드를 **저장** 합니다.

1. C# 코드 컴파일 결과를 확인합니다. 로그 출력의 끝에 **컴파일 성공** 메시지가 표시되어야 합니다.

    > &#128221; 로그 출력에 경고 메시지가 표시될 수 있습니다. 이러한 경고는 이 랩에 영향을 주지 않습니다.

1. 로그 섹션을 **최대화** 하고 출력 창을 확장하여 사용 가능한 최대 공간을 채웁니다.

    > &#128221; 다른 도구를 사용하여 Azure Cosmos DB SQL API 컨테이너에서 항목을 생성합니다. 항목을 생성하면 이 브라우저 창으로 돌아가 출력을 확인합니다. 브라우저 창을 조기에 닫지 마세요.

## <a name="seed-your-azure-cosmos-db-sql-api-account-with-sample-data"></a>샘플 데이터를 사용하여 Azure Cosmos DB SQL API 계정 시드

**Cosmicworks** 데이터베이스 및 **products** 컨테이너를 만드는 명령줄 유틸리티를 사용합니다. 그런 다음 도구는 터미널 창에서 실행되는 변경 피드 프로세서를 사용하여 관찰할 항목 집합을 만듭니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

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

1. **Visual Studio Code** 를 닫습니다.

1. Azure Functions 로그 섹션이 확장된 상태로 현재 열려 있는 브라우저 창 또는 탭으로 돌아갑니다.

1. 함수의 로그 출력을 확인합니다. 터미널에는 변경 피드를 사용하여 전송된 각 변경 내용에 대해 **검색된 작업** 이라는 메시지가 출력됩니다. 작업은 최대 100개의 작업 그룹으로 일괄 처리됩니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
