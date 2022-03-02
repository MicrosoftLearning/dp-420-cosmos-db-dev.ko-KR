---
lab:
  title: Azure Resource Manager 템플릿을 사용하여 Azure Cosmos DB SQL API 컨테이너 만들기
  module: Module 12 - Manage an Azure Cosmos DB SQL API solution using DevOps practices
ms.openlocfilehash: 55e5430aac14807552378acfc01791818da5f267
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025018"
---
# <a name="create-an-azure-cosmos-db-sql-api-container-using-azure-resource-manager-templates"></a>Azure Resource Manager 템플릿을 사용하여 Azure Cosmos DB SQL API 컨테이너 만들기

Azure Resource Manager 템플릿은 Azure에 배포하려는 인프라를 선언적으로 정의하는 JSON 파일입니다. Azure Resource Manager 템플릿은 Azure에 서비스를 배포하는 일반적인 IaC(Infrastructure as Code) 솔루션입니다. Bicep은 JSON 템플릿을 만드는 데 사용할 수 있는 도메인별 언어를 더 쉽게 이해하도록 정의하여 개념을 좀 더 자세히 설명합니다.

이 랩에서는 Azure Resource Manager 템플릿을 사용하여 새 Azure Cosmos DB 계정, 데이터베이스 및 컨테이너를 만듭니다. 먼저 원시 JSON에서 템플릿을 만든 다음, Bicep 도메인별 언어를 사용하여 템플릿을 만듭니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="create-azure-cosmos-db-sql-api-resources-using-azure-resource-manager-templates"></a>Azure Resource Manager 템플릿을 사용하여 Azure Cosmos DB SQL API 리소스 만들기

Azure Resource Manager의 **Microsoft.DocumentDB** 리소스 공급자를 사용하면 JSON 파일을 사용하여 계정, 데이터베이스 및 컨테이너를 배포할 수 있습니다. 파일이 복잡할 수 있지만 예측 가능한 형식을 따르며 Visual Studio Code 확장의 도움을 받아 작성할 수 있습니다.

> &#128161; 템플릿이 중단되어 구문 오류를 파악할 수 없는 경우 이 [Azure Resource Manager 템플릿 솔루션][github.com/arm-template-guide]을 가이드로 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **31-create-container-arm-template** 폴더로 이동합니다.

1. **deploy.json** 파일을 엽니다.

1. 빈 Azure Resource Manager 템플릿을 확인합니다.

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. **리소스** 배열 내에서 새 JSON 개체를 추가하여 새 Azure Cosmos DB 계정을 만듭니다.

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **리소스 종류** | *Microsoft.DocumentDB/databaseAccounts* |
    | **API 버전** | *2021-05-15* |
    | **계정 이름** | *csmsarm* &amp; *계정 이름에서 생성된 고유 문자열*  |
    | **위치** | 리소스 그룹의 현재 위치 |
    | **계정 제품 유형** | *표준* |
    | **위치** | 미국 서부만 해당 |

1. **deploy.json** 파일을 저장합니다.

1. **31-create-container-arm-template** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 이미 **31-create-container-arm-template** 폴더로 설정된 터미널을 엽니다.

1. 다음 명령을 사용하여 이 랩의 앞부분에서 만들거나 본 리소스 그룹의 이름으로 새 변수 이름인 **resourceGroup** 을 만듭니다.

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 예를 들어 리소스 그룹의 이름이 **dp420** 인 경우 명령은 **$resourceGroup="dp420"** 입니다.

1. **echo** cmdlet을 사용하여 다음 명령을 통해 터미널 출력에 **$resourceGroup** 변수의 값을 씁니다.

    ```
    echo $resourceGroup
    ```

1. 다음 명령을 사용하여 Azure CLI의 대화형 로그인 절차를 시작합니다.

    ```
    az login
    ```

1. Azure CLI는 웹 브라우저 창 또는 탭을 자동으로 엽니다. 브라우저 인스턴스 내에서 구독과 연결된 Microsoft 자격 증명을 사용하여 Azure CLI에 로그인합니다.

1. [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] 명령을 사용하여 Azure Resource Manager 템플릿을 배포합니다.

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 통합 터미널을 열어 두고 **deploy.json** 파일의 편집기로 돌아갑니다.

1. **리소스** 배열 내에서 다른 새 JSON 개체를 추가하여 새 Azure Cosmos DB SQL API 데이터베이스를 만듭니다.

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **리소스 종류** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 버전** | *2021-05-15* |
    | **계정 이름** | *csmsarm* &amp; 계정 이름에서 생성된 고유 문자열 &amp; */cosmicworks*   |
    | **리소스 ID** | *cosmicworks* |
    | **종속성** | 템플릿의 앞부분에서 만든 databaseAccount |

1. **deploy.json** 파일을 저장합니다.

1. 통합 터미널로 돌아갑니다.

1. **az deployment group create** 명령을 사용하여 Azure Resource Manager 템플릿을 배포합니다.

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 통합 터미널을 열어 두고 **deploy.json** 파일의 편집기로 돌아갑니다.

1. **리소스** 배열 내에서 다른 새 JSON 개체를 추가하여 새 Azure Cosmos DB SQL API 컨테이너를 만듭니다.

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **리소스 종류** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **API 버전** | *2021-05-15* |
    | **계정 이름** | *csmsarm* &amp; 계정 이름에서 생성된 고유 문자열 &amp; */cosmicworks/products*   |
    | **리소스 ID** | *products* |
    | **처리량** | *400* |
    | **파티션 키** | */categoryId* |
    | **종속성** | 템플릿의 앞부분에서 만든 계정 및 데이터베이스 |

1. **deploy.json** 파일을 저장합니다.

1. 통합 터미널로 돌아갑니다.

1. **az deployment group create** 명령을 사용하여 최종 Azure Resource Manager 템플릿을 배포합니다.

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 통합 터미널을 닫습니다.

## <a name="observe-deployed-azure-cosmos-db-resources"></a>배포된 Azure Cosmos DB 리소스 확인

Azure Cosmos DB SQL API 리소스가 배포되면 Azure Portal에서 해당 리소스로 이동할 수 있습니다. 데이터 탐색기를 사용하여 계정, 데이터베이스 및 컨테이너가 모두 올바르게 배포되고 구성되었는지 확인합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택하고, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택한 다음, 이 랩에서 만든 접두사가 **csmsarm** 인 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **스케일링 및 설정** 을 선택합니다.

1. **스케일링** 섹션 내의 값을 확인합니다. 특히 **처리량** 섹션에서 **수동** 옵션이 선택되었고 프로비저닝된 처리량이 **400** RU/s로 설정되어 있는지 확인합니다.

1. **설정** 섹션 내의 값을 확인합니다. 특히 **파티션 키** 값이 **/categoryId** 로 설정되어 있는지 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="create-azure-cosmos-db-sql-api-resources-using-bicep-templates"></a>Bicep 템플릿을 사용하여 Azure Cosmos DB SQL API 리소스 만들기

Bicep은 Azure Resource Manager 템플릿보다 더 간단하고 쉽게 Azure 리소스를 배포할 수 있는 효율적인 도메인별 언어입니다. 차이점\[\]을 설명하기 위해 Bicep 및 다른 이름을 사용하여 정확하게 동일한 리소스를 배포합니다.

> &#128161; 템플릿이 중단되어 구문 오류를 파악할 수 없는 경우 이 [ 템플릿 솔루션][github.com/bicep-template-guide]을 가이드로 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **31-create-container-arm-template** 폴더로 이동합니다.

1. 빈 **deploy.bicep** 파일을 엽니다.

1. 파일 내에서 새 개체를 추가하여 새 Azure Cosmos DB 계정을 만듭니다.

    ```
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: resourceGroup().location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **별칭** | *계정* |
    | **이름** | *csmsarm* &amp; 계정 이름에서 생성된 고유 문자열 |
    | **리소스 종류** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 버전** | *2021-05-15* |
    | **위치** | 리소스 그룹의 현재 위치 |
    | **계정 제품 유형** | *표준* |
    | **위치** | 미국 서부만 해당 |

1. **deploy.bicep** 파일을 저장합니다.

1. **31-create-container-arm-template** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. 다음 명령을 사용하여 이 랩의 앞부분에서 만들거나 본 리소스 그룹의 이름으로 새 변수 이름인 **resourceGroup** 을 만듭니다.

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > &#128221; 예를 들어 리소스 그룹의 이름이 **dp420** 인 경우 명령은 **$resourceGroup="dp420"** 입니다.

1. **az deployment group create** 명령을 사용하여 Bicep 템플릿을 배포합니다.

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 통합 터미널을 열어 두고 **deploy.bicep** 파일의 편집기로 돌아갑니다.

1. 파일 내에서 다른 새 개체를 추가하여 새 Azure Cosmos DB 데이터베이스를 만듭니다.

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **부모** | 템플릿의 앞부분에서 만든 계정 |
    | **별칭** | *데이터베이스* |
    | **이름** | *cosmicworks*  |
    | **리소스 종류** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API 버전** | *2021-05-15* |
    | **리소스 ID** | *cosmicworks* |

1. **deploy.bicep** 파일을 저장합니다.

1. 통합 터미널로 돌아갑니다.

1. **az deployment group create** 명령을 사용하여 Bicep 템플릿을 배포합니다.

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 통합 터미널을 열어 두고 **deploy.bicep** 파일의 편집기로 돌아갑니다.

1. 파일 내에서 다른 새 개체를 추가하여 새 Azure Cosmos DB 컨테이너를 만듭니다.

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    개체는 다음 설정으로 구성됩니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **부모** | 템플릿의 앞부분에서 만든 데이터베이스 |
    | **별칭** | *컨테이너* |
    | **이름** | *products*  |
    | **리소스 ID** | *products* |
    | **처리량** | *400* |
    | **파티션 키 경로** | */categoryId* |

1. **deploy.bicep** 파일을 저장합니다.

1. 통합 터미널로 돌아갑니다.

1. **az deployment group create** 명령을 사용하여 최종 Bicep 템플릿을 배포합니다.

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

## <a name="observe-bicep-template-deployment-results"></a>Bicep 템플릿 배포 결과 확인

Azure Resource Manager 배포와 동일한 많은 기술을 사용하여 Bicep 배포의 유효성을 검사할 수 있습니다. 계정, 데이터베이스 및 컨테이너가 배포되었는지 확인할 뿐만 아니라 6개의 모든 배포에서 배포 기록도 볼 수 있습니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **리소스 그룹** 을 선택한 다음, 이 랩의 앞부분에서 만들거나 본 리소스 그룹을 선택합니다.

1. 리소스 그룹 내에서 **배포** 창으로 이동합니다.

1. Azure Resource Manager 템플릿 및 Bicep 파일에서 6개의 배포를 확인합니다.

1. 리소스 그룹 내에서 계속해서 **개요** 창으로 이동합니다.

1. 리소스 그룹 내에서 이 랩에서 만든 접두사가 **csmsbicep** 인 **Azure Cosmos DB 계정** 리소스를 선택합니다.

1. **Azure Cosmos DB** 계정 리소스 내에서 **데이터 탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 확인합니다.

1. **SQL API** 탐색 트리 내에서 **products** 컨테이너 노드를 선택한 다음, **스케일링 및 설정** 을 선택합니다.

1. **스케일링** 섹션 내의 값을 확인합니다. 특히 **처리량** 섹션에서 **수동** 옵션이 선택되었고 프로비저닝된 처리량이 **400** RU/s로 설정되어 있는지 확인합니다.

1. **설정** 섹션 내의 값을 확인합니다. 특히 **파티션 키** 값이 **/categoryId** 로 설정되어 있는지 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
