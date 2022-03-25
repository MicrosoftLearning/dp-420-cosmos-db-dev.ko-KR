---
lab:
  title: Azure Cosmos DB SQL API SDK를 사용하여 애플리케이션 문제 해결
  module: Module 11 - Monitor and troubleshoot an Azure Cosmos DB SQL API solution
ms.openlocfilehash: e87e27c83e9aa41ed7494097ce3e0e64a2b46a2f
ms.sourcegitcommit: c3778722311b55568f083480ecc69c9b3e837a18
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/19/2022
ms.locfileid: "138025167"
---
# <a name="troubleshoot-an-application-using-the-azure-cosmos-db-sql-api-sdk"></a>Azure Cosmos DB SQL API SDK를 사용하여 애플리케이션 문제 해결

Azure Cosmos DB는 다양한 작업 유형에서 발생할 수 있는 문제를 쉽게 해결하는 데 도움이 되는 광범위한 응답 코드 집합을 제공합니다. 중요한 점은 Azure Cosmos DB용 앱을 만들 때 적절한 오류 처리를 프로그래밍하는 것입니다.

이 랩에서는 두 개의 문서 중 하나를 삽입하거나 삭제할 수 있는 메뉴 기반 프로그램을 만듭니다. 이 랩의 주요 목적은 가장 일반적인 응답 코드 중 일부를 사용하는 방법과 앱의 오류 처리 코드에서 사용하는 방법을 소개하는 것입니다.  여러 응답 코드에 대한 오류 처리를 코딩하지만 두 가지 유형의 조건만 트리거하게 됩니다.  또한 오류 처리는 응답 코드에 따라 복잡한 작업을 수행하지 않으며 화면에 메시지를 표시하거나 10초 동안 기다렸다가 작업을 다시 한 번 더 시도합니다. 

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 를 시작합니다.

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [Visual Studio Code 시작 가이드][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

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
    | **무료 계층 할인 적용** | *`Do Not Apply`* |

    > &#128221; 랩 환경에서는 새 리소스 그룹을 만들지 못하게 하는 제한 사항이 있을 수 있습니다. 이러한 경우에는 미리 만든 기존 리소스 그룹을 사용합니다.

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

1. 새로 만든 **Azure Cosmos DB** 계정 리소스로 이동하여 **키** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **URI** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **엔드포인트** 값을 사용합니다.

    1. **PRIMARY KEY** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **키** 값을 사용합니다.

1. 최소화하되 브라우저 창을 닫지는 마세요. 다음 단계에서 백그라운드 워크로드를 시작한 후 몇 분 후에 Azure Portal로 돌아올 것입니다


## <a name="import-the-microsoftazurecosmos-library-into-a-net-script"></a>Microsoft.Azure.Cosmos 라이브러리를 .NET 스크립트로 가져오기

.NET CLI에는 미리 구성된 패키지 피드에서 패키지를 가져오는 [패키지 추가][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] 명령이 포함되어 있습니다. .NET 설치는 NuGet을 기본 패키지 피드로 사용합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **26-sdk-troubleshoot** 폴더로 이동합니다.

1. **26-sdk-troubleshoot** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **26-sdk-troubleshoot** 폴더로 이미 설정된 터미널을 엽니다.

1. 다음 명령을 사용하여 NuGet에서 [ Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] 패키지를 추가합니다.

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

## <a name="run-a-script-to-create-menu-driven-options-to-insert-and-delete-documents"></a>스크립트를 실행하여 문서를 삽입하고 삭제하는 메뉴 기반 옵션 만들기.

애플리케이션을 실행하기 전에 Azure Cosmos DB 계정으로 연결해야 합니다. 

1. **Visual Studio Code** 의 **탐색기** 창에서 **26-sdk-troubleshoot** 폴더로 이동합니다.

1. **Program.cs** 코드 파일을 엽니다.

1. **endpoint** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **endpoint** 로 설정된 값으로 업데이트합니다.
  
    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; 예를 들어 엔드포인트가 **https&shy;://dp420.documents.azure.com:443/** 인 경우 C# 문은 **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";** 이 됩니다.

1. **키** 라는 기존 변수를 이전에 만든 Azure Cosmos DB 계정의 **키** 로 설정된 값으로 업데이트합니다.

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; 예를 들어 키가 **fDR2ci9QgkdkvERTQ==** 인 경우 C# 문은 **private static readonly string key = "fDR2ci9QgkdkvERTQ==";** 가 됩니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```
    > &#128221; 이것은 매우 간단한 프로그램입니다.  아래와 같이 5개의 옵션이 있는 메뉴가 표시됩니다. 미리 정의된 문서를 삽입하는 두 가지 옵션, 미리 정의된 문서를 삭제하는 두 가지 옵션, 프로그램을 종료하는 옵션입니다.

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

## <a name="time-to-insert-and-delete-documents"></a>문서를 삽입하고 삭제할 시간입니다.

1. **1** 을 선택하고 **Enter** 키를 눌러 첫 번째 문서를 삽입합니다. 프로그램에서 첫 번째 문서를 삽입하고 다음 메시지를 반환합니다.

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. 다시 **1** 을 선택하고 **Enter** 키를 눌러 첫 번째 문서를 삽입합니다. 이번에는 프로그램이 예외로 충돌합니다. 오류 스택을 살펴보면 프로그램 실패의 원인을 찾을 수 있습니다. 오류 스택에서 추출된 메시지를 통해 알 수 있듯이 처리되지 않은 예외 “충돌(409)”이 발생합니다.

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. 문서를 삽입하고 있으므로 문서를 만들 때 반환되는 일반적인 [문서 상태 코드 만들기][/rest/api/cosmos-db/create-a-document#status-codes] 목록을 검토해야 합니다. 이 코드에 대한 설명은 새 문서에 제공된 ID는 기존 문서에서 가져온 것입니다. 몇 분 전에 동일한 문서를 만들기 위해 메뉴 옵션을 실행했으므로 당연한 일입니다.

1. 스택을 자세히 알아보면 이 예외가 줄 99에서 호출되었고 차례로 줄 52에서 호출된 것을 볼 수 있습니다.

    ```
    at Program.CreateDocument1(Container Customer) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 101   at Program.Main(String[] args) in C:\WWL\Git\DP-420\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 47
    ```

1. 예상대로 줄 101을 검토하면 CreateItemAsync 작업으로 인해 오류가 발생했습니다. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. 또한 줄 101~104를 검토하면 이 코드에 오류 처리가 없는 것이 분명합니다. 이 문제를 해결해야 합니다. 

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. 오류 처리 코드가 무엇을 수행해야 하는지 결정해야 합니다. [문서 상태 코드 만들기][/rest/api/cosmos-db/create-a-document#status-codes]를 검토하여 이 작업의 가능한 모든 상태 코드에 대한 오류 처리 코드를 만들도록 선택할 수 있습니다.  이 랩에서는 이 목록(상태 코드 403 a 409)에서만 살펴보겠습니다.  반환된 다른 모든 상태 코드는 시스템 오류 메시지만 표시합니다.

    > &#128221; 403 예외가 발생하면 수행할 작업을 코딩하지만 이 랩에서는 예외로 해당 유형의 오류를 생성하지는 않습니다.

1. `CompeteTaskOnCosmosDB`라는 상위 함수에 대한 오류 처리를 추가해 보겠습니다. 함수 `Main`에서 줄 45에 대한 `while` 주기를 찾고 오류 처리를 사용하여 `CompeteTaskOnCosmosDB` 호출을 래핑합니다. 이 새 코드에서 가장 먼저 확인해야 할 점은 **catch** 에서 `CosmosException` 클래스 형식의 예외를 캡처한다는 것입니다.  이 클래스에는 Azure Cosmos DB 서비스에서 요청 완료 상태 코드를 가져오는 속성 `StatusCode`가 포함되어 있습니다. `StatusCode` 속성은 `System.Net.HttpStatusCode` 형식이므로 실제로 해당 값을 .NET [HTTP 상태 코드][dotnet/api/system.net.httpstatuscode]의 필드 이름과 비교합니다.  

    ```C#
        try
        {
            await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. 파일을 저장하고 충돌한 후에는 메뉴 프로그램을 다시 실행해야 하므로 다음 명령을 실행합니다.

    ```
    dotnet run
    ```
 
1. 다시 **1** 을 선택하고 **Enter** 키를 눌러 첫 번째 문서를 삽입합니다. 이번에는 충돌하지 않지만 발생한 일에 대해 보다 사용자에게 친숙한 메시지를 받을 수 있습니다.

    ```
    Insert Failed. 
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    Press [ENTER] to continue
    ```

1. 문서 만들기의 두 가지 특정 예외인 403 및 409를 처리한다는 것은 다행이지만 모든 유형의 작업에서 발생할 수 있는 세 가지 다른 통신 유형의 예외가 있습니다.  이러한 예외는 각각 429, 503, 408 또는 너무 많은 요청, 서비스를 사용할 수 없음, 요청 시간 초과입니다. 이러한 예외를 발견하면 다음을 수행하는 몇 가지 코드를 추가합니다. 10분 동안 기다린 다음, 문서를 한 번 더 삽입해 봅니다.  코드 외에 다음을 추가합니다.

    ```C#
                    default:
                        Console.WriteLine(e.Message);
                        break;
    ```


    예외 처리 코드:


    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; 429, 503 또는 408 예외가 발생하면 수행할 작업을 코딩하지만 이 랩에서는 해당 유형의 예외로 오류를 생성하지는 않습니다.

1. 이제 `Main` 함수는 다음과 같이 표시됩니다.

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. `CreateDocument2` 함수는 위의 변경 내용으로도 수정됩니다.

1. 마지막으로 `DeleteDocument1` 및 `DeleteDocument2` 함수는 `CreateDocument1` 함수와 유사한 적절한 오류 처리 코드로 대체할 다음 코드도 필요합니다. **CreateItemAsync** 대신 **DeleteItemAsync** 를 사용하는 것 외에 이러한 함수와 다른 유일한 차이점은 [상태 코드 삭제][/rest/api/cosmos-db/delete-a-document]가 삽입 상태 코드와 다르다는 것입니다. 삭제의 경우에는 **404** 상태 코드에 대해서만 주의하면 됩니다. 추가 사례로 `CompeteTaskOnCosmosDB` 함수의 오류 처리를 업데이트할 수 있습니다.


    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;         
    ```

1.  계속해서 일부 오류 처리를 수행하는 코드를 사용하여 삭제 함수 처리에 대한 다음 코드를 추가합니다. 또한 상태 코드 **429**, **503**, **508** 에 대한 다시 시도 논리도 포함됩니다. 다음 코드를 `default` 사례 위에 추가해야 합니다.

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;       
    ```

1. 모든 함수를 수정한 후에는 모든 메뉴 옵션을 여러 번 테스트하여 예외가 발생하고 충돌이 발생하지 않을 때 앱이 메시지를 반환하는지 확인합니다.  앱이 충돌하면 오류를 수정하고 명령을 다시 실행합니다.

    ```
    dotnet run
    ```


1. 피킹하지 않아도 작업이 완료되면 `Main` 코드가 다음과 같이 표시됩니다.

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");
    
            string consoleinputcharacter;
        
            while((consoleinputcharacter = Console.ReadLine()) != "5") 
            {
                    try
                    {
                        await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists."); 
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break; 
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompeteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;    
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## <a name="conclusion"></a>결론

대부분의 주니어 개발자 조차도 적절한 오류 처리를 모든 코드에 추가해야 한다는 것을 알고 있습니다. 이 코드의 오류 처리는 간단하지만 사용자 코드에서 강력한 오류 처리 솔루션을 만들 수 있는 Azure Cosmos DB 예외 구성 요소에 대한 기본 사항이 제공되어야 합니다.


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

