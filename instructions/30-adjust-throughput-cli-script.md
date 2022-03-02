---
lab:
  title: 프로비저닝된 처리량을 Azure CLI 스크립트를 사용하여 조정
  module: Module 12 - Manage an Azure Cosmos DB SQL API solution using DevOps practices
ms.openlocfilehash: f6eaf4f0e07037010cec92f3db348a4806304d94
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025107"
---
# <a name="adjust-provisioned-throughput-using-an-azure-cli-script"></a>프로비저닝된 처리량을 Azure CLI 스크립트를 사용하여 조정

Azure CLI는 Azure 전체의 다양한 리소스를 관리하는 데 사용할 수 있는 명령 집합입니다. Azure Cosmos DB에는 선택한 API에 상관없이 Azure Cosmos DB 계정의 다양한 패싯을 관리하는 데 사용할 수 있는 풍부한 명령 그룹이 있습니다.

이 랩에서는 Azure CLI를 사용하여 Azure Cosmos DB 계정, 데이터베이스, 컨테이너를 만듭니다. 그런 다음 Azure CLI를 사용하여 프로비저닝된 처리량을 조정합니다.

## <a name="log-in-to-the-azure-cli"></a>Azure CLI에 로그인합니다.

Azure CLI를 사용하기 전에 먼저 CLI 버전을 확인하고 Azure 자격 증명을 사용하여 로그인해야 합니다.

1. **Visual Studio Code** 를 시작합니다.

1. **터미널** 메뉴를 연 다음, **새 터미널** 을 선택하여 새 터미널 인스턴스를 엽니다.

1. 다음 명령을 사용하여 Azure CLI 버전을 확인합니다.

    ```
    az --version
    ```

1. 다음 명령을 사용하여 가장 일반적인 Azure CLI 명령 그룹을 확인합니다.

    ```
    az --help
    ```

1. 다음 명령을 사용하여 Azure CLI의 대화형 로그인 절차를 시작합니다.

    ```
    az login
    ```

1. Azure CLI는 웹 브라우저 창 또는 탭을 자동으로 엽니다. 브라우저 인스턴스 내에서 구독과 연결된 Microsoft 자격 증명을 사용하여 Azure CLI에 로그인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="create-azure-cosmos-db-account-using-the-azure-cli"></a>Azure CLI를 사용하여 Azure Cosmos DB 계정 만들기

**cosmosdb** 명령 그룹에는 CLI를 사용하여 Azure Cosmos DB 계정을 만들고 관리하는 기본 명령이 포함되어 있습니다. Azure Cosmos DB 계정에는 주소 지정 가능한 URI가 있으므로 스크립트를 통해 계정을 만들더라도 새 계정에 대해 전역적으로 고유한 이름을 만드는 것이 중요합니다.

1. **Visual Studio Code** 내에 이미 열려 있는 터미널 인스턴스로 돌아갑니다.

1. 다음 명령을 사용하여 **Azure Cosmos DB** 와 관련된 가장 많은 Azure CLI 명령을 확인합니다.

    ```
    az cosmosdb --help
    ```

1. 다음 명령을 사용하여 [Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] PowerShell cmdlet으로 **suffix** 라는 새 변수를 만듭니다.

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    > &#128221; Get-Random cmdlet은 0에서 1,000,000 사이의 임의 정수를 생성합니다. 이는 서비스에는 전역적으로 고유한 이름이 필요하기 때문에 유용합니다.

1. 하드 코딩된 문자열 **csms** 및 변수 대체를 사용하여 다른 새 변수 이름 **accountName** 을 만들어 다음 명령을 사용하여 **$suffix** 변수 값을 삽입할 수 있습니다.

    ```
    $accountName="csms$suffix"
    ```

1. 다음 명령을 사용하여 이 랩의 앞부분에서 만들거나 본 리소스 그룹의 이름으로 또 다른 새 변수 이름인 **resourceGroup** 을 만듭니다.

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 예를 들어 리소스 그룹의 이름이 **dp420** 인 경우 명령은 **$resourceGroup="dp420"** 입니다.

1. **echo** cmdlet을 사용하여 다음 명령을 통해 터미널 출력에 **$accountName** 및 **$resourceGroup** 변수의 값을 씁니다.

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. 다음 명령을 사용하여 **az cosmosdb create** 에 대한 옵션을 봅니다.

    ```
    az cosmosdb create --help
    ```

1. 미리 정의된 변수 및 다음 명령을 사용하여 새 Azure Cosmos DB 계정을 만듭니다.

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. **create** 명령이 실행을 완료하고 반환할 때까지 기다린 후 이 랩을 계속 진행합니다.

    > &#128161; **create** 명령은 평균적으로 완료하는 데 2~12분 정도 걸릴 수 있습니다.

## <a name="create-azure-cosmos-db-sql-api-resources-using-the-azure-cli"></a>Azure CLI를 사용하여 Azure Cosmos DB SQL API 리소스 만들기

**cosmosdb sql** 명령 그룹에는 Azure Cosmos DB의 SQL API 관련 리소스를 관리하는 명령이 포함되어 있습니다. **--help** 플래그를 항상 사용하여 이러한 명령 그룹에 대한 옵션을 검토할 수 있습니다.

1. **Visual Studio Code** 내에 이미 열려 있는 터미널 인스턴스로 돌아갑니다.

1. 다음 명령을 사용하여 **Azure Cosmos DB SQL API** 와 관련된 가장 많은 Azure CLI 명령 그룹을 확인합니다.

    ```
    az cosmosdb sql --help
    ```

1. 다음 명령을 사용하여 **Azure Cosmos DB SQL API** 데이터베이스를 관리하기 위한 Azure CLI 명령을 확인합니다.

    ```
    az cosmosdb sql database --help
    ```

1. 미리 정의된 변수, 데이터베이스 이름 **cosmicworks**, 다음 명령을 사용하여 새 Azure Cosmos DB 데이터베이스를 만듭니다.

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **create** 명령이 실행을 완료하고 반환할 때까지 기다린 후 이 랩을 계속 진행합니다.

1. 다음 명령을 사용하여 **Azure Cosmos DB SQL API** 컨테이너를 관리하기 위한 Azure CLI 명령을 확인합니다.

    ```
    az cosmosdb sql container --help
    ```

1. 미리 정의된 변수, 데이터베이스 이름 **cosmicworks**, 컨테이너 이름 **products**, 다음 명령을 사용하여 새 Azure Cosmos DB 컨테이너를 만듭니다.

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **create** 명령이 실행을 완료하고 반환할 때까지 기다린 후 이 랩을 계속 진행합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 접두사가 **csms** 인 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **스케일링 및 설정** 을 선택합니다.

1. **스케일링** 탭 내의 값을 관찰합니다. 특히 **처리량** 섹션에서 **수동** 옵션이 선택되었고 프로비저닝된 처리량이 **400** RU/s로 설정되어 있는지 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="adjust-the-throughput-of-an-existing-container-using-the-azure-cli"></a>Azure CLI를 사용하여 기존 컨테이너의 처리량 조정

Azure CLI를 사용하여 처리량의 수동 프로비저닝과 자동 스케일링 프로비저닝 간의 컨테이너를 마이그레이션할 수 있습니다. 컨테이너가 자동 스케일링 처리량을 사용하는 경우 CLI를 사용하여 허용되는 최대 처리량 값을 동적으로 조정할 수 있습니다.

1. **Visual Studio Code** 내에 이미 열려 있는 터미널 인스턴스로 돌아갑니다.

1. 다음 명령을 사용하여 **Azure Cosmos DB SQL API** 컨테이너 처리량을 관리하기 위한 Azure CLI 명령을 확인합니다.

    ```
    az cosmosdb sql container throughput --help
    ```

1. 다음 명령을 사용하여 수동 프로비저닝에서 자동 스케일링으로 **products** 컨테이너 처리량을 마이그레이션합니다.

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **migrate** 명령이 실행을 완료하고 반환할 때까지 기다린 후 이 랩을 계속 진행합니다.

1. 다음 명령을 사용하여 가능한 최소 처리량 값을 확인하려면 **products** 컨테이너를 쿼리합니다.

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 다음 명령을 사용하여 **products** 컨테이너의 최대 자동 스케일링 조정 처리량을 기본값 **4,000** 에서 새 값 **5,000** 으로 업데이트합니다.

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. **update** 명령이 실행을 완료하고 반환할 때까지 기다린 후 이 랩을 계속 진행합니다.

1. **Visual Studio Code** 를 닫습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 접두사가 **csms** 인 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **스케일링 및 설정** 을 선택합니다.

1. **스케일링** 탭 내의 값을 관찰합니다. 특히 **처리량** 섹션에서 **자동 스케일링** 옵션이 선택되었고 프로비저닝된 처리량이 **5,000** RU/s로 설정되어 있는지 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random
