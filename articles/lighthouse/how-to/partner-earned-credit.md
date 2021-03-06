---
title: 파트너 ID를 연결 하 여 위임 된 리소스에 대 한 영향 추적
description: Azure Lighthouse를 통해 관리 하는 고객 리소스에서 파트너 ID를 연결 하 여 PEC (파트너 획득 크레딧)를 받는 방법에 대해 알아봅니다.
ms.date: 10/30/2020
ms.topic: how-to
ms.openlocfilehash: fcbcc70e380116b8e9f9b1c1e365dee1adb87a99
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080280"
---
# <a name="link-your-partner-id-to-track-your-impact-on-delegated-resources"></a>파트너 ID를 연결 하 여 위임 된 리소스에 대 한 영향 추적 

[Microsoft 파트너 네트워크](https://partner.microsoft.com/)구성원 인 경우 위임 된 고객 리소스를 관리 하는 데 사용 되는 자격 증명과 파트너 ID를 연결할 수 있습니다. PAL (파트너 관리자 링크)을 통해 Microsoft는 Azure 고객의 성공을 유도 하는 파트너를 식별 하 고 인식할 수 있습니다. 이 링크를 통해 [CSP (클라우드 솔루션 공급자)](/partner-center/csp-overview) 파트너는 [Microsoft 고객 계약 (MCA)에 서명](/partner-center/confirm-customer-agreement) 하 고 [Azure 요금제](/partner-center/azure-plan-get-started)에 있는 고객에 대해 [PEC (관리 서비스)의 파트너 획득 크레딧을](/partner-center/partner-earned-credit) 받을 수 있습니다.

[Azure Marketplace에서 관리 서비스 제품](publish-managed-services-offers.md)을 사용 하 여 고객을 등록 하는 경우 제품을 게시 하는 데 사용 된 파트너 센터 계정과 연결 된 MPN ID를 사용 하 여 연결이 자동으로 수행 됩니다. 이러한 고객에 대 한 영향을 추적 하기 위해 추가 작업이 필요 하지 않습니다.

[Azure 리소스 관리 템플릿을 사용 하 여 고객](onboard-customer.md)을 등록 하는 경우이 링크를 만드는 작업을 수행 해야 합니다. 이러한 작업은 각 등록 구독에 대 한 액세스 권한이 있는 관리 테 넌 트의 하나 이상의 사용자 계정에 [MPN ID를 연결](../../cost-management-billing/manage/link-partner-id.md) 하 여 수행 됩니다.

## <a name="associate-your-partner-id-when-you-onboard-new-customers"></a>새 고객을 등록할 때 파트너 ID 연결

Azure Resource Manager 템플릿 (ARM 템플릿)을 통해 고객을 온 보 딩 하는 경우 다음 프로세스를 사용 하 여 파트너 ID를 연결 하 고 파트너 획득 크레딧을 사용 하도록 설정 합니다 (해당 하는 경우). 이러한 단계를 완료 하려면 [MPN 파트너 ID](/partner-center/partner-center-account-setup#locate-your-mpn-id) 를 알고 있어야 합니다. 파트너 프로필에 표시된 **관련 MPN ID** 를 사용해야 합니다.

간단히 하기 위해 테 넌 트에 서비스 주체 계정을 만들고 **연결 된 MPN ID** 에 연결한 다음, [PEC에 적격 한 Azure 기본 제공 역할](/partner-center/azure-roles-perms-pec)을 사용 하 여 등록 한 모든 고객에 게 액세스 권한을 부여 하는 것이 좋습니다.

1. 관리 테 넌 트에서 [서비스 사용자 계정을 만듭니다](../../active-directory/develop/howto-authenticate-service-principal-powershell.md) . 이 예에서는이 서비스 사용자에 대해 이름 *공급자 Automation 계정을* 사용 합니다.
1. 해당 서비스 주체 계정을 사용 하 여 관리 하는 테 넌 트의 [연결 된 MPN ID에 연결](../../cost-management-billing/manage/link-partner-id.md#link-to-a-partner-id) 합니다. 이 작업은 한 번만 수행 하면 됩니다.
1. [ARM 템플릿을 사용 하 여 고객을](onboard-customer.md)등록 하는 경우 [Azure 기본 제공 역할](/partner-center/azure-roles-perms-pec)을 사용 하는 사용자로 서, PEC에 적격 한 권한 부여를 포함 해야 합니다.

이러한 단계를 수행 하 여 관리 하는 모든 고객 테 넌 트가 파트너 ID와 연결 됩니다. 공급자 Automation 계정은 고객 테 넌 트에서 인증 하거나 작업을 수행할 필요가 없습니다.

:::image type="content" source="../media/lighthouse-pal.jpg" alt-text="Azure Lighthouse를 사용 하 여 PAL 프로세스를 보여 주는 다이어그램입니다.":::

## <a name="add-your-partner-id-to-previously-onboarded-customers"></a>이전에 등록 고객에 게 파트너 ID 추가

이미 고객을 등록 하는 경우 다른 배포를 수행 하 여 공급자 Automation 계정 서비스 주체를 추가 하지 않으려고 할 수 있습니다. 대신, 해당 고객 테 넌 트의 작업에 대 한 액세스 권한이 이미 있는 사용자 계정과 **연결 된 MPN ID** 를 연결할 수 있습니다. 계정에는 [PEC에 적합 한 Azure 기본 제공 역할이](/partner-center/azure-roles-perms-pec)부여 되어 있어야 합니다.

계정이 관리 되는 테 넌 트의 [연결 된 MPN ID에 연결](../../cost-management-billing/manage/link-partner-id.md#link-to-a-partner-id) 되 면 해당 고객에 대 한 영향에 대 한 인식을 추적할 수 있습니다.

## <a name="confirm-partner-earned-credit"></a>파트너의 획득 크레딧을 확인 합니다.

[Azure Portal에서 pec 세부 정보를 보고](/partner-center/partner-earned-credit-explanation#azure-cost-management) , pec의 혜택을 받은 비용을 확인할 수 있습니다. PEC는 MCA에 서명 하 고 Azure 요금제에 있는 CSP 고객 에게만 적용 됩니다.

위의 단계를 수행한 후 예상 되는 연결이 표시 되지 않으면 Azure Portal에서 지원 요청을 엽니다.

[파트너 센터 SDK](/partner-center/develop/get-invoice-unbilled-consumption-lineitems) 를 사용 하 고에 필터를 설정 `rateOfPartnerEarnedCredit` 하 여 구독에 대 한 PEC 유효성 검사를 자동화할 수도 있습니다.

## <a name="next-steps"></a>다음 단계

- [Microsoft 파트너 네트워크](/partner-center/mpn-overview)에 대해 자세히 알아보세요.
- [PEC를 계산 하 고 지불 하는 방법을](/partner-center/partner-earned-credit-explanation)알아봅니다.
- Azure Lighthouse에 [고객](onboard-customer.md) 을 등록 합니다.