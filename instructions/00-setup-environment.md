---
lab:
  title: 랩 환경 설정
  module: Setup
ms.openlocfilehash: 6beb401fed4863929ed8772e2e8b3dff4b61434b
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025063"
---
# <a name="setup-local-lab-environment"></a>로컬 랩 환경 설정

호스트된 랩 환경에서 이러한 랩을 완료하는 것이 가장 좋습니다. 사용자의 개인 컴퓨터에서 완료하려면 다음 소프트웨어를 설치하여 완료하면 됩니다. 사용자의 개인 환경을 사용할 경우 예기치 않은 대화 상자와 동작이 발생할 수 있습니다. 가능한 로컬 구성이 광범위하기 때문에 과정 팀에서는 사용자의 개인 환경에서 발생할 수 있는 문제를 지원할 수 없습니다.

## <a name="windows-installation"></a>Windows 설치

> &#128221; 아래 지침은 Windows 10 컴퓨터용입니다. Linux 또는 MacOS를 사용할 수도 있습니다. 선택한 OS의 랩 지침을 조정해야 할 수도 있습니다.

### <a name="windows-10-os"></a>Windows 10(OS)

1. Windows 10(버전 2004 이상)을 설치합니다.

1. 사용 가능한 모든 업데이트를 적용합니다.

### <a name="edge"></a>Edge

1. [microsoft.com/edge]에서 최신 버전의 Microsoft Edge를 설치합니다.

### <a name="net-6-sdk"></a>.NET 6 SDK

1. [dotnet.microsoft.com/download/dotnet/6.0]에서 SDK(런타임 아님)를 다운로드하여 설치합니다.

### <a name="powershell-7"></a>PowerShell 7

1. [github.com/powershell/powershell/releases]에서 다운로드하여 설치합니다.

### <a name="git"></a>Git

1. [git-scm.com/downloads]에서 다운로드하여 설치합니다.

    - 설치 관리자에서 기본 옵션을 사용합니다.

### <a name="windows-terminal"></a>Windows 터미널

1. [github.com/microsoft/terminal/releases]에서 다운로드하여 설치합니다.

1. **PowerShell** 을 기본 터미널로 구성

### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code(및 확장)

1. [code.visualstudio.com/download]에서 다운로드하여 설치합니다.

    - 설치 관리자에서 기본 옵션을 사용합니다.

1. 설치 후 Visual Studio Code를 시작합니다.

1. **확장** 메뉴에서 다음 Microsoft 확장을 검색하여 설치합니다.

    - [C#][marketplace.visualstudio.com/ms-dotnettools.csharp]

### <a name="azure-cosmos-db-emulator"></a>Azure Cosmos DB 에뮬레이터

1. [docs.microsoft.com/azure/cosmos-db/local-emulator]에서 다운로드하여 설치합니다.
    - 설치 관리자에서 기본 옵션을 사용합니다.

[code.visualstudio.com/download]: https://code.visualstudio.com/download
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[dotnet.microsoft.com/download/dotnet/6.0]: https://dotnet.microsoft.com/download/dotnet/6.0
[git-scm.com/downloads]: https://git-scm.com/downloads
[github.com/microsoft/terminal/releases]: https://github.com/microsoft/terminal/releases/latest
[github.com/powershell/powershell/releases]: https://github.com/powershell/powershell/releases/latest
[marketplace.visualstudio.com/ms-dotnettools.csharp]: https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp
[microsoft.com/edge]: https://microsoft.com/edge
