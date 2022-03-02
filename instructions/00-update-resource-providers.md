---
lab:
  title: 리소스 공급자 사용
  module: Setup
ms.openlocfilehash: 4bb2ea5cdd9d123d1b235b28da67f295b9e26969
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025090"
---
# <a name="enable-azure-resource-providers"></a>Azure 리소스 공급자 사용

Azure 구독에 등록해야 하는 일부 리소스 공급자가 있습니다. 다음 단계에 따라 등록되었는지 확인합니다.

1. 새 웹 브라우저 창 또는 탭에서 Azure Portal(``portal.azure.com``)로 이동합니다.

1. 구독과 연결된 Microsoft 자격 증명을 사용하여 포털에 로그인합니다.

1. **홈** 페이지에서 **구독** 을 선택합니다.

    > &#128161; 또는 **&#8801;** 메뉴를 확장하고 **모든 서비스** 를 선택한 다음, **전체** 범주에서 **구독** 을 선택합니다.

1. Azure 구독을 선택합니다.

    > &#128221; 구독이 여러 개인 경우 Azure Pass를 사용하여 만든 구독을 선택합니다.

1. 구독 블레이드의 **설정** 섹션에서 **리소스 공급자** 를 선택합니다.

1. 리소스 공급자 목록에서 다음 공급자가 등록되어 있는지 확인합니다.
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; 공급자가 등록되지 않은 경우 해당 공급자를 선택한 다음, **등록** 을 선택합니다.

1. 웹 브라우저 창 또는 탭을 닫습니다.

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites
