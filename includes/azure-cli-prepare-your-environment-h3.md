---
ms.topic: include
ms.date: 09/10/2020
author: dbradish-microsoft
ms.author: dbradish
manager: barbkess
ms.technology: azure-cli
ms.service: azure-cli
ms.devlang: azurecli
ms.custom: devx-track-azurecli
ms.openlocfilehash: 01e1f956270ebbc9b9d5d0ebdfdff52875cafedf
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95558328"
---
### <a name="prepare-your-environment-for-the-azure-cli"></a>Azure CLI에 대한 환경 준비

- Bash 환경을 통해 [Azure Cloud Shell](../articles/cloud-shell/quickstart.md)을 사용합니다.

   [![Embed 시작](https://shell.azure.com/images/launchcloudshell.png "Azure Cloud Shell 시작")](https://shell.azure.com)   
- 원하는 경우 Azure CLI를 [설치](/cli/azure/install-azure-cli)하여 CLI 참조 명령을 실행합니다.
   - local install을 사용하는 경우 [az login](/cli/azure/reference-index#az-login) 명령을 사용하여 Azure CLI에 로그인합니다.  인증 프로세스를 완료하려면 터미널에 표시되는 단계를 수행합니다.  추가 로그인 옵션은 [Azure CLI를 사용하여 로그인](/cli/azure/authenticate-azure-cli)을 참조하세요.
  - 메시지가 표시되면 처음 사용할 때 Azure CLI 확장을 설치합니다.  확장에 대한 자세한 내용은 [Azure CLI에서 확장 사용](/cli/azure/azure-cli-extensions-overview)을 참조하세요.
  - [az version](/cli/azure/reference-index?#az_version)을 실행하여 설치된 버전과 종속 라이브러리를 찾습니다. 최신 버전으로 업그레이드하려면 [az upgrade](/cli/azure/reference-index?#az_upgrade)를 실행합니다.