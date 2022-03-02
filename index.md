---
title: 온라인 호스팅 지침
permalink: index.html
layout: home
ms.openlocfilehash: 13dd011c620f0d260b29a807eb919d7922e3e8df
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025038"
---
이 리포지토리에는 Microsoft 과정 [DP-420 Microsoft Azure AI 솔루션 설계 및 구현][course-description]의 실습 랩 연습과 [Microsoft Learn의 동등한 자기 주도적 모듈][learn-collection]이 포함되어 있습니다. 이 연습에서는 학습 자료가 함께 제공되며 학습 자료에서 설명하는 기술을 사용하여 연습할 수 있도록 설계되었습니다.

> &#128221; 이러한 연습을 완료하려면 Microsoft Azure 구독이 필요합니다. 강사가 Microsoft Azure 구독을 제공하지 않은 경우 [https://azure.microsoft.com][azure]에서 무료 평가판에 등록할 수 있습니다.

## <a name="labs"></a>랩

{% assign labs = site.pages | where_exp:"page", "page.url contains '/instructions'" %}
| 모듈 | 랩 |
| --- | --- |
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

[azure]: https://azure.microsoft.com
[course-description]: https://docs.microsoft.com/learn/certifications/courses/dp-420t00
[learn-collection]: https://docs.microsoft.com/users/msftofficialcurriculum-4292/collections/1k8wcz8zooj2nx
