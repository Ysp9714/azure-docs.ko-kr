---
title: Facebook 계정으로 등록 및 로그인 설정
titleSuffix: Azure AD B2C
description: 고객에게 Azure Active Directory B2C를 사용하여 애플리케이션에서 Facebook 계정으로 등록 및 로그인을 제공합니다.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/26/2019
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 62956000e143f5504d32dae26953bcf877ce96a6
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92628578"
---
# <a name="set-up-sign-up-and-sign-in-with-a-facebook-account-using-azure-active-directory-b2c"></a>Azure Active Directory B2C를 사용하여 Facebook 계정으로 등록 설정 및 로그인

## <a name="create-a-facebook-application"></a>Facebook 애플리케이션 만들기

Azure Active Directory B2C (Azure AD B2C)에서 Facebook 계정을 [id 공급자로](authorization-code-flow.md) 사용 하려면 테 넌 트에서 응용 프로그램을 나타내는 응용 프로그램을 만들어야 합니다. Facebook 계정이 없는 경우 [https://www.facebook.com/](https://www.facebook.com/)에서 등록할 수 있습니다.

1. Facebook 계정 자격 증명으로 [개발자용 Facebook](https://developers.facebook.com/)에 로그인합니다.
1. 아직 등록하지 않은 경우 Facebook 개발자로 등록해야 합니다. 이를 수행하려면 페이지의 오른쪽 위에서 **시작** 을 선택하고 Facebook의 정책에 동의한 후 등록 단계를 완료합니다.
1. **내 앱** 을 선택한 다음, **앱 만들기** 를 선택합니다.
1. **연결 된 환경 빌드** 를 선택 합니다.
1. **표시 이름** 및 유효한 **연락처 전자 메일** 을 제공합니다.
1. **앱 ID 만들기** 를 선택합니다. Facebook 플랫폼 정책을 수용하고 온라인 보안 검사를 완료해야 합니다.
1. **설정** > **기본** 을 선택합니다.
    1. `Business and Pages` 등의 **범주** 를 선택합니다. 이 값은 Facebook의 경우 필수이지만 Azure AD B2C에서는 사용되지 않습니다.
    1. **서비스 약관 url** (예:)에 대 한 url을 입력 `http://www.contoso.com/tos` 합니다. 정책 URL은 응용 프로그램에 대 한 사용 약관을 제공 하기 위해 유지 관리 하는 페이지입니다.
    1. **개인 정보 취급 방침 URL** 의 URL(예: `http://www.contoso.com/privacy`)을 입력합니다. 정책 URL은 애플리케이션에 대한 개인 정보를 제공하기 위해 유지 관리하는 페이지입니다.
1. 페이지의 맨 아래에서 **플랫폼 추가** 를 선택한 후 **웹 사이트** 를 선택합니다.
1. **사이트 URL** 에 웹 사이트의 주소를 입력 합니다 (예:) `https://contoso.com` . 
1. **변경 내용 저장** 을 선택합니다.
1. 페이지의 맨 위에서 **앱 ID** 값을 복사합니다.
1. **표시** 를 선택하고 **앱 비밀** 값을 복사합니다. 테넌트에서 Facebook을 ID 공급자로 구성하려면 둘 다 사용합니다. **앱 암호** 는 중요한 보안 자격 증명입니다.
1. 메뉴에서 **제품** 옆에 있는 **더하기** 기호를 선택 합니다. **앱에 제품 추가** 아래의 **Facebook 로그인** 아래에서 **설정** 을 선택 합니다.
1. 메뉴에서 **Facebook 로그인** 을 선택 하 고 **설정** 을 선택 합니다.
1. **유효한 OAuth 리디렉션 URI** 에 `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp`를 입력합니다. `your-tenant-name`을 테넌트 이름으로 바꿉니다. 페이지 아래쪽에 있는 **변경 내용 저장** 을 선택합니다.
1. Facebook 응용 프로그램을 Azure AD B2C 사용할 수 있도록 하려면 페이지의 오른쪽 위에 있는 상태 선택기를 선택 하 고 응용 **프로그램을 공개로 설정한** 다음 **스위치 모드** 를 선택 합니다.  이때 상태가 **개발** 에서 **라이브** 로 변경됩니다.

## <a name="configure-a-facebook-account-as-an-identity-provider"></a>Facebook 계정을 ID 공급자로 구성

1. Azure AD B2C 테넌트의 전역 관리자로 [Azure Portal](https://portal.azure.com/)에 로그인합니다.
1. Azure AD B2C 테넌트를 포함하는 디렉터리를 사용하려면 위쪽 메뉴에서 **디렉터리 + 구독** 필터를 선택하고, 테넌트가 포함된 디렉터리를 선택합니다.
1. Azure Portal의 왼쪽 상단 모서리에서 **모든 서비스** 를 선택하고 **Azure AD B2C** 를 검색하여 선택합니다.
1. **ID 공급자** 를 선택한 다음, **Facebook** 을 선택합니다.
1. **이름** 을 입력합니다. 예를 들어 *Facebook* 입니다.
1. **클라이언트 ID** 에 대해 이전에 만든 Facebook 애플리케이션의 앱 ID를 입력합니다.
1. **클라이언트 암호** 에는 기록했던 앱 비밀을 입력합니다.
1. **저장** 을 선택합니다.
