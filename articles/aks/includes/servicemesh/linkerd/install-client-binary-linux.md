---
author: paulbouwer
ms.topic: include
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: d303def85d66d5c2039b426e48186f90c2b3fff6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86277816"
---
## <a name="download-and-install-the-linkerd-linkerd-client-binary"></a>Linkerd Linkerd 클라이언트 이진 파일을 다운로드 하 여 설치 합니다.

Linux 또는 linux [용 Windows 하위 시스템][install-wsl]의 bash 기반 셸에서 다음과 `curl` 같이를 사용 하 여 linkerd 릴리스를 다운로드 합니다.

```bash
# Specify the Linkerd version that will be leveraged throughout these instructions
LINKERD_VERSION=stable-2.6.0

curl -sLO "https://github.com/linkerd/linkerd2/releases/download/$LINKERD_VERSION/linkerd2-cli-$LINKERD_VERSION-linux"
```

클라이언트 `linkerd` 이진 파일은 클라이언트 컴퓨터에서 실행 되며 Linkerd 서비스 메시와 상호 작용할 수 있습니다. 다음 명령을 사용 하 여 linux의 `linkerd` bash 기반 셸 또는 [Linux 용 Windows 하위 시스템][install-wsl]에 linkerd 클라이언트 이진을 설치 합니다. 이 명령을 사용하여 `linkerd` 클라이언트 이진 파일을 `PATH`의 표준 사용자 프로그램 위치에 복사합니다.

```bash
sudo cp ./linkerd2-cli-$LINKERD_VERSION-linux /usr/local/bin/linkerd
sudo chmod +x /usr/local/bin/linkerd
```

Linkerd 클라이언트 이진에 대 한 명령줄 완성을 원하는 경우 다음과 `linkerd` 같이 설정 합니다.

```bash
# Generate the bash completion file and source it in your current shell
mkdir -p ~/completions && linkerd completion bash > ~/completions/linkerd.bash
source ~/completions/linkerd.bash

# Source the bash completion file in your .bashrc so that the command-line completions
# are permanently available in your shell
echo "source ~/completions/linkerd.bash" >> ~/.bashrc
```

<!-- LINKS - external -->
[install-wsl]: /windows/wsl/install-win10
