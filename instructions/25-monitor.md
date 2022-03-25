---
lab:
  title: Azure Monitor를 사용하여 Azure Cosmos DB SQL API 계정 분석
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: c1b3db394c1082fd9165c0b5689e80144a87bc87
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025048"
---
# <a name="use-azure-monitor-to-analyze-an-azure-cosmos-db-sql-api-account"></a>Azure Monitor를 사용하여 Azure Cosmos DB SQL API 계정 분석

Azure Monitor는 Azure 리소스를 모니터링하는 전체 기능 집합을 제공하는 Azure의 전체 스택 모니터링 서비스입니다.  Azure Cosmos DB는 Azure Monitor를 사용하여 모니터링 데이터를 만듭니다.  Azure Monitor는 Cosmos DB의 메트릭 및 원격 분석 데이터를 캡처합니다.

이 랩에서는 Azure Cosmos DB 컨테이너에 대해 시뮬레이션된 워크로드를 실행하고 해당 워크로드가 Azure Cosmos DB 계정에 어떤 영향을 미치는지 분석합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 시작

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API 계정 만들기

Azure Cosmos DB는 여러 API를 지원하는 클라우드 기반 NoSQL 데이터베이스 서비스입니다. Azure Cosmos DB 계정을 처음 프로비전할 때 계정을 지원할 API(예: **Mongo API** 또는 **SQL API**)를 선택합니다. Azure Cosmos DB SQL API 계정이 프로비전되면 엔드포인트 및 키를 검색할 수 있습니다. 엔드포인트와 키를 사용하여 프로그래밍 방식으로 Azure Cosmos DB SQL API 계정에 연결합니다. .NET용 Azure SDK 또는 다른 SDK의 연결 문자열에서 엔드포인트 및 키를 사용합니다.

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
    | **무료 계층 할인 적용** | *`Do Not Apply`* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 최소화하되 브라우저 창을 닫지는 마세요. 다음 단계에서 백그라운드 워크로드를 시작한 후 몇 분 후에 Azure Portal로 돌아갑니다.


## <a name="import-the-microsoftazurecosmos-and-newtonsoftjson-libraries-into-a-net-script"></a>Microsoft.Azure.Cosmos 및 Newtonsoft.Json 라이브러리를 .NET 스크립트로 가져오기

.NET CLI에는 미리 구성된 패키지 피드에서 패키지를 가져오는 [패키지 추가][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 명령이 포함되어 있습니다. .NET 설치는 NuGet을 기본 패키지 피드로 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **25-monitor** 폴더로 이동합니다.

1. **25-monitor** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **25-monitor** 폴더로 이미 설정된 터미널을 엽니다.

1. 다음 명령을 사용하여 NuGet에서 [ Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 다음 명령을 사용하여 NuGet에서 [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] 패키지를 추가합니다.

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

## <a name="run-a-script-to-create-the-containers-and-the-workload"></a>스크립트를 실행하여 컨테이너 및 워크로드 만들기

이제 워크로드를 실행하여 Azure Cosmos DB 계정의 사용량을 모니터링할 준비가 되었습니다.  백그라운드에서 실행할 스크립트입니다. 이 스크립트는 세 개의 컨테이너를 만들고 일부 데이터를 해당 컨테이너에 로드합니다. 그런 다음, 스크립트는 일부 SQL 쿼리를 임의로 실행하여 Azure Cosmos DB 계정에 도달하는 여러 사용자 애플리케이션을 에뮬레이트합니다. 

1. **Visual Studio Code** 의 **탐색기** 창에서 **25-monitor** 폴더로 이동합니다.

1. **Program.cs** 코드 파일을 엽니다.

1. **endpoint** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **endpoint** 로 설정된 값으로 업데이트합니다.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 인 경우 C# 문은 **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";** 이 됩니다.

1. **key** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **key** 로 설정된 값으로 업데이트합니다.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; 예를 들어 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 C# 문은 **private static readonly string key = "fDR2ci9QgkdkvERTQ==";** 가 됩니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```
    > &#128221; 이 스크립트의 첫 번째 부분에서는 세 개의 컨테이너를 만들고 데이터를 로드합니다. 이 작업은 약 2분 정도 걸립니다. 일부 속도 제한 이벤트를 에뮬레이트하기 위해 스크립트는 프로비전된 처리량을 400RU/s로 설정합니다. 그런 다음, ***시뮬레이션된 백그라운드 워크로드 생성 중이라는 메시지를 받고 5~10분 정도 기다린 후 연습의 다음 단계로 이동*** 해야 합니다. Azure 리소스는 모니터링 데이터를 Azure 모니터에 비동기적으로 업로드하므로 Azure Monitor 메트릭 및 인사이트에서 일부 진단 데이터를 가져오려면 잠시 기다려야 합니다. 5~10분 후 다음 단계로 이동합니다. 원하는 경우 추가 진단 데이터를 수집하기 위해 5~10분 후에 스크립트를 중지할 필요가 없으며 실습이 끝날 때까지 기다리기만 하면 됩니다.

    > &#128221; 컴파일러가 스크립트가 많은 작업을 동기적으로 실행하고 작업의 응답을 기다리지 않는다는 것을 감지하기 때문에 노란색으로 몇 가지 경고가 표시됩니다. 여러 SQL 스크립트를 동시에 실행할 것으로 예상되는 동작이므로 이러한 경고를 무시할 수 있습니다.

## <a name="use-azure-monitor-to-analyze-the-azure-cosmos-db-account-usage"></a>Azure Monitor를 사용하여 Azure Cosmos DB 계정 사용량 분석

이 연습 부분에서는 브라우저로 돌아가서 Azure Monitor 인사이트 및 메트릭 보고서 중 일부를 검토합니다.

### <a name="azure-monitor-metricss-reports"></a>Azure Monitor 메트릭 원본

1. 앞에서 최소화한 열린 브라우저 창으로 돌아갑니다. 닫은 경우 새 창을 열고 portal.azure.com에서 Azure Cosmos DB 계정 페이지로 이동합니다.

1. Azure Comsos DB 왼쪽 메뉴의 *모니터링* 아래에서 **메트릭** 을 선택합니다. **범위** 및 **메트릭 네임스페이스** 필드가 올바른 정보로 미리 채워집니다. 다음 단계에서는 몇 가지 **메트릭** 옵션과 *필터 추가* 및 *분할 적용* 기능을 살펴보겠습니다.

1. 기본적으로 *메트릭* 섹션에는 지난 24시간 동안의 진단 정보가 표시됩니다. 이전 단계에서 생성한 워크로드 동안 메트릭을 보려면 더 세분화해야 합니다. 오른쪽 위 모서리에서 ***현지 시간: 지난 24시간(자동)** _이라는 레이블이 지정된 단추를 선택하면 여러 라디오 단추 시간 범위 옵션이 있는 창이 나타납니다.  _*지난 30분** 레이블이 지정된 라디오 단추를 선택하고 **적용** 단추를 선택합니다. 필요한 경우 *사용자 지정* 라디오 단추를 선택하고 시작 및 종료 날짜와 시간을 선택하여 훨씬 더 세분화할 수 있습니다. 

1. 이제 진단 차트에 대한 적절한 시간 범위가 있으므로 몇 가지 메트릭을 살펴보겠습니다. 일반 메트릭부터 시작하겠습니다. *메트릭* 풀다운에서 **총 요청 단위** 를 선택합니다. 기본적으로 이 메트릭은 RU의 총 합계로 표시됩니다. 또는 집계 풀다운을 평균 또는 최대로 변경할 수 있습니다. 이 두 집계를 확인한 후 다음 단계에서 다시 *합계* 로 설정합니다.

1. 이 메트릭은 Azure Cosmos DB 계정에서 사용된 요청 단위 수에 대한 좋은 아이디어를 제공합니다. 그러나 현재 차트는 계정에 여러 데이터베이스 또는 컨테이너가 있는 경우 문제 발생 시 도움이 되지 않을 수 있습니다. 이를 변경하고 데이터베이스에서 RU를 얼마나 소비하는지 검토해 보겠습니다. 문자 제목 아래의 메뉴에서 **분할 적용** 을 선택하고 **값** 풀다운에서 **DatabaseName** 을 선택한 다음, 차트의 아무 곳이나 선택하여 변경 내용을 적용합니다. 이제 **분할 기준 = DatabaseName** 단추는 차트 바로 위에 배치됩니다. 

1. 훨씬 더 좋은 점은 현재 대부분의 작업을 수행하는 데이터베이스를 알고 있다는 것입니다. 이 정보는 좋지만 어떤 컨테이너가 모든 작업을 수행하는지 알 수 없습니다.  **분할 기준 = DatabaseName** 단추를 선택하여 분할 조건을 변경하고 *값* 풀다운에서 **CollectionName** 을 선택합니다. 이제 **customer** 및 **salesOrder** 컬렉션에 대한 데이터가 있어야 합니다. 이 차트에 문제가 하나 있는데, **salesOrder** 컬렉션은 두 데이터베이스 **database-v2** 및 **database-v3** 에 있으므로 이 값은 두 데이터베이스의 해당 컬렉션 이름을 집계한 것입니다.

1. 이는 쉽게 수정할 수 있고, **필터 추가** 단추를 선택하고, *속성* 풀다운에서 **DatabaseName** 을 선택하고, *값* 아래에서 **database-V3** 을 선택합니다.

1. 두 가지 메트릭을 더 살펴보겠습니다. 기존 차트를 편집하고 원하는 경우 새 차트를 만들 수도 있습니다. 차트 위에서 *Azure Cosmos DB 계정 이름* 과 **총 요청 단위** 레이블이 있는 단추를 선택합니다. **메트릭** 풀다운에서 *총 요청 수* 를 선택합니다. 사용 가능한 집계는 *개수* 뿐입니다.

1. 여기서 두 가지 주요 필터를 사용하면 다양한 유형의 문제를 해결하는 데 도움이 될 수 있습니다. *값* 이 **200** 및 **429** 인 경우에서 속성 **StatusCode** 를 사용하여 필터를 추가해 보겠습니다(다른 형식의 세부 정보를 가진 유사한 필터는 **Status** 입니다). StatusCode를 사용하도록 분할을 변경합니다. 성공적인 요청 상태인 200에 비해, 429 또는 비율 제한 요청이 너무 많습니다. 프로비전된 처리량을 400RU/s로 설정하는 동안 스크립트가 초당 수천 개의 요청을 전송하기 때문에 429 예외가 발생했습니다. *성공적인 요청에 비해 이렇게 많은 429 예외는 프로덕션 환경에서 정상적일 수 없습니다. 프로덕션 환경에서 429 예외는 정상적인 Azure Cosmos DB 계정에서 자주 발생하면 안 됩니다*.  **총 요청 단위** 에 대해 유사한 문제 해결 방식으로 **StautusCode** 또는 **Status** *Properties* 를 사용할 수도 있습니다.

1. 계속해서 **총 요청** 을 살펴보고 분할을 **OperationType** 으로 변경해 보겠습니다.  이 속성은 작업의 대부분을 수행하는 읽기 또는 쓰기 작업을 확인하는 데 도움이 됩니다. 다시 말하지만, 이 속성은 **총 요청 단위** 에 대해서도 유사하게 사용될 수 있습니다.

1. **총 요청 단위** 에서 수행했듯이 다양한 필터와 분할 옵션을 선택하여 실험을 진행합니다. 

1. 이 연습에서 살펴볼 마지막 메트릭은 **정규화된 RU 사용량** 메트릭입니다. 분할을 **PartitionKeyRangeId** 로 변경합니다. 이 메트릭을 사용하면 어느 파티션 키 범위 사용량이 더 많은지 식별할 수 있습니다. 메트릭을 통해 파티션 키 범위에 대한 처리량의 편향을 알 수 있습니다. 계속해서 *메트릭* 풀다운에서 해당 메트릭을 선택합니다. 이제 이 차트는 일정하게 100% **정규화된 RU 소비량** 에 도달하는 매우 비정상적인 시스템을 표시합니다.

> &#128221; 한 번에 두 개 이상의 차트를 보려면 차트 이름 위에 있는 **+ 새 차트** 옵션을 클릭합니다. 

> &#128221; 메트릭을 직접 저장할 수는 없지만 기존 대시보드를 만들거나 사용하고 차트의 오른쪽 위 모서리에 있는 **대시보드에 고정** 단추를 클릭하여 이 차트를 추가할 수 있습니다.  단추를 클릭하고 **새로 만들기** 탭을 선택하고 이름을 *DP-420 labs* 로 지정한 다음, **만들기 및 고정** 을 클릭합니다. 프라이빗 대시보드를 보려면 왼쪽 위 모서리에 있는 포털 메뉴로 이동하여 Azure 리소스 옵션에서 대시보드를 선택해야 합니다. 대시보드가 처음 표시되는 데 몇 분 정도 걸릴 수 있습니다.

> &#128221; 차트를 공유하는 한 가지 방법은 공유 풀다운을 클릭하고 Excel 파일 또는 링크 복사 옵션으로 다운로드하는 것입니다.

### <a name="azure-monitor-insights-reports"></a>Azure Monitor 인사이트 보고서

Azure Monitor 메트릭 진단 보고서를 미세 조정하는 데 시간이 필요할 수 있습니다.  Cosmos DB Insights는 Azure Cosmos DB 리소스의 전반적인 성능, 오류 및 운영 상태에 대한 보기를 제공합니다. 이러한 인사이트 차트는 메트릭 차트와 유사한 미리 작성된 차트입니다. 이 중 일부 값을 살펴보겠습니다.

1. Azure Comsos DB 왼쪽 메뉴의 *모니터링* 아래에서 **인사이트** 를 선택합니다. 개요에서 관리 옵션까지 여러 탭이 있음을 알 수 있습니다. 이러한 **인사이트** 차트 중 몇 가지를 살펴보겠습니다. 첫 번째 탭인 개요 탭에서는 사용할 수 있는 가장 일반적인 차트에 대한 요약을 제공합니다. 예를 들어, 총 요청, 데이터 및 인덱스 사용량, 429 예외 및 정규화된 RU 사용량과 같은 차트입니다.  이전 섹션에서 이러한 차트의 대부분을 살펴보았습니다.

1. 차트 맨 위에서 **시간 범위** 로 곧장 이동할 수 있으므로 *15* 또는 *30* 분을 선택하여 이 연습의 워크로드를 평가합니다.

1. *각* 차트의 오른쪽 위 모서리에 ***메트릭 탐색기 열기** _ 옵션이 있습니다. 계속해서 *총 요청* 차트에 대한 _ **메트릭 탐색기 열기*** 옵션을 선택하겠습니다. 이 옵션을 선택하면 앞에서 검토한 메트릭의 보고서로 이동합니다. 메트릭 탐색기를 열면 차트의 상당 부분이 이미 구축되어 있다는 장점이 있습니다.

1. 메트릭 차트의 오른쪽 위에 있는 **X** 를 선택하여 Insights 페이지로 돌아갈 수 있습니다.

1. 처리량 탭을 선택합니다. 이러한 차트는 처리량 문제를 정확히 파악하는 데 적합합니다.  핫 파티션을 검색하는 데 사용할 수 있는 **PartitionKeyRangeID별 정규화된 RU 사용량(%)** 차트에 주의하세요.

1. 요청 탭을 선택합니다. 이 차트는 계정이 경험한 제한 이벤트 수(429 대 200)를 분석하거나 작업 유형당 요청 수를 검토하는 데 모두 유용합니다.  

1. 스토리지 탭을 선택합니다. 이러한 차트는 컬렉션의 증가와 데이터 및 인덱스 사용량을 모두 보여줍니다.  

1. 시스템 탭을 선택합니다. 애플리케이션이 계정 메타데이터를 자주 생성, 삭제 또는 쿼리하는 경우 429 예외가 발생할 수 있습니다.  이 차트는 빈번한 메타데이터 액세스가 429 예외의 원인인지 판단하는 데 도움이 됩니다. 또한 메타데이터 요청의 상태를 확인할 수 있습니다.  

### <a name="azure-monitor-insights-reports"></a>Azure Monitor 인사이트 보고서

1. 프로그램이 여전히 실행 중이면 Visual Studio Code 명령 터미널로 돌아갑니다.

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[nuget.org/packages/Newtonsoft.Json/13.0.1]: https://www.nuget.org/packages/Newtonsoft.Json/13.0.1
