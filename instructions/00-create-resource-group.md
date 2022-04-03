---
lab:
  title: 랩 리소스 그룹 만들기
  module: Setup
ms.openlocfilehash: c13cf83d1d7d5cb9d4902e320b44f234422367d1
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025023"
---
# <a name="create-azure-resource-group-for-lab"></a>랩의 Azure Resource 그룹 만들기

이 랩을 완료하기 전에 새로 배포된 Azure 리소스를 배포할 새 [리소스 그룹][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]을 만들어야 합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **Home** 페이지에서 **리소스 그룹을** 선택합니다.

    > &#128161; 또는 **&#8801;** 메뉴를 확장하고 **모든 서비스** 를 선택한 다음, **전체** 범주에서 **리소스 그룹을** 선택합니다.

1. **+ 만들기** 를 선택합니다.

1. **리소스 그룹 만들기** 팝업에서 다음 설정을 사용하여 새 리소스 그룹을 만들고 나머지 모든 설정은 기본값으로 둡니다.

    | **설정** | **값** |
    | ---: | :--- |
    | **구독** | 기존 Azure 구독 |
    | **리소스 그룹** | 리소스 그룹에 고유한 이름 지정 |
    | **지역** | 사용 가능한 지역을 선택합니다. |

1. 배포 작업이 완료될 때까지 기다린 후 이 작업을 계속합니다.

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal
