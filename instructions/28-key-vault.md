---
lab:
  title: Azure Cosmos DB SQL API 계정 키를 Azure Key Vault에 저장
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: 929b8c5078ff27ae474f86393ad61f44dd3b66b3
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025078"
---
# <a name="store-azure-cosmos-db-sql-api-account-keys-in-azure-key-vault"></a>Azure Cosmos DB SQL API 계정 키를 Azure Key Vault에 저장

Azure Cosmos DB 계정 연결 코드를 애플리케이션에 추가하는 것은 계정의 URI 및 키를 제공하는 것만큼 간단합니다. 이 보안 정보는 경우에 따라 애플리케이션 코드에 하드 코딩될 수 있습니다. 그러나 애플리케이션이 Azure App Service에 배포되는 경우 암호화 연결 정보를 Azure Key Vault에 저장할 수 있습니다.

이 랩에서는 Azure Cosmos DB 계정 연결 문자열을 암호화하고 Azure Key Vault에 저장합니다. 그런 다음 Azure Key Vault에서 해당 자격 증명을 검색하는 Azure App Service 웹앱을 만듭니다. 애플리케이션은 이러한 자격 증명을 사용하여 Azure Cosmos DB 계정에 연결합니다. 그런 다음 애플리케이션이 Azure Cosmos DB 계정 컨테이너에 일부 문서를 만들고 해당 상태를 웹 페이지로 다시 반환합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 *Visual Studio Code를_ _ **닫습니다***. 나중에 **28-key-vault** 폴더를 직접 가리키는 파일을 엽니다.

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

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **기본 연결 문자열** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **연결 문자열** 값을 사용합니다.

## <a name="create-an-azure-key-vault-and-store-the-azure-cosmos-db-account-credentials-as-a-secret"></a>Azure Key Vault를 만들고 Azure Cosmos DB 계정 자격 증명을 비밀로 저장

웹앱을 만들기 전에 암호화된 비밀 *Azure Key Vault* 에 복사하여 Azure Cosmos DB 계정 연결 문자열을 보호합니다. 이제 방법을 알아보겠습니다.

1. Azure Portal에서 **키 자격 증명 모음** 페이지로 이동합니다.

1. * **+ 만들기** _ 단추를 선택하여 자격 증명 모음을 추가하여 다음 설정으로 자격 증명 모음을 작성하고 _나머지 모든 설정을 기본값으로 유지한 다음*, 자격 증명 모음을 만들도록 선택합니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | 기존 Azure 구독 |
    | **리소스 그룹** | 기존 리소스 그룹을 선택하거나 새로 만듭니다. |
    | **Key Vault 이름** | 전역적으로 고유한 이름을 입력합니다. |
    | **지역** | 사용 가능한 지역을 선택합니다. |

1. 자격 증명 모음이 만들어지면 해당 자격 증명 모음으로 이동합니다.

1. 설정 섹션에서 **비밀** 을 선택합니다.

1. **+ 생성/가져오기** 를 선택하여 자격 증명 연결 문자열을 암호화하여 다음 설정으로 비밀 값을 입력하고 나머지 모든 설정을 기본값으로 유지한 다음, 비밀을 만들도록 선택합니다. 

    | **설정** | **값** |
    | ---: | :--- |
    | **업로드 옵션** | *수동* |
    | **이름** | 비밀에 레이블을 지정할 이름 |
    | **값** | 이 필드는 작성할 가장 중요한 필드입니다. 이 값은 Azure Cosmos DB 계정의 키 섹션에서 이전에 복사한 기본 연결 문자열입니다. 이 값은 비밀로 변환됩니다. |
    | **Enabled** | *예* |
 
1. 이제 비밀 아래에 새로운 비밀이 나열됩니다. 웹앱의 코드에 추가할 비밀 식별자를 가져와야 합니다. 만든 **비밀** 을 선택합니다.

1. Azure Key Vault를 사용하면 여러 버전의 비밀을 만들 수 있지만 이 랩에서는 하나의 버전만 필요합니다. **현재 버전** 을 선택합니다.

1. **비밀 식별자** 필드의 값을 기록합니다. 이 값은 Key Vault에서 비밀을 얻기 위해 애플리케이션의 코드에서 사용할 값입니다.  이 값은 URL입니다. 이 비밀을 제대로 작동시키기 위해서는 한 단계가 더 있지만 그 단계는 잠시 후에 수행하겠습니다.

## <a name="create-an-azure-app-service-webapp"></a>Azure App Service 웹앱 만들기

Azure Cosmos DB 계정에 연결하고 일부 컨테이너 및 문서를 만드는 웹앱을 만듭니다. 이 앱에서 Azure Cosmos DB 자격 증명을 하드 코딩하지 않고 대신 키 자격 증명 모음에서 **비밀 식별자** 를 하드 코딩합니다. Azure 레이어의 웹앱에 할당된 적절한 권한이 없으면 이 식별자가 얼마나 불필요해지는지 살펴보겠습니다. 코딩을 시작하겠습니다.



1. **Visual Studio Code** 를 사용하여 ASP.NET 5 API 앱을 만드는 방법을 보여줍니다.  파일 >열기 폴더를 선택하고 **28-key-vault** 폴더에 대한 모든 경로를 탐색하여 **28-key-vault** 폴더를 엽니다.

    > &#128221; 참고로 **28-key-vault** 폴더와 해당 파일 및 하위 폴더만 **탐색기** 트리에 표시됩니다. 앞에서 복제한 전체 GitHub 리포지토리가 표시되면 웹앱이 제대로 작동하지 않게 되므로 **28-key-vault** 폴더와 해당 파일 및 하위 폴더만 **탐색기** 트리에서 볼 수 있는지 확인합니다.

1. **28-key-vault** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **28-key-vault** 폴더로 이미 설정된 터미널을 엽니다.

1. MVC 웹앱 셸을 만들어 보겠습니다. 생성된 파일 중 몇 개는 잠시 후에 교체할 예정입니다. 다음 명령을 실행하여 웹앱을 만듭니다.

    ```
    dotnet new mvc
    ```


1. 이 명령은 웹앱의 셸을 만들어 여러 파일 및 디렉터리를 추가했습니다. 이미 필요한 모든 코드가 포함된 파일이 몇 개 있습니다. `.\KeyvaultFiles` 디렉터리의 해당 파일에 대해 `.\Controllers\HomeController.cs` 파일과 `.\Views\Home\Index.cshtml` 파일을 바꿉니다.

1. 파일을 바꾸면 `.\KeyvaultFiles` 디렉터리를 삭제합니다.

## <a name="import-the-multiple-missing-libraries-into-the-net-script"></a>누락된 여러 라이브러리를 .NET 스크립트로 가져오기

.NET CLI에는 미리 구성된 패키지 피드에서 패키지를 가져오는 [패키지 추가][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 명령이 포함되어 있습니다. .NET 설치는 NuGet을 기본 패키지 피드로 사용합니다.

1. 만약 그렇게 하지 않았다면 **Visual Studio Code** 의 **탐색기** 창에서 **28-key-vault** 폴더를 찾아봅니다.

1. 만약 그렇게 하지 않았다면 **28-key-vault** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **28-key-vault** 폴더로 이미 설정된 터미널을 엽니다.

1. 다음 명령을 사용하여 NuGet에서 [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 다음 명령을 사용하여 NuGet에서 [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] 패키지를 추가합니다.

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. 다음 명령을 사용하여 NuGet에서 [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. 다음 명령을 사용하여 NuGet에서 [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication
    ```

## <a name="adding-the-secret-identifier-to-your-webapp"></a>웹앱에 비밀 식별자 추가

1. Visual Studio에서 `.\Controllers\HomeControler.cs` 파일을 엽니다.

1. **GetKeyVaultSecret** 사용자 정의 함수는 Azure Cosmos DB 계정 비밀을 가져옵니다. 함수는 줄 98에서 시작하며 아래 스크립트와 유사해야 합니다.

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. 이 함수가 만드는 중요한 호출을 검토해 보겠습니다.

    - 줄 100에서는 현재 웹앱의 토큰을 정의합니다. 이 토큰은 자격 증명 모음에 액세스하려는 앱을 식별하기 위해 Azure Key Vault에 제공됩니다. 
    - 줄 104~105에서는 Azure Key Vault에 연결할 Key Vault 클라이언트를 준비합니다.  웹앱 토큰을 매개 변수로 보냅니다. 
    - 줄 107~108에서는 Key Vault 클라이언트에 해당 키 자격 증명 모음에 저장된 비밀을 반환하는 **비밀 식별자** URL 주소를 제공합니다. 

1.  웹앱을 배포하기 전에 **비밀 식별자** URL을 보내야 합니다.  줄 107에서는 문자열 **`<Key Vault Secret Identifier>`** 를 비밀 섹션에 기록한 **비밀 식별자** URL로 바꾸고 파일을 저장합니다. 

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

## <a name="optional-install-the-azure-app-services-extension"></a>(선택 사항) Azure App Services 확장 설치

Visual Studio에서 명령 팔레트(**Ctrl+Shift+P**)를 가져오고 Azure 앱 리소스 명령을 검색할 때 아무 것도 반환하지 않으면 확장을 설치해야 합니다.

1. 왼쪽 Visual Studio Code 메뉴에서 **확장** 옵션을 선택합니다.

1. 검색 창에서 Azure App Service를 검색하고 선택합니다.

1. 설치 단추를 선택하여 설치합니다.

1. **확장** 탭을 닫고 코드로 돌아갑니다.

## <a name="deploy-your-application-to-azure-app-services"></a>Azure App Service에 애플리케이션 배포

나머지 코드는 간단합니다. 연결 문자열을 가져와서 Azure Cosmos DB에 연결하고 일부 문서를 추가합니다. 또한 애플리케이션은 문제에 대한 몇 가지 피드백을 제공해야 합니다. 애플리케이션을 배포한 후에는 더 이상 변경할 필요가 없습니다. 이제 시작하겠습니다. 

> &#128221; 아래 단계는 대부분 Visual Studio 화면의 중앙 상단에 있는 명령 팔레트에서 실행됩니다.

1. Visual Studio Code에서 명령 팔레트를 열고 Azure App Service: 새 웹앱 만들기...(고급)를 검색합니다.

1. Azure에 로그인...을 선택합니다. 이 옵션은 웹 브라우저 창을 열고 로그인 프로세스를 수행한 후, 완료되면 브라우저를 닫습니다.

1. 웹앱의 전역적으로 고유한 이름을 입력합니다.

1. 기존 리소스 그룹을 선택하거나 필요한 경우 새 리소스 그룹을 만듭니다.

1. **.NET 6(LTS)** 를 선택합니다.

1. **Windows** 를 선택합니다.

1. 사용 가능한 위치를 선택합니다.

1. **+ 새 App Service 요금제 만들기** 를 선택합니다.

1. App Service 요금제의 기본 이름을 적용하거나(웹앱 이름과 동일해야 함) 새 이름을 선택합니다.

1. **Free (F1) Try out Azure at no cost**(무료(F1) 무료로 Azure 사용해 보기)를 선택합니다.

1. Application Insights에 대해 **지금은 건너뛰기** 를 선택합니다.

1. 이제 배포가 오른쪽 아래 모서리에 있는 상태 표시줄로 실행되어야 합니다. 

1. 메시지가 나타나면 **배포** 를 선택합니다.

1. **찾아보기** 를 선택하면 **28-key-vault** 폴더 내에 있어야 합니다. 해당 폴더를 선택하세요.

1. **배포에 필요한 구성이 “28-key-vault”에서 누락되었습니다** 는 메시지가 표시된 팝업이 나타나면 구성 추가 단추를 선택합니다.  이 옵션은 누락된 `.vscode` 폴더를 만듭니다.

    > &#128221; 매우 중요한 것은 앱을 처음 배포할 때 이 팝업이 표시되지 않으면 Azure App Services에 업로드할 파일이 누락된다는 것입니다. 배포는 성공하지만 웹 사이트에서 항상 이 디렉터리 또는 페이지를 볼 수 있는 권한이 없습니다는 메시지를 반환합니다. 가장 가능성이 높은 원인은 Visual Studio Code가 **28-key-vault** 폴더 대신 GitHub 복제된 리포지토리에서 열렸다는 것입니다.

1. 항상 해당 작업 영역에 배포하라는 메시지가 표시되면 **예** 를 선택합니다.

1. 메시지가 표시되면 웹 사이트 찾아보기를 선택합니다.  또는 브라우저를 열고 **`https://<yourwebappname>.azurewebsites.net`** 으로 이동합니다. 두 경우 모두 문제가 있습니다. 웹 페이지에서 사용자 정의 메시지를 받았어야 합니다. 확장된 오류 메시지와 함께 **Key Vault에 액세스할 수 없습니다** 라는 메시지가 표시되어야 합니다. 이 문제를 해결해 보겠습니다.

## <a name="allow-our-app-to-use-a-managed-identity"></a>앱에서 관리 ID 사용 허용

해결해야 하는 첫 번째 문제는 앱에서 관리 ID를 사용할 수 있도록 하는 것입니다. 관리 ID를 사용하면 앱에서 Azure Key Vault와 같은 Azure 서비스를 사용할 수 있습니다.

1. 브라우저를 열고 Azure Portal에 로그인합니다.

1. **App Services** 페이지를 엽니다. 나열되어 있는 사용자의 웹앱 이름을 선택합니다.

1. 설정 섹션에서 **ID** 를 선택합니다.

1. 상태에서 **켜기** 및 **저장** 을 선택합니다.  할당한 관리 ID를 사용하도록 설정하라는 메시지가 표시되면 **예** 를 선택합니다.

1. 웹앱을 다시 시도해 보겠습니다.  브라우저에서 **`https://<yourwebappname>.azurewebsites.net`** 으로 이동합니다.

1. 여전히 한 가지 문제가 있습니다. 첫 번째 메시지는 프로그램에서 보내고 있는 사용자 정의 메시지인 반면 두 번째 메시지는 시스템에서 생성된 메시지입니다. 두 번째 메시지는 Key Vault에 연결할 수 있는 액세스 권한은 부여되었지만 자격 증명 모음 내의 비밀을 볼 수 있는 액세스 권한은 부여되지 않았다는 것을 의미합니다.  이 문제를 해결하기 위해 마지막 설정을 하나 설정해 보겠습니다.

## <a name="granting-our-web-application-an-access-policy-to-the-key-vault-secrets"></a>웹 애플리케이션에 Key Vault 비밀에 대한 액세스 정책 부여

이 랩의 원래 목표는 Azure Cosmos DB 계정이 애플리케이션에서 하드 코딩되는 것을 방지하는 것이었습니다. 그러나 누구나 볼 수 있는 **비밀 식별자** URL을 하드 코딩했습니다. 그렇다면 자격 증명을 어떻게 보호할 수 있을까요? 다행인 것은 비밀 식별자 자체가 쓸모가 없다는 것입니다. **비밀 식별자** 는 Azure Key Vault 문으로만 이동하지만 자격 증명 모음은 누가 들어오고 누가 문에 머무르는지 결정합니다. 즉, 애플리케이션에 대한 Key Vault 액세스 정책을 만들어 해당 자격 증명 모음의 비밀을 볼 수 있도록 해야 한다는 것입니다. 해당 솔루션을 살펴보겠습니다.

1. (선택 사항) 정책을 만들기 전에 Azure Cosmos DB 데이터베이스의 현재 콘텐츠를 검토해 보겠습니다.  Azure Portal에서 Azure Cosmos DB 계정으로 이동하면 **GlobalCustomers** 데이터베이스가 있나요? 없는 경우에는 웹앱이 성공적으로 실행되어 만들어집니다. 있는 경우에는 데이터베이스의 항목 수를 검토하고 웹앱을 성공적으로 실행하여 더 많은 항목을 추가합니다.

1. Azure Portal에서 이전에 만든 Key Vault로 이동합니다.

1. *설정* 섹션에서 **액세스 정책** 을 선택합니다.

1. **+ 액세스 정책 추가** 를 선택합니다.

1. 다음 설정으로 액세스 정책 값을 입력하고 나머지 모든 설정을 기본값으로 유지한 다음, 선택하여 정책을 추가합니다. 

    | **설정** | **값** |
    | ---: | :--- |
    | **키 권한** | *가져오기* |
    | **비밀 권한** | *가져오기* |
    | **보안 주체 선택** | 애플리케이션 이름 검색 및 선택 |

    > &#128221; 권한 있는 애플리케이션을 선택하지 마세요.

1. 새 정책을 **저장** 합니다.

1. 웹앱을 다시 시도해 보겠습니다.  브라우저에서 **`https://<yourwebappname>.azurewebsites.net`** 으로 이동합니다.

1. 성공! 웹 페이지에서 고객 컨테이너에 새 항목을 삽입했음을 나타내야 합니다. 또한 실제 비밀이 표시되는 것을 볼 수도 있습니다.

    > &#128221; 프로덕션 환경에서는 절대 비밀을 표시하지 **않습니다**. 이 작업은 단지 설명을 목적으로만 수행되었습니다.


1. Azure Cosmos DB 계정으로 이동하여 데이터가 포함된 새 **GlobalCustomers** 데이터베이스가 있는지 또는 데이터베이스가 이미 있는지, 데이터베이스에 더 많은 항목이 있는지 확인합니다.

이제 Azure Key Vault를 사용하여 Azure Cosmos DB 계정의 키를 성공적으로 보호했습니다.
