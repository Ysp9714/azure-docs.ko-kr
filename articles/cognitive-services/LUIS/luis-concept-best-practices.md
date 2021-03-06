---
title: LUIS 앱 빌드에 대 한 모범 사례
description: LUIS 앱 모델에서 최상의 결과를 얻기 위한 모범 사례를 알아봅니다.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 05/17/2020
ms.openlocfilehash: fc4475eb64744688e901d2b37e30632d044b0115
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95019316"
---
# <a name="best-practices-for-building-a-language-understanding-luis-app"></a>언어 이해 (LUIS) 앱 빌드에 대 한 모범 사례
앱 제작 프로세스를 사용 하 여 LUIS 앱을 빌드합니다.

* 빌드 언어 모델 (의도 및 엔터티)
* 몇 가지 교육 예 길이 발언 (내재 된 15-30)를 추가 합니다.
* 끝점에 게시
* 엔드포인트에서 테스트

앱이 [게시](luis-how-to-publish-app.md)되 면 개발 수명 주기를 사용 하 여 끝점에서 기능을 추가 하 고, 게시 하 고, 테스트 합니다. 길이 발언 예제를 추가 하 여 다음 제작 주기를 시작 하지 마세요. LUIS는 실제 사용자 길이 발언를 사용 하 여 모델을 학습할 수 없기 때문입니다.

두 예제의 현재 집합 및 끝점 길이 발언가 확신, 높은 예측 점수를 반환할 때까지 길이 발언를 확장 하지 마십시오. [활성 학습](luis-concept-review-endpoint-utterances.md)을 사용 하 여 점수를 향상 합니다.




## <a name="do-and-dont"></a>허용 및 금지
다음 목록에는 LUIS 앱의 모범 사례가 포함되어 있습니다.

|수행|안 함|
|--|--|
|[스키마 계획](#do-plan-your-schema)|[계획 없이 빌드 및 게시](#dont-publish-too-quickly)|
|[고유한 의도 정의](#do-define-distinct-intents)<br>[의도에 기능 추가](#do-add-features-to-intents)<br>
[컴퓨터에서 배운 엔터티 사용](#do-use-machine-learned-entities) |[의도에 많은 예제 발화 추가](#dont-add-many-example-utterances-to-intents)<br>[소수의 엔터티 또는 단순 엔터티 사용](#dont-use-few-or-simple-entities) |
|[각 의도에 너무 일반적 및 너무 구체적 사이에 안정적인 지점 찾기](#do-find-sweet-spot-for-intents)|[LUIS를 학습 플랫폼으로 사용](#dont-use-luis-as-a-training-platform)|
|[버전을 사용 하 여 반복적으로 앱 빌드](#do-build-your-app-iteratively-with-versions)<br>[모델 분해를 위한 엔터티 빌드](#do-build-for-model-decomposition)|[동일한 형식의 많은 예제 발화를 추가하여 다른 형식 무시](#dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats)|
|[이후 반복에서 패턴 추가](#do-add-patterns-in-later-iterations)|[의도 및 엔터티 정의 혼합](#dont-mix-the-definition-of-intents-and-entities)|
|None 의도를 제외한 [모든 의도에서 발언의 균형 맞추기](#balance-your-utterances-across-all-intents)<br>[None 의도에 예제 발화 추가](#do-add-example-utterances-to-none-intent)|[모든 가능한 값을 사용하여 구문 목록 만들기](#dont-create-phrase-lists-with-all-the-possible-values)|
|[활성 학습의 제안 기능 활용](#do-leverage-the-suggest-feature-for-active-learning)|[너무 많은 패턴 추가](#dont-add-many-patterns)|
|[Batch 테스트를 사용 하 여 앱 성능 모니터링](#do-monitor-the-performance-of-your-app)|[추가된 모든 단일 예제를 사용하여 학습 및 게시](#dont-train-and-publish-with-every-single-example-utterance)|

## <a name="do-plan-your-schema"></a>스키마 계획

앱의 스키마 빌드를 시작 하기 전에이 앱을 사용 하려는 및 위치를 식별 해야 합니다. 더 철저 하 고 구체적으로 계획 하면 앱이 더 좋아집니다.

* 대상 사용자 조사
* 앱을 나타내는 종단 간 가상 사용자 정의-음성, 아바타, 문제 처리 (예방적, 반응)
* 채널을 사용 하 여 사용자 조작 (텍스트, 음성) 확인, 기존 솔루션에 전달 또는이 앱에 대 한 새 솔루션 만들기
* 종단 간 사용자 경험
    * 이 앱이 수행 해야 하는 작업은 무엇 인가요? * 수행 해야 하는 작업의 우선 순위는 무엇 인가요?
    * 주요 사용 사례는 무엇 인가요?
* 데이터 수집-데이터 수집 및 준비에 [대해 알아보기](data-collection.md)

## <a name="do-define-distinct-intents"></a>고유한 의도 정의
각 의도의 어휘가 해당 의도에만 사용되고 다른 의도와 겹치지 않는지 확인합니다. 예를 들어, 항공사 항공편 및 호텔과 같은 여행 계획을 처리하는 앱을 사용하려는 경우, 이러한 주제 영역을 발언 내부에 별도의 의도로 포함하거나 특정 데이터의 엔터티가 있는 동일한 의도로 포함하도록 선택할 수 있습니다.

두 의도의 어휘가 동일한 경우, 의도를 결합하고 엔터티를 사용합니다.

다음 예제 발화를 고려해 보세요.

|예제 발화|
|--|
|항공권 예약|
|호텔 예약|

`Book a flight` 및는 `Book a hotel` 의 동일한 어휘를 사용 `book a ` 합니다. 이 형식은 동일 하므로의 다른 단어와 추출 된 엔터티로 동일한 의도 여야 합니다 `flight` `hotel` .

## <a name="do-add-features-to-intents"></a>의도에 기능 추가

기능은 의도에 대 한 개념을 설명 합니다. 기능은 해당 의도 나 해당 의도에 중요 한 엔터티에 중요 한 단어의 문구 목록이 될 수 있습니다.

## <a name="do-find-sweet-spot-for-intents"></a>의도의 안정적인 지점 찾기
LUIS의 예측 데이터를 사용하여 의도가 겹치는지 확인합니다. 겹치는 의도는 LUIS에 혼동을 유발합니다. 결과는 상위 점수 의도가 다른 의도에 너무 가깝다는 것입니다. LUIS는 매번 학습할 데이터를 통해 똑같은 경로를 사용하지 않으므로 겹치는 의도는 학습에서 첫 번째 또는 두 번째가 될 수 있습니다. 이러한 플립플롭이 발생하지 않도록 각 의도의 발언 점수가 멀리 떨어지는 좋습니다. 의도를 분명히 구별하면 매번 상위 의도가 예측됩니다.

## <a name="do-use-machine-learned-entities"></a>컴퓨터에서 배운 엔터티 사용

컴퓨터에서 배운 엔터티는 앱에 맞게 조정 되며 성공적으로 레이블을 지정 해야 합니다. 컴퓨터에서 배운 엔터티를 사용 하지 않는 경우 잘못 된 도구를 사용할 수 있습니다.

컴퓨터에서 배운 엔터티는 다른 엔터티를 기능으로 사용할 수 있습니다. 이러한 다른 엔터티는 정규식 엔터티 또는 목록 엔터티와 같은 사용자 지정 엔터티가 될 수도 있고 미리 작성 된 엔터티를 기능으로 사용할 수도 있습니다.

[효과적인 기계 학습 엔터티에](luis-concept-entity-types.md#effective-machine-learned-entities)대해 알아보세요.

<a name="#do-build-the-app-iteratively"></a>

## <a name="do-build-your-app-iteratively-with-versions"></a>버전을 사용 하 여 반복적으로 앱 빌드

각각의 작성 주기는 기존 버전에서 복제된 새 [버전](./luis-concept-app-iteration.md) 내에 있어야 합니다.

## <a name="do-build-for-model-decomposition"></a>모델 분해를 위한 빌드 수행

모델 분해의 일반적인 프로세스는 다음과 같습니다.

* 클라이언트 앱의 사용자 의도에 따라 **의도** 만들기
* 실제 사용자 입력을 기반으로 15-30 예제 길이 발언 추가
* 예 utterance에서 최상위 데이터 개념 레이블
* 하위 엔터티로 데이터 개념 분할
* 하위 엔터티에 기능 추가
* 의도에 기능 추가

의도를 만들고 예제 길이 발언를 추가한 후에는 다음 예제에서 엔터티 분해에 대해 설명 합니다.

먼저 utterance에서 추출 하려는 전체 데이터 개념을 파악 합니다. 이것은 기계 학습 엔터티입니다. 그런 다음 구를 해당 부분으로 분해 합니다. 여기에는 하위 엔터티 및 기능을 식별 하는 작업이 포함 됩니다.

예를 들어 주소를 추출 하려는 경우 위쪽 기계 학습 엔터티를 호출할 수 있습니다 `Address` . 주소를 만드는 동안 주소, 구/군/시, 시/도 및 우편 번호와 같은 하위 엔터티 중 일부를 식별 합니다.

다음 방법으로 이러한 요소를 계속 분해.
* 일반 식 엔터티로 우편 번호의 필수 기능을 추가 합니다.
* 분해 주소를 다음과 같이 구성 합니다.
    * 미리 작성 된 number 엔터티의 필수 기능을 포함 하는 **번 지 수** 입니다.
    * **주소의 이름** 입니다.
    * 통로, 원,도로, 레인 등의 단어를 포함 하는 목록 엔터티의 필수 기능을 포함 하는 **번 지 형식** 입니다.

V3 authoring API는 모델 분해를 허용 합니다.

## <a name="do-add-patterns-in-later-iterations"></a>이후 반복에서 패턴 추가

패턴 [은 패턴을 추가 하기](luis-concept-patterns.md) 전에 패턴을 추가 하는 방법을 이해 해야 합니다. 패턴은 예 길이 발언 보다 훨씬 더 많은 비중을 가지 며 확신을 기울입니다.

앱이 작동 하는 방식을 이해 했으면 앱에 적용 되는 패턴을 추가 합니다. 각 [반복](luis-concept-app-iteration.md)에 추가 하지 않아도 됩니다.

모델 디자인의 시작 부분에는 아무 문제가 없지만 모델을 길이 발언로 테스트 한 후에는 각 패턴이 모델을 어떻게 변경 하는지 쉽게 확인할 수 있습니다.

<a name="balance-your-utterances-across-all-intents"></a>

## <a name="do-balance-your-utterances-across-all-intents"></a>모든 의도에서 길이 발언의 균형을 유지 합니다.

LUIS 예측의 정확도를 높이려면 각 의도(None 의도 제외)의 예제 발화 양이 상대적으로 동등해야 합니다.

예제 발언이 100개인 의도와 예제 발언이 20개인 의도가 있으면 발언이 100개인 의도의 예측 비율이 더 높아집니다.

## <a name="do-add-example-utterances-to-none-intent"></a>None 의도에 예제 발화 추가

이러한 의도는 응용 프로그램 외부의 모든 항목을 나타내는 대체 의도입니다. LUIS 앱의 나머지 부분에 있는 예제 발화 10개마다 하나의 예제 발화를 None 의도에 추가합니다.

## <a name="do-leverage-the-suggest-feature-for-active-learning"></a>활성 학습의 제안 기능 활용

의도에 더 많은 예제 의도를 추가하는 대신 정기적으로 [활성 학습](luis-how-to-review-endpoint-utterances.md)의 **엔드포인트 발화 검토** 를 사용합니다. 앱이 지속적으로 엔드포인트 발화를 수신하기 때문에 이 목록은 계속 증가하고 변경됩니다.

## <a name="do-monitor-the-performance-of-your-app"></a>앱 성능 모니터링

[일괄 테스트](luis-concept-batch-test.md) 세트를 사용하여 예측 정확도를 모니터링합니다.

길이 발언 또는 endpoint 길이 발언 [예제](luis-concept-utterance.md) 로 사용 되지 않는 별도의 길이 발언 집합을 유지 합니다. 테스트 집합의 앱을 계속 개선합니다. 실제 사용자 발화를 반영하도록 테스트 집합을 조정합니다. 이 테스트 세트를 사용하여 앱의 반복 또는 버전을 평가합니다.

## <a name="dont-publish-too-quickly"></a>너무 빨리 게시 하지 않음

[적절 한 계획](#do-plan-your-schema)없이 앱을 너무 빨리 게시 하면 다음과 같은 몇 가지 문제가 발생할 수 있습니다.

* 앱은 적절 한 수준의 성능으로 실제 시나리오에서 작동 하지 않습니다.
* 스키마 (의도 및 엔터티)가 적절 하지 않으며 스키마를 따라 클라이언트 앱 논리를 개발한 경우 처음부터 다시 작성 해야 할 수 있습니다. 이로 인해 작업 중인 프로젝트에 대 한 예기치 않은 지연 및 추가 비용이 발생할 수 있습니다.
* 모델에 추가 하면 디버그 하 고 식별 하기 어려운 예제 utterance 집합에 길이 발언 수 있습니다. 또한 특정 스키마를 커밋한 후 모호성을 제거 합니다.

## <a name="dont-add-many-example-utterances-to-intents"></a>의도에 많은 예제 발화 추가 안 함

앱이 게시 되 면 개발 수명 주기 프로세스에서 활성 학습의 길이 발언 추가 합니다. 발화가 너무 비슷한 경우, 패턴을 추가합니다.

## <a name="dont-use-few-or-simple-entities"></a>소수의 엔터티 또는 단순 엔터티 사용 안 함

엔터티는 데이터 추출 및 예측을 위해 빌드됩니다. 각 의도에는 의도 한 데이터를 설명 하는 기계 학습 엔터티가 있어야 합니다. 이렇게 하면 클라이언트 응용 프로그램에서 추출 된 엔터티를 사용할 필요가 없는 경우에도 LUIS을 예측 하는 데 도움이 됩니다.

## <a name="dont-use-luis-as-a-training-platform"></a>LUIS를 학습 플랫폼으로 사용 안 함

LUIS는 언어 모델의 도메인에 관련됩니다. 일반 자연어 학습 플랫폼으로 작동하지 않습니다.

## <a name="dont-add-many-example-utterances-of-the-same-format-ignoring-other-formats"></a>동일한 형식의 많은 예제 발화를 추가하여 다른 형식 무시 안 함

LUIS는 의도의 발화에서 변형을 예측합니다. 전체 의미는 동일하지만 발화가 달라질 수 있습니다. 변형에는 발화 길이, 단어 선택 및 단어 배치가 포함될 수 있습니다.

|같은 형식 사용 안 함|다양한 형식 사용|
|--|--|
|시애틀행 항공권 구입<br>파리행 항공권 구입<br>올랜도행 항공권 구입|시애틀행 항공권 1매 구입<br>다음 월요일에 파리행 야간 항공편에서 2석 예약<br>봄 방학을 위해 올란도행 항공권 3매를 예약|

두 번째 열에는 여러 가지 동사(구입, 예약), 여러 가지 수량(1, 2, 3), 여러 가지 단어 배열이 사용되지만 모두 여행을 위해 항공사 티켓을 구입하려는 동의할 의도를 포함합니다.

## <a name="dont-mix-the-definition-of-intents-and-entities"></a>의도 및 엔터티 정의 혼합 안 함

봇이 수행할 작업의 의도를 만듭니다. 작업을 가능하게 하는 매개 변수로 엔터티를 사용합니다.

항공편을 위한 봇의 경우 **Bookflight** 의도를 만듭니다. 모든 항공사 또는 모든 목적지의 의도는 만들지 마세요. 이러한 데이터 조각을 [엔터티](luis-concept-entity-types.md)로 사용하고 예제 발화에서 이러한 데이터를 표시합니다.

## <a name="dont-create-phrase-lists-with-all-the-possible-values"></a>모든 가능한 값을 사용하여 구문 목록 만들지 않음

[구 목록](luis-concept-feature.md) 에 몇 가지 예를 제공 하지만 일부 단어 또는 문구는 제공 하지 않습니다. LUIS는 컨텍스트를 일반화하고 고려합니다.

## <a name="dont-add-many-patterns"></a>너무 많은 패턴 추가 안 함

[패턴](luis-concept-patterns.md)을 너무 많이 추가하지 마세요. LUIS는 더 적은 예제를 사용하여 빠르게 학습합니다. 시스템을 불필요하게 오버로드하지 마세요.

## <a name="dont-train-and-publish-with-every-single-example-utterance"></a>모든 단일 예제를 사용하여 학습 및 게시 안 함

학습 및 게시 전에 10개 또는 15개의 발화를 추가합니다. 이렇게 하면 예측 정확도에 미치는 영향을 확인할 수 있습니다. 단일 발화를 추가하면 점수에 분명한 영향을 주지 않습니다.

## <a name="next-steps"></a>다음 단계

* LUIS 앱에서 [앱을 계획](luis-how-plan-your-app.md)하는 방법을 알아봅니다.