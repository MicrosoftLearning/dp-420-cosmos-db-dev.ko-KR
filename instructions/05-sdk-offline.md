---
lab:
  title: 오프라인 개발을 위한 Azure Cosmos DB SQL API SDK 구성
  module: Module 3 - Connect to Azure Cosmos DB SQL API with the SDK
ms.openlocfilehash: d6d5bad51d4adc029e901352f0becc9268acab3e
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025079"
---
# <a name="configure-the-azure-cosmos-db-sql-api-sdk-for-offline-development"></a>오프라인 개발을 위한 Azure Cosmos DB SQL API SDK 구성

Azure Cosmos DB Emulator는 개발 및 테스트를 위해 Azure Cosmos DB 서비스를 에뮬레이트하는 로컬 도구입니다. 에뮬레이터는 SQL API를 지원하며 .NET용 Azure SDK를 사용하여 코드를 개발할 때 클라우드 서비스 대신 사용할 수 있습니다.

이 랩에서는 .NET용 Azure SDK에서 Azure Cosmos DB Emulator에 연결합니다.

## <a name="prepare-your-development-environment"></a>개발 환경 준비

**DP-420** 에 대한 랩 코드 리포지토리를 이 랩에서 작업 중인 환경에 아직 복제하지 않은 경우 다음 단계를 수행합니다. 그렇지 않으면 이전에 복제한 폴더를 **Visual Studio Code** 에서 엽니다.

1. **Visual Studio Code** 시작

    > &#128221; Visual Studio Code 인터페이스에 익숙하지 않은 경우 [시작 설명서][code.visualstudio.com/docs/getstarted]를 검토하세요.

1. 명령 팔레트를 열고 **Git: Clone** 을 실행하여 선택한 로컬 폴더에 ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub 리포지토리를 복제합니다.

    > &#128161; **CTRL+SHIFT+P** 바로 가기 키를 사용하여 명령 팔레트를 열 수 있습니다.

1. 리포지토리가 복제되면 **Visual Studio Code** 에서 선택한 로컬 폴더를 엽니다.

## <a name="start-the-azure-cosmos-db-emulator"></a>Azure Cosmos DB Emulator 시작

환경에 이미 에뮬레이터가 사전 설치되어 있어야 합니다. 그렇지 않은 경우 [설치 지침][docs.microsoft.com/azure/cosmos-db/local-emulator]을 참조하여 Azure Cosmos DB Emulator를 설치합니다. 에뮬레이터가 시작되면 연결 문자열을 검색하여 .NET용 Azure SDK 또는 선택한 다른 SDK를 사용하여 에뮬레이터에 연결할 수 있습니다.

1. **Azure Cosmos DB Emulator** 를 시작합니다.

    > &#128221; 에뮬레이터를 시작하기 위해 관리자에게 액세스 권한을 부여하라는 메시지가 표시될 수 있습니다. 랩 환경에서 **관리자** 계정에는 **학생** 계정과 동일한 암호가 있습니다.

    > &#128161; Azure Cosmos DB Emulator는 Windows 작업 표시줄과 시작 메뉴 모두에 고정됩니다.

1. 에뮬레이터가 기본 브라우저를 자동으로 열고 **localhost:8081/_explorer/index.html** 방문 페이지로 이동할 때까지 기다립니다.

1. **Azure Cosmos DB Emulator** 방문 페이지에서 **탐색기** 창으로 이동합니다.

1. 이 창에는 SDK에서 계정에 연결하는 데 필요한 연결 세부 정보 및 자격 증명이 포함되어 있습니다. 특히:

    1. **기본 연결 문자열** 필드의 값을 기록합니다. 이 연습의 뒷부분에서 이 **연결 문자열** 값을 사용합니다.

1. **탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **SQL API** 탐색 트리 내에 노드가 없는지 확인합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="connect-to-the-emulator-from-the-sdk"></a>SDK에서 에뮬레이터에 연결

**Microsoft.Azure.Cosmos** 라이브러리는 이 연습에서 사용할 .NET 스크립트에 이미 사전 설치되어 있습니다. 또한 시간을 절약하기 위해 상용구 코드 중 일부가 이미 작성되었습니다. 상용구 연결 문자열 값을 업데이트하고 스크립트를 완료하려면 몇 줄의 코드를 작성해야 합니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **05-sdk-offline** 폴더로 이동합니다.

1. **05-sdk-offline** 폴더 내에서 **script.cs** 코드 파일을 엽니다.

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** 라이브러리는 이미 NuGet에서 미리 가져왔습니다.

1. 값이 Azure Cosmos DB Emulator의 **연결 문자열** 로 설정된 **connectionString** 이라는 기존 변수를 업데이트합니다.
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; 에뮬레이터의 URI는 일반적으로 ``localhost:[port]`` 기본 포트가 **8081** 로 설정된 SSL을 사용합니다.

    > &#128221; ``C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==``는 에뮬레이터의 모든 설치에 대한 기본 키입니다. 이 키는 명령줄 옵션을 사용하여 변경할 수 있습니다.

1. 에뮬레이터 내에서 만들려는 새 데이터베이스(**cosmicworks**)의 이름을 전달하고 결과를 [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] 형식의 변수에 저장하는 **client** 변수의 [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] 메서드를 비동기적으로 호출합니다.

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **새 컨테이너** 라는 헤더와 함께 Database 클래스의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] 속성을 인쇄합니다.

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **05-sdk-offline** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

    > &#128221; 이 명령은 시작 디렉터리가 **05-sdk-offline** 폴더로 이미 설정된 터미널을 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

## <a name="view-the-changes-in-the-emulator"></a>에뮬레이터의 변경 내용 보기

이제 Azure Cosmos DB 에뮬레이터에서 새 데이터베이스를 만들었으므로 온라인 **데이터 탐색기** 를 사용하여 에뮬레이터 내의 새 SQL API 데이터베이스를 관찰합니다.

1. Windows 시스템 트레이에서 에뮬레이터 아이콘으로 이동하고 상황에 맞는 메뉴를 연 다음, **데이터 탐색기 열기...** 를 선택하여 기본 브라우저를 사용하여 **localhost:8081/_explorer/** 방문 페이지로 이동합니다.

1. **Azure Cosmos DB Emulator** 방문 페이지에서 **탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **SQL API** 탐색 트리 내의 새 **cosmicworks** 데이터베이스 노드를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="create-and-view-a-new-container"></a>새 컨테이너 만들기 및 보기

새 컨테이너를 만드는 것은 새 데이터베이스를 만드는 데 사용되는 패턴과 유사합니다. 여기서 학습하는 코드는 클라우드 또는 에뮬레이터에서 리소스를 만드는지 여부에 관계없이 연결 문자열을 변경하기만 하면 됩니다. 스크립트 파일을 더 확장하여 데이터베이스와 함께 새 컨테이너를 만듭니다.

1. **Visual Studio Code** 의 **탐색기** 창에서 **05-sdk-offline** 폴더로 이동합니다.

1. **05-sdk-offline** 폴더 내에서 **script.cs** 코드 파일을 다시 엽니다.

1. 새 컨테이너(**products**), 파티션 키 경로( **/categoryId**) 및 **cosmicworks** 데이터베이스 내에서 만들려는 처리량(**400**)을 전달하고 결과를 [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 형식의 변수에 저장하는 **database** 변수의 [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] 메서드를 비동기적으로 호출합니다.

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 기본 제공 **Console.WriteLine** 정적 메서드를 사용하여 **새 컨테이너** 라는 헤더와 함께 Container 클래스의 [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] 속성을 인쇄합니다.

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 완료되면 코드 파일에 다음이 포함됩니다.
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **script.cs** 코드 파일을 **저장** 합니다.

1. **Visual Studio Code** 에서 **05-sdk-offline** 폴더의 상황에 맞는 메뉴를 연 다음, **통합 터미널에서 열기** 를 선택하여 새 터미널 인스턴스를 엽니다.

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] 명령을 사용하여 프로젝트를 빌드하고 실행합니다.

    ```
    dotnet run
    ```

1. 통합 터미널을 닫습니다.

1. **Visual Studio Code** 를 닫습니다.

1. Windows 시스템 트레이에서 에뮬레이터 아이콘으로 이동하고 상황에 맞는 메뉴를 연 다음, **데이터 탐색기 열기...** 를 선택하여 기본 브라우저를 사용하여 **localhost:8081/_explorer/** 방문 페이지로 이동합니다.

1. **Azure Cosmos DB Emulator** 방문 페이지에서 **탐색기** 창으로 이동합니다.

1. **데이터 탐색기** 에서 **cosmicworks** 데이터베이스 노드를 확장한 다음, **SQL API** 탐색 트리 내에서 새 **products** 컨테이너 노드를 관찰합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

## <a name="stop-the-azure-cosmos-db-emulator"></a>Azure Cosmos DB Emulator 중지

사용자 환경에서 시스템 리소스를 사용할 수 있으므로 에뮬레이터 사용을 완료하면 에뮬레이터를 중지하는 것이 중요합니다. 시스템 트레이 아이콘을 사용하여 에뮬레이터 및 실행 중인 모든 인스턴스를 중지합니다.

1. Windows 시스템 트레이에서 에뮬레이터 아이콘으로 이동하고 상황에 맞는 메뉴를 연 다음, **종료** 를 선택하여 에뮬레이터를 종료합니다.

    > &#128221; 에뮬레이터의 모든 인스턴스가 종료되는 데 1분 정도 걸릴 수 있습니다.

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
