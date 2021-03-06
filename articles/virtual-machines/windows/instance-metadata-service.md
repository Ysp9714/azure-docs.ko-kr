---
title: Azure Instance Metadata Service
description: RESTful 인터페이스를 통해 Vm 계산, 네트워크 및 예정 된 유지 관리 이벤트에 대 한 정보를 가져올 수 있습니다.
services: virtual-machines
author: KumariSupriya
manager: paulmey
ms.service: virtual-machines
ms.subservice: monitoring
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 03/30/2020
ms.author: sukumari
ms.reviewer: azmetadatadev
ms.openlocfilehash: 1de04a9486103af65deb6ca3b653b9efa6707716
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967077"
---
# <a name="azure-instance-metadata-service"></a>Azure Instance Metadata Service

Azure IMDS(Instance Metadata Service)는 가상 머신 인스턴스를 실행하는 방법에 대한 정보를 제공하며 가상 머신을 관리 및 구성하는 데 사용할 수 있습니다.
이 정보에는 SKU, 저장소, 네트워크 구성 및 예정 된 유지 관리 이벤트가 포함 됩니다. 사용할 수 있는 데이터의 전체 목록은 [메타데이터 API](#metadata-apis)를 참조하세요.
Instance Metadata Service는 가상 머신 및 가상 머신 확장 집합 인스턴스를 실행 하는 데 사용할 수 있습니다. 모든 Api는 [Azure Resource Manager](/rest/api/resources/)을 사용 하 여 생성/관리 되는 vm을 지원 합니다. 증명 된 및 네트워크 끝점 에서만 클래식 (ARM) Vm이 지원 되며, 증명 된는 제한 된 범위에만 가능 합니다.

Azure의 IMDS는 잘 알려진 라우팅할 수 없는 IP 주소 ()에서 사용할 수 있는 REST 끝점으로 `169.254.169.254` , VM 내 에서만 액세스할 수 있습니다. VM과 IMDS 간의 통신은 호스트를 유지 하지 않습니다.
HTTP 클라이언트는 IMDS를 쿼리할 때 VM 내에서 웹 프록시를 우회 하 고와 동일 하 게 처리 하는 것이 좋습니다 `169.254.169.254` [`168.63.129.16`](../../virtual-network/what-is-ip-address-168-63-129-16.md) .

## <a name="security"></a>보안

Instance Metadata Service 엔드포인트는 라우팅이 불가능한 IP 주소에서 실행 중인 가상 머신 인스턴스 내부에서만 액세스할 수 있습니다. 또한 `X-Forwarded-For` 헤더가 포함된 모든 요청은 서비스에 의해 거부됩니다.
실제 요청이 의도치 않은 리디렉션의 일환이 아니라 직접적으로 의도된 것이라는 것을 확인하기 위해 요청에 `Metadata: true` 헤더가 포함되어야 합니다.

> [!IMPORTANT]
> Instance Metadata Service는 중요한 데이터에 대한 채널이 아닙니다. 이 엔드포인트는 VM의 모든 프로세스에 대해 열려 있습니다. 이 서비스를 통해 노출되는 정보는 VM 내에서 실행 중인 모든 애플리케이션에 대한 공유 정보로 간주되어야 합니다.

## <a name="usage"></a>사용

### <a name="accessing-azure-instance-metadata-service"></a>Azure Instance Metadata Service 액세스

Instance Metadata Service에 액세스하려면 [Azure Resource Manager](/rest/api/resources/) 또는 [Azure Portal](https://portal.azure.com)에서 VM을 만들고 아래 예제를 따릅니다.
IMDS를 쿼리하는 방법에 대한 추가 예제는 [Azure 인스턴스 메타 데이터 샘플](https://github.com/microsoft/azureimds)에서 찾을 수 있습니다.

다음은 인스턴스에 대한 모든 메타데이터를 검색하는 샘플 코드입니다. 특정 데이터 소스에 액세스하려면 [메타데이터 API](#metadata-apis) 섹션을 참조하세요. 

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance?api-version=2020-09-01 | ConvertTo-Json
```
> [!NOTE]
> `-NoProxy`플래그는 PowerShell 6 이상 에서만 사용할 수 있습니다. 프록시 설정이 없는 경우 플래그를 생략할 수 있습니다.

**응답**

> [!NOTE]
> 응답은 JSON 문자열입니다. Microsoft는이 cmdlet을 통해 `ConvertTo-Json` 예쁜 인쇄를 위해 REST 쿼리를 파이프 합니다.

```json
{
    "compute": {
        "azEnvironment": "AZUREPUBLICCLOUD",
        "isHostCompatibilityLayerVm": "true",
        "licenseType":  "Windows_Client",
        "location": "westus",
        "name": "examplevmname",
        "offer": "Windows",
        "osProfile": {
            "adminUsername": "admin",
            "computerName": "examplevmname",
            "disablePasswordAuthentication": "true"
        },
        "osType": "linux",
        "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
        "plan": {
            "name": "planName",
            "product": "planProduct",
            "publisher": "planPublisher"
        },
        "platformFaultDomain": "36",
        "platformUpdateDomain": "42",
        "publicKeys": [{
                "keyData": "ssh-rsa 0",
                "path": "/home/user/.ssh/authorized_keys0"
            },
            {
                "keyData": "ssh-rsa 1",
                "path": "/home/user/.ssh/authorized_keys1"
            }
        ],
        "publisher": "RDFE-Test-Microsoft-Windows-Server-Group",
        "resourceGroupName": "macikgo-test-may-23",
        "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
        "securityProfile": {
            "secureBootEnabled": "true",
            "virtualTpmEnabled": "false"
        },
        "sku": "Windows-Server-2012-R2-Datacenter",
        "storageProfile": {
            "dataDisks": [{
                "caching": "None",
                "createOption": "Empty",
                "diskSizeGB": "1024",
                "image": {
                    "uri": ""
                },
                "lun": "0",
                "managedDisk": {
                    "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                    "storageAccountType": "Standard_LRS"
                },
                "name": "exampledatadiskname",
                "vhd": {
                    "uri": ""
                },
                "writeAcceleratorEnabled": "false"
            }],
            "imageReference": {
                "id": "",
                "offer": "UbuntuServer",
                "publisher": "Canonical",
                "sku": "16.04.0-LTS",
                "version": "latest"
            },
            "osDisk": {
                "caching": "ReadWrite",
                "createOption": "FromImage",
                "diskSizeGB": "30",
                "diffDiskSettings": {
                    "option": "Local"
                },
                "encryptionSettings": {
                    "enabled": "false"
                },
                "image": {
                    "uri": ""
                },
                "managedDisk": {
                    "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                    "storageAccountType": "Standard_LRS"
                },
                "name": "exampleosdiskname",
                "osType": "Linux",
                "vhd": {
                    "uri": ""
                },
                "writeAcceleratorEnabled": "false"
            }
        },
        "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
        "tags": "baz:bash;foo:bar",
        "version": "15.05.22",
        "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
        "vmScaleSetName": "crpteste9vflji9",
        "vmSize": "Standard_A3",
        "zone": ""
    },
    "network": {
        "interface": [{
            "ipv4": {
               "ipAddress": [{
                    "privateIpAddress": "10.144.133.132",
                    "publicIpAddress": ""
                }],
                "subnet": [{
                    "address": "10.144.133.128",
                    "prefix": "26"
                }]
            },
            "ipv6": {
                "ipAddress": [
                 ]
            },
            "macAddress": "0011AAFFBB22"
        }]
    }
}
```

### <a name="data-output"></a>데이터 출력


기본적으로 Instance Metadata Service는 JSON 형식(`Content-Type: application/json`)의 데이터를 반환합니다. 그러나 일부 Api는 요청 된 경우 다른 형식으로 데이터를 반환할 수 있습니다.
다음 표는 API에서 지원할 수 있는 다른 데이터 형식에 대한 참조입니다.

API | 기본 데이터 형식 | 다른 형식
--------|---------------------|--------------
/attested | json : | none
/identity | json : | none
/instance | json : | text
/scheduledevents | json : | none

기본이 아닌 응답 형식에 액세스하려면 요청된 형식을 요청의 쿼리 문자열 매개 변수로 지정합니다. 다음은 그 예입니다.

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance?api-version=2017-08-01&format=text"
```

> [!NOTE]
> /Metadata/instance의 리프 노드에 대 한는 `format=json` 작동 하지 않습니다. `format=text`기본 형식이 json 이므로 이러한 쿼리를 명시적으로 지정 해야 합니다.

### <a name="versioning"></a>버전 관리

Instance Metadata Service는 버전이 관리되며 HTTP 요청에 API 버전을 필수로 지정해야 합니다.

지원 되는 API 버전은 다음과 같습니다. 
- 2017-03-01
- 2017-04-02
- 2017-08-01 
- 2017-10-01
- 2017-12-01 
- 2018-02-01
- 2018-04-02
- 2018-10-01
- 2019-02-01
- 2019-03-11
- 2019-04-30
- 2019-06-01
- 2019-06-04
- 2019-08-01
- 2019-08-15
- 2019-11-01
- 2020-06-01
- 2020-07-15
- 2020-09-01
- 2020-10-01

> [!NOTE]
> 버전 2020-10-01은 현재 롤아웃 중 이며, 일부 지역에서는 아직 사용할 수 없습니다.

새로운 버전이 추가되더라도 스크립트가 특정 데이터 형식에 종속성이 있는 경우 이전 버전에 여전히 액세스할 수 있습니다.

버전을 지정하지 않으면 지원되는 최신 버전의 목록과 함께 오류가 반환됩니다.

> [!NOTE]
> 응답은 JSON 문자열입니다. 다음 예는 버전이 지정되지 않은 경우의 오류 조건을 나타내며 가독성을 위해 응답이 프리티 프린트(pretty-print) 처리되었습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance
```

**응답**

```json
{
    "error": "Bad request. api-version was not specified in the request. For more information refer to aka.ms/azureimds",
    "newest-versions": [
        "2020-10-01",
        "2020-09-01",
        "2020-07-15"
    ]
}
```

## <a name="metadata-apis"></a>메타데이터 API

Metadata Service에는 서로 다른 데이터 원본을 나타내는 여러 Api가 포함 되어 있습니다.

API | Description | 도입된 버전
----|-------------|-----------------------
/attested | [증명된 데이터](#attested-data) 참조 | 2018-10-01
/identity | [액세스 토큰 획득](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) 참조 | 2018-02-01
/instance | [인스턴스 API](#instance-api) 참조 | 2017-04-02
/scheduledevents | [예정된 이벤트](scheduled-events.md) 참조 | 2017-08-01

## <a name="instance-api"></a>인스턴스 API

인스턴스 API는 VM, 네트워크 및 저장소를 포함 하 여 VM 인스턴스에 대 한 중요 한 메타 데이터를 제공 합니다. 인스턴스/컴퓨팅을 통해 다음 범주에 액세스할 수 있습니다.

데이터 | Description | 도입된 버전
-----|-------------|-----------------------
azEnvironment | VM이 실행되는 Azure 환경 | 2018-10-01
customData | 이 기능은 현재 사용할 수 없습니다. 이 설명서를 사용할 수 있게 되 면 업데이트 합니다. | 2019-02-01
isHostCompatibilityLayerVm | VM이 호스트 호환성 계층에서 실행 되는지 확인 합니다. | 2020-06-01
licenseType | [Azure 하이브리드 혜택](https://azure.microsoft.com/pricing/hybrid-benefit)라이선스의 유형입니다. 이는 AHB 사용 Vm에 대해서만 존재 합니다. | 2020-09-01
위치 | VM을 실행하는 Azure 지역 | 2017-04-02
name | VM의 이름 | 2017-04-02
제품 | VM 이미지에 대한 제품 정보이며 Azure 이미지 갤러리에서 배포된 이미지에만 있습니다. | 2017-04-02
osProfile.adminUsername | 관리자 계정의 이름을 지정 합니다. | 2020-07-15
osProfile | 컴퓨터의 이름을 지정 합니다. | 2020-07-15
osProfile. disablePasswordAuthentication | 암호 인증을 사용할 수 없는지 여부를 지정합니다. Linux Vm에만 존재 합니다. | 2020-10-01
osType | Linux 또는or Windows | 2017-04-02
placementGroupId | 가상 머신 확장 집합의 [배치 그룹](../../virtual-machine-scale-sets/virtual-machine-scale-sets-placement-groups.md) | 2017-08-01
계획 | Azure Marketplace 이미지에 해당하는 VM의 이름, 제품 및 게시자를 포함하는 [계획](/rest/api/compute/virtualmachines/createorupdate#plan) | 2018-04-02
platformUpdateDomain |  VM을 실행 중인 [업데이트 도메인](../manage-availability.md) | 2017-04-02
platformFaultDomain | VM을 실행 중인 [장애 도메인](../manage-availability.md) | 2017-04-02
provider | VM의 공급자 | 2018-10-01
publicKeys | VM 및 경로에 할당된 [공개 키 컬렉션](/rest/api/compute/virtualmachines/createorupdate#sshpublickey) | 2018-04-02
publisher | VM 이미지 게시자 | 2017-04-02
resourceGroupName | Virtual Machine에 대한 [리소스 그룹](../../azure-resource-manager/management/overview.md) | 2017-08-01
resourceId | 리소스의 [정규화된](/rest/api/resources/resources/getbyid) ID | 2019-03-11
sku | VM 이미지에 해당하는 SKU | 2017-04-02
securityProfile. secureBootEnabled | UEFI 보안 부팅이 VM에서 사용 되는지 여부를 식별 합니다. | 2020-06-01
securityProfile. virtualTpmEnabled | VM에서 TPM (가상 신뢰할 수 있는 플랫폼 모듈)이 사용 되는지 여부를 식별 합니다. | 2020-06-01
storageProfile | [스토리지 프로필](#storage-metadata) 참조 | 2019-06-01
subscriptionId | Virtual Machine에 대한 Azure 구독 | 2017-08-01
tags | Virtual Machine에 대한 [태그](../../azure-resource-manager/management/tag-resources.md)  | 2017-08-01
tagsList | 원활한 프로그래매틱 구문 분석을 위해 JSON 배열로 형식이 지정된 태그  | 2019-06-04
버전 | VM 이미지의 버전 | 2017-04-02
vmId | VM의 [고유 식별자](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) | 2017-04-02
vmScaleSetName | 가상 머신 확장 집합의 [가상 머신 확장 집합 이름](../../virtual-machine-scale-sets/overview.md) | 2017-12-01
vmSize | [VM 크기](../sizes.md) | 2017-04-02
영역 | 가상 머신의 [가용성 영역](../../availability-zones/az-overview.md) | 2017-12-01

### <a name="sample-1-tracking-vm-running-on-azure"></a>샘플 1: Azure에서 실행 중인 VM 추적

서비스 공급자는 소프트웨어를 실행 중인 VM의 수를 추적하거나 VM의 고유성을 추적해야 하는 에이전트가 있어야 합니다. VM의 고유 ID를 가져오려면 Instance Metadata Service의 `vmId` 필드를 사용합니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/compute/vmId?api-version=2017-08-01&format=text"
```

**응답**

```text
5c08b38e-4d57-4c23-ac45-aca61037f084
```

### <a name="sample-2-placement-of-containers-data-partitions-based-faultupdate-domain"></a>샘플 2: 장애/업데이트 도메인 기반 컨테이너, 데이터 파티션 배치

특정 시나리오의 경우 다양한 데이터 복제본 배치가 매우 중요합니다. 예를 들어 [HDFS 복제본 배치](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html#Replica_Placement:_The_First_Baby_Steps) 또는 [orchestrator](https://kubernetes.io/docs/user-guide/node-selection/)를 통한 컨테이너 배치를 위해서는 VM을 실행 중인 `platformFaultDomain` 및 `platformUpdateDomain`을 알아야 합니다.
인스턴스에 대해 [가용성 영역](../../availability-zones/az-overview.md)을 사용하여 이러한 결정을 내릴 수도 있습니다.
Instance Metadata Service를 통해 이 데이터를 직접 쿼리할 수 있습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/compute/platformFaultDomain?api-version=2017-08-01&format=text"
```

**응답**

```text
0
```

### <a name="sample-3-getting-more-information-about-the-vm-during-support-case"></a>샘플 3: 지원 사례 중 VM에 대한 자세한 정보 얻기

서비스 공급자는 지원 요청을 받을 수 있으며 이 때 VM에 대해 자세한 정보를 알아야 하는 경우가 있습니다. 고객에게 컴퓨팅 메타데이터를 공유하도록 요청하면 지원 전문가가 Azure에서 VM의 종류에 대해 알 수 있도록 기본 정보가 제공될 수 있습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance/compute?api-version=2020-09-01
```

**응답**

> [!NOTE]
> 응답은 JSON 문자열입니다. 다음 예제 응답은 가독성을 높이기 위해 적절히 인쇄되었습니다.

```json
{
    "azEnvironment": "AZUREPUBLICCLOUD",
    "isHostCompatibilityLayerVm": "true",
    "licenseType":  "Windows_Client",
    "location": "westus",
    "name": "examplevmname",
    "offer": "Windows",
    "osProfile": {
        "adminUsername": "admin",
        "computerName": "examplevmname",
        "disablePasswordAuthentication": "true"
    },
    "osType": "linux",
    "placementGroupId": "f67c14ab-e92c-408c-ae2d-da15866ec79a",
    "plan": {
        "name": "planName",
        "product": "planProduct",
        "publisher": "planPublisher"
    },
    "platformFaultDomain": "36",
    "platformUpdateDomain": "42",
    "publicKeys": [{
            "keyData": "ssh-rsa 0",
            "path": "/home/user/.ssh/authorized_keys0"
        },
        {
            "keyData": "ssh-rsa 1",
            "path": "/home/user/.ssh/authorized_keys1"
        }
    ],
    "publisher": "RDFE-Test-Microsoft-Windows-Server-Group",
    "resourceGroupName": "macikgo-test-may-23",
    "resourceId": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/virtualMachines/examplevmname",
    "securityProfile": {
        "secureBootEnabled": "true",
        "virtualTpmEnabled": "false"
    },
    "sku": "Windows-Server-2012-R2-Datacenter",
    "storageProfile": {
        "dataDisks": [{
            "caching": "None",
            "createOption": "Empty",
            "diskSizeGB": "1024",
            "image": {
                "uri": ""
            },
            "lun": "0",
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampledatadiskname",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }],
        "imageReference": {
            "id": "",
            "offer": "UbuntuServer",
            "publisher": "Canonical",
            "sku": "16.04.0-LTS",
            "version": "latest"
        },
        "osDisk": {
            "caching": "ReadWrite",
            "createOption": "FromImage",
            "diskSizeGB": "30",
            "diffDiskSettings": {
                "option": "Local"
            },
            "encryptionSettings": {
                "enabled": "false"
            },
            "image": {
                "uri": ""
            },
            "managedDisk": {
                "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
                "storageAccountType": "Standard_LRS"
            },
            "name": "exampleosdiskname",
            "osType": "Linux",
            "vhd": {
                "uri": ""
            },
            "writeAcceleratorEnabled": "false"
        }
    },
    "subscriptionId": "xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "tags": "baz:bash;foo:bar",
    "version": "15.05.22",
    "vmId": "02aab8a4-74ef-476e-8182-f6d2ba4166a6",
    "vmScaleSetName": "crpteste9vflji9",
    "vmSize": "Standard_A3",
    "zone": ""
}
```

### <a name="sample-4-getting-azure-environment-where-the-vm-is-running"></a>샘플 4: VM이 실행되는 Azure 환경 가져오기

Azure에는 [Azure Government](https://azure.microsoft.com/overview/clouds/government/)와 같은 다양한 소버린 클라우드가 있습니다. 경우에 따라 몇 가지 런타임 결정을 내리려면 Azure 환경이 필요합니다. 다음 샘플에서는 이러한 동작을 수행하는 방법을 보여줍니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/compute/azEnvironment?api-version=2018-10-01&format=text"
```

**응답**

```text
AzurePublicCloud
```

클라우드 및 Azure 환경의 값은 아래에 나열되어 있습니다.

 클라우드   | Azure 환경
---------|-----------------
[일반 공급되는 모든 글로벌 Azure 지역](https://azure.microsoft.com/regions/)     | AzurePublicCloud
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)              | AzureUSGovernmentCloud
[Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/)         | AzureChinaCloud
[Azure 독일](https://azure.microsoft.com/overview/clouds/germany/)                    | AzureGermanCloud

## <a name="network-metadata"></a>네트워크 메타데이터 

네트워크 메타데이터는 인스턴스 API의 일부입니다. 인스턴스/네트워크 엔드포인트를 통해 다음 네트워크 범주를 사용할 수 있습니다.

데이터 | Description | 도입된 버전
-----|-------------|-----------------------
ipv4/privateIpAddress | VM의 로컬 IPv4 주소 | 2017-04-02
ipv4/publicIpAddress | VM의 공용 IPv4 주소 | 2017-04-02
subnet/address | VM의 서브넷 주소 | 2017-04-02
subnet/prefix | 서브넷 접두사, 예:24 | 2017-04-02
ipv6/ipaddress | VM의 로컬 IPv6 주소 | 2017-04-02
macAddress | VM MAC 주소 | 2017-04-02

> [!NOTE]
> 모든 API 응답은 JSON 문자열입니다. 다음 예제 응답은 모두 가독성을 높이기 위해 적절히 인쇄되었습니다.

#### <a name="sample-1-retrieving-network-information"></a>샘플 1: 네트워크 정보 검색

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance/network?api-version=2017-08-01
```

**응답**

> [!NOTE]
> 응답은 JSON 문자열입니다. 다음 예제 응답은 가독성을 높이기 위해 적절히 인쇄되었습니다.

```json
{
  "interface": [
    {
      "ipv4": {
        "ipAddress": [
          {
            "privateIpAddress": "10.1.0.4",
            "publicIpAddress": "X.X.X.X"
          }
        ],
        "subnet": [
          {
            "address": "10.1.0.0",
            "prefix": "24"
          }
        ]
      },
      "ipv6": {
        "ipAddress": []
      },
      "macAddress": "000D3AF806EC"
    }
  ]
}

```

#### <a name="sample-2-retrieving-public-ip-address"></a>샘플 2: 공용 IP 주소 검색

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/network/interface/0/ipv4/ipAddress/0/publicIpAddress?api-version=2017-08-01&format=text"
```

## <a name="storage-metadata"></a>스토리지 메타데이터

스토리지 메타데이터는 instance/compute/storageProfile 엔드포인트의 인스턴스 API의 일부입니다.
VM에 연결된 스토리지 디스크에 대한 세부 정보를 제공합니다. 

VM의 저장소 프로필은 이미지 참조, OS 디스크 및 데이터 디스크의 세 가지 범주로 구분 됩니다.

이미지 참조 개체에는 OS 이미지에 대한 다음 정보가 포함되어 있습니다.

데이터    | Description
--------|-----------------
id      | 리소스 ID
제품   | 플랫폼의 제품 또는 마켓플레이스 이미지
publisher | 이미지 게시자입니다.
sku     | 이미지 SKU
버전 | 플랫폼의 버전 또는 마켓플레이스 이미지

OS 디스크 개체에는 VM에서 사용하는 OS 디스크에 대한 다음 정보가 포함됩니다.

데이터    | Description
--------|-----------------
캐싱 | 캐싱 요구 사항
createOption | VM이 생성된 방법에 대한 정보
diffDiskSettings | 임시 디스크 설정
diskSizeGB | 디스크 크기(GB)
encryptionSettings | 디스크의 암호화 설정
이미지   | 원본 사용자 이미지 가상 하드 디스크
managedDisk | 관리 디스크 매개 변수
name    | 디스크 이름
osType  | 디스크에 포함된 OS 유형
VHD     | 가상 하드 디스크
writeAcceleratorEnabled | writeAccelerator를 디스크에서 사용할 수 있는지 여부

데이터 디스크 배열에는 VM에 연결된 데이터 디스크 목록이 포함됩니다. 각 데이터 디스크 개체에는 다음 정보가 들어 있습니다.

데이터    | Description
--------|-----------------
캐싱 | 캐싱 요구 사항
createOption | VM이 생성된 방법에 대한 정보
diffDiskSettings | 임시 디스크 설정
diskSizeGB | 디스크 크기(GB)
이미지   | 원본 사용자 이미지 가상 하드 디스크
lun     | 디스크의 논리 단위 번호
managedDisk | 관리 디스크 매개 변수
name    | 디스크 이름
VHD     | 가상 하드 디스크
writeAcceleratorEnabled | writeAccelerator를 디스크에서 사용할 수 있는지 여부

다음 예제에서는 VM의 저장소 정보를 쿼리 하는 방법을 보여 줍니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance/compute/storageProfile?api-version=2019-06-01
```

**응답**

> [!NOTE]
> 응답은 JSON 문자열입니다. 다음 예제 응답은 가독성을 높이기 위해 적절히 인쇄되었습니다.

```json
{
    "dataDisks": [
      {
        "caching": "None",
        "createOption": "Empty",
        "diskSizeGB": "1024",
        "image": {
          "uri": ""
        },
        "lun": "0",
        "managedDisk": {
          "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampledatadiskname",
          "storageAccountType": "Standard_LRS"
        },
        "name": "exampledatadiskname",
        "vhd": {
          "uri": ""
        },
        "writeAcceleratorEnabled": "false"
      }
    ],
    "imageReference": {
      "id": "",
      "offer": "UbuntuServer",
      "publisher": "Canonical",
      "sku": "16.04.0-LTS",
      "version": "latest"
    },
    "osDisk": {
      "caching": "ReadWrite",
      "createOption": "FromImage",
      "diskSizeGB": "30",
      "diffDiskSettings": {
        "option": "Local"
      },
      "encryptionSettings": {
        "enabled": "false"
      },
      "image": {
        "uri": ""
      },
      "managedDisk": {
        "id": "/subscriptions/xxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/macikgo-test-may-23/providers/Microsoft.Compute/disks/exampleosdiskname",
        "storageAccountType": "Standard_LRS"
      },
      "name": "exampleosdiskname",
      "osType": "Linux",
      "vhd": {
        "uri": ""
      },
      "writeAcceleratorEnabled": "false"
    }
}
```

## <a name="vm-tags"></a>VM 태그

VM 태그는 인스턴스/컴퓨팅/태그 엔드포인트 아래 인스턴스 API에 포함되어 있습니다.
태그를 특정 분류법에 따라 논리적으로 정리하기 위해 Azure VM에 태그가 적용되었을 수 있습니다. VM에 할당된 태그는 아래 요청을 사용하여 검색할 수 있습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/instance/compute/tags?api-version=2018-10-01&format=text"
```

**응답**

```text
Department:IT;Environment:Test;Role:WebRole
```

`tags` 필드는 태그가 세미콜론으로 구분된 문자열입니다. 이 출력은 태그 자체에서 세미콜론을 사용 하는 경우 문제가 될 수 있습니다. 프로그래밍 방식으로 태그를 추출 하도록 파서를 작성 한 경우에는 필드를 사용 해야 합니다 `tagsList` . `tagsList`필드는 구분 기호가 없는 JSON 배열 이므로 구문 분석 하기가 더 쉽습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/instance/compute/tagsList?api-version=2019-06-04
```

**응답**

```json
[
  {
    "name": "Department",
    "value": "IT"
  },
  {
    "name": "Environment",
    "value": "Test"
  },
  {
    "name": "Role",
    "value": "WebRole"
  }
]
```

## <a name="attested-data"></a>증명된 데이터

인스턴스 메타데이터 서비스에서 처리하는 시나리오 부분은 제공된 데이터가 Azure에서 온 것임을 보증하기 위한 것입니다. Marketplace 이미지가 Azure에서 실행되는 이미지임을 확인할 수 있도록 이 정보의 일부를 서명합니다.

### <a name="sample-1-getting-attested-data"></a>샘플 1: 증명된 데이터 가져오기

> [!NOTE]
> 모든 API 응답은 JSON 문자열입니다. 다음 예제 응답은 가독성을 높이기 위해 적절히 출력되었습니다.

**요청**

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254/metadata/attested/document?api-version=2018-10-01&nonce=1234567890"
```

> [!NOTE]
> IMDS의 캐싱 메커니즘으로 인해 이전에 캐시된 nonce 값이 반환될 수 있습니다.

API 버전은 필수 필드입니다. 지원되는 API 버전은 [사용 섹션](#usage)을 참조하세요.
nonce는 선택적 10자리 문자열입니다. 이를 제공하지 않을 경우 IMDS는 현재 UTC 타임스탬프를 대신 반환합니다.

**응답**

> [!NOTE]
> 응답은 JSON 문자열입니다. 다음 예제 응답은 가독성을 높이기 위해 적절히 인쇄되었습니다.

```json
{
 "encoding":"pkcs7","signature":"MIIEEgYJKoZIhvcNAQcCoIIEAzCCA/8CAQExDzANBgkqhkiG9w0BAQsFADCBugYJKoZIhvcNAQcBoIGsBIGpeyJub25jZSI6IjEyMzQ1NjY3NjYiLCJwbGFuIjp7Im5hbWUiOiIiLCJwcm9kdWN0IjoiIiwicHVibGlzaGVyIjoiIn0sInRpbWVTdGFtcCI6eyJjcmVhdGVkT24iOiIxMS8yMC8xOCAyMjowNzozOSAtMDAwMCIsImV4cGlyZXNPbiI6IjExLzIwLzE4IDIyOjA4OjI0IC0wMDAwIn0sInZtSWQiOiIifaCCAj8wggI7MIIBpKADAgECAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBBAUAMCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tMB4XDTE4MTEyMDIxNTc1N1oXDTE4MTIyMDIxNTc1NlowKzEpMCcGA1UEAxMgdGVzdHN1YmRvbWFpbi5tZXRhZGF0YS5henVyZS5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBAML/tBo86ENWPzmXZ0kPkX5dY5QZ150mA8lommszE71x2sCLonzv4/UWk4H+jMMWRRwIea2CuQ5RhdWAHvKq6if4okKNt66fxm+YTVz9z0CTfCLmLT+nsdfOAsG1xZppEapC0Cd9vD6NCKyE8aYI1pliaeOnFjG0WvMY04uWz2MdAgMBAAGjYDBeMFwGA1UdAQRVMFOAENnYkHLa04Ut4Mpt7TkJFfyhLTArMSkwJwYDVQQDEyB0ZXN0c3ViZG9tYWluLm1ldGFkYXRhLmF6dXJlLmNvbYIQZ8VuSofHbJRAQNBNpiASdDANBgkqhkiG9w0BAQQFAAOBgQCLSM6aX5Bs1KHCJp4VQtxZPzXF71rVKCocHy3N9PTJQ9Fpnd+bYw2vSpQHg/AiG82WuDFpPReJvr7Pa938mZqW9HUOGjQKK2FYDTg6fXD8pkPdyghlX5boGWAMMrf7bFkup+lsT+n2tRw2wbNknO1tQ0wICtqy2VqzWwLi45RBwTGB6DCB5QIBATA/MCsxKTAnBgNVBAMTIHRlc3RzdWJkb21haW4ubWV0YWRhdGEuYXp1cmUuY29tAhBnxW5Kh8dslEBA0E2mIBJ0MA0GCSqGSIb3DQEBCwUAMA0GCSqGSIb3DQEBAQUABIGAld1BM/yYIqqv8SDE4kjQo3Ul/IKAVR8ETKcve5BAdGSNkTUooUGVniTXeuvDj5NkmazOaKZp9fEtByqqPOyw/nlXaZgOO44HDGiPUJ90xVYmfeK6p9RpJBu6kiKhnnYTelUk5u75phe5ZbMZfBhuPhXmYAdjc7Nmw97nx8NnprQ="
}
```

서명 Blob은 [pkcs7](https://aka.ms/pkcs7)으로 서명된 버전의 문서입니다. 여기에는 특정 VM 관련 세부 정보와 함께 서명에 사용 되는 인증서가 포함 됩니다. ARM Vm의 경우 여기에는 vmId, sku, nonce, subscriptionId, 문서를 만들고 만료 하기 위한 타임 스탬프 및 이미지에 대 한 계획 정보가 포함 됩니다. 요금제 정보는 Azure Marketplace 이미지에 대해서만 채워집니다. 클래식 (ARM이 아닌) Vm의 경우 vmId만 채워집니다. 응답에서 추출한 인증서를 사용하여 응답이 유효하고 Azure에서 제공되는지 확인할 수 있습니다.
문서에는 다음 필드가 포함 되어 있습니다.

데이터 | Description | 도입된 버전
-----|-------------|-----------------------
licenseType | [Azure 하이브리드 혜택](https://azure.microsoft.com/pricing/hybrid-benefit)라이선스의 유형입니다. 이는 AHB 사용 Vm에 대해서만 존재 합니다. | 2020-09-01
nonce | 요청에 선택적으로 제공할 수 있는 문자열입니다. Nonce를 제공 하지 않으면 현재 UTC 타임 스탬프가 사용 됩니다. | 2018-10-01
계획 | [Azure Marketplace 이미지 계획](/rest/api/compute/virtualmachines/createorupdate#plan)입니다. 계획 ID (이름), 제품 이미지 또는 제품 (제품) 및 게시자 ID (게시자)를 포함 합니다. | 2018-10-01
타임스탬프/createdOn | 서명 된 문서를 만든 시간에 대 한 UTC 타임 스탬프입니다. | 2018-20-01
timestamp/expiresOn | 서명 된 문서가 만료 되는 시간에 대 한 UTC 타임 스탬프입니다. | 2018-10-01
vmId |  VM의 [고유 식별자](https://azure.microsoft.com/blog/accessing-and-using-azure-vm-unique-id/) | 2018-10-01
subscriptionId | Virtual Machine에 대한 Azure 구독 | 2019-04-30
sku | VM 이미지에 해당하는 SKU | 2019-11-01

> [!NOTE]
> 클래식 (ARM이 아닌) Vm의 경우 vmId만 채워집니다.

### <a name="sample-2-validating-that-the-vm-is-running-in-azure"></a>샘플 2: VM이 Azure에서 실행 중인지 확인

Marketplace 공급업체는 해당 소프트웨어가 Azure에서만 실행되도록 사용이 허가되었는지 확인하려고 합니다. 사용자가 온-프레미스에 VHD를 복사하는 경우 이 사실을 검색하는 방법이 있어야 합니다. Instance Metadata Service를 호출함으로써 Marketplace 공급업체는 Azure의 응답임을 보증하는 서명된 데이터를 얻을 수 있습니다.

```powershell
# Get the signature
$attestedDoc = Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri http://169.254.169.254/metadata/attested/document?api-version=2020-09-01
# Decode the signature
$signature = [System.Convert]::FromBase64String($attestedDoc.signature)
```

서명이 Microsoft Azure 인지 확인 하 고 인증서 체인에서 오류를 확인 합니다.

```powershell
# Get certificate chain
$cert = [System.Security.Cryptography.X509Certificates.X509Certificate2]($signature)
$chain = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Chain
$chain.Build($cert)
# Print the Subject of each certificate in the chain
foreach($element in $chain.ChainElements)
{
    Write-Host $element.Certificate.Subject
}

# Get the content of the signed document
Add-Type -AssemblyName System.Security
$signedCms = New-Object -TypeName System.Security.Cryptography.Pkcs.SignedCms
$signedCms.Decode($signature);
$content = [System.Text.Encoding]::UTF8.GetString($signedCms.ContentInfo.Content)
Write-Host "Attested data: " $content
$json = $content | ConvertFrom-Json
# Do additional validation here
```

> [!NOTE]
> IMDS의 캐싱 메커니즘으로 인해 이전에 캐시된 nonce 값이 반환될 수 있습니다.

초기 요청에서 nonce 매개 변수를 제공한 경우 서명 된 문서에서 nonce를 비교할 수 있습니다.

> [!NOTE]
> 공용 클라우드와 각 소 버린 클라우드의 인증서가 서로 다릅니다.

클라우드 | 인증서
------|------------
[일반 공급되는 모든 글로벌 Azure 지역](https://azure.microsoft.com/regions/) | *.metadata.azure.com
[Azure Government](https://azure.microsoft.com/overview/clouds/government/)          | *.metadata.azure.us
[Azure China 21Vianet](https://azure.microsoft.com/global-infrastructure/china/)     | *.metadata.azure.cn
[Azure 독일](https://azure.microsoft.com/overview/clouds/germany/)                | *.metadata.microsoftazure.de

> [!NOTE]
> 서명에 사용되는 인증서와 관련하여 알려진 문제가 있습니다. 인증서에 퍼블릭 클라우드의 `metadata.azure.com`과 정확하게 일치하는 항목이 없을 수 있습니다. 따라서 인증 유효성 검사는 `.metadata.azure.com` 하위 도메인의 일반 이름을 허용해야 합니다.

유효성을 검사하는 동안 네트워크 제약 조건으로 인해 중간 인증서를 다운로드할 수 없는 경우 중간 인증서를 고정할 수 있습니다. 그러나 Azure는 표준 PKI 방법에 따라 인증서를 롤오버합니다. 롤오버가 발생하면 고정된 인증서를 업데이트해야 합니다. 중간 인증서 업데이트에 대한 변경 사항이 계획될 때마다 Azure 블로그가 업데이트되고 Azure 고객에게 알림이 제공됩니다. [여기](https://www.microsoft.com/pki/mscorp/cps/default.htm)에서 중간 인증서를 확인할 수 있습니다. 각 지역의 중간 인증서는 다를 수 있습니다.

> [!NOTE]
> Azure China 21Vianet의 중간 인증서는 Baltimore가 아닌 DigiCert Global Root CA에서 가져옵니다.
또한 루트 체인 인증 기관을 변경하는 과정에서 Azure 중국에 대한 중간 인증서를 고정한 경우 중간 인증서를 업데이트해야 합니다.

## <a name="failover-clustering-in-windows-server"></a>Windows Server에서의 장애 조치(failover) 클러스터링

장애 조치 (Failover) 클러스터링을 사용 하 여 Instance Metadata Service를 쿼리 하는 특정 시나리오의 경우 라우팅 테이블에 경로를 추가 해야 합니다.

1. 관리자 권한으로 명령 프롬프트를 엽니다.

1. 다음 명령을 실행하고 IPv4 경로 테이블에 네트워크 대상의 인터페이스 주소(`0.0.0.0`)를 기록합니다.

```bat
route print
```

> [!NOTE]
> 장애 조치(failover) 클러스터를 사용하는 Windows Server VM의 다음 예제 출력에는 간단히 나타내기 위해 IPv4 경로 테이블만 포함되어 있습니다.

```text
IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0         10.0.1.1        10.0.1.10    266
         10.0.1.0  255.255.255.192         On-link         10.0.1.10    266
        10.0.1.10  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.15  255.255.255.255         On-link         10.0.1.10    266
        10.0.1.63  255.255.255.255         On-link         10.0.1.10    266
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
      169.254.0.0      255.255.0.0         On-link     169.254.1.156    271
    169.254.1.156  255.255.255.255         On-link     169.254.1.156    271
  169.254.255.255  255.255.255.255         On-link     169.254.1.156    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link     169.254.1.156    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link     169.254.1.156    271
  255.255.255.255  255.255.255.255         On-link         10.0.1.10    266
```

다음 명령을 실행하고 네트워크 대상의 인터페이스 주소(`0.0.0.0`)를 사용합니다(이 예제에서는 `10.0.1.10`).

```bat
route add 169.254.169.254/32 10.0.1.10 metric 1 -p
```

## <a name="managed-identity-via-metadata-service"></a>Metadata Service를 통해 관리되는 ID

VM에서 시스템 할당 관리 id를 사용 하도록 설정 하거나 하나 이상의 사용자 할당 관리 id를 VM에 할당할 수 있습니다.
그런 다음 관리 되는 id에 대 한 토큰을 Instance Metadata Service에서 요청할 수 있습니다. 이러한 토큰을 사용 하 여 Azure Key Vault 같은 다른 Azure 서비스를 인증할 수 있습니다.

이 기능을 사용하도록 설정하는 자세한 단계는 [액세스 토큰 획득](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md)을 참조하세요.

## <a name="scheduled-events-via-metadata-service"></a>Metadata Service를 통한 Scheduled Events
메타데이터 서비스를 통해 예약된 이벤트의 상태를 가져올 수 있으며, 그러면 사용자가 이러한 이벤트에 대해 실행할 작업 세트를 지정할 수 있습니다.  자세한 내용은 [Scheduled Events](scheduled-events.md)를 참조하세요. 

## <a name="regional-availability"></a>국가별 가용성

서비스는 일반적으로 모든 Azure 클라우드에서 **사용할 수** 있습니다.

## <a name="sample-code-in-different-languages"></a>다양한 언어로 된 샘플 코드

VM 내의 서로 다른 언어를 사용하여 메타데이터 서비스를 호출하는 샘플:

언어      | 예제
--------------|----------------
C++(Windows) | https://github.com/Microsoft/azureimds/blob/master/IMDSSample-windows.cpp
C#            | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.cs
Go            | https://github.com/Microsoft/azureimds/blob/master/imdssample.go
Java          | https://github.com/Microsoft/azureimds/blob/master/imdssample.java
NodeJS        | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.js
Perl          | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.pl
PowerShell    | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.ps1
Puppet        | https://github.com/keirans/azuremetadata
Python        | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.py
Ruby          | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.rb
Visual Basic  | https://github.com/Microsoft/azureimds/blob/master/IMDSSample.vb

## <a name="error-and-debugging"></a>오류 및 디버깅

찾을 수 없는 데이터 요소 또는 형식이 잘못된 요청이 있으면 Instance Metadata Service는 표준 HTTP 오류를 반환합니다. 다음은 그 예입니다.

HTTP 상태 코드 | 이유
-----------------|-------
200 정상 |
400 잘못된 요청 | `Metadata: true` `format=json` 리프 노드를 쿼리할 때 누락 된 헤더 또는 누락 된 매개 변수
404 찾을 수 없음  | 요청된 요소가 없음
405 메서드를 사용할 수 없음 | `GET` 요청만 지원됨
410 없음 | 최대 70초 후 다시 시도
429 요청이 너무 많음 | API는 현재 초당 최대 5개의 쿼리를 지원함
500 서비스 오류     | 잠시 후 다시 시도하세요.

### <a name="known-issues-and-faq"></a>알려진 문제 및 FAQ

1. `400 Bad Request, Required metadata header not specified` 오류가 발생했습니다. 무슨 의미인가요?
   * Instance Metadata Service에서는 `Metadata: true` 헤더를 요청에 포함시켜 전달해야 합니다. REST 호출에서 이 헤더를 전달하면 Instance Metadata Service에 액세스가 허용됩니다.
1. VM에 대한 컴퓨팅 정보를 구할 수 없는 이유가 무엇인가요?
   * 현재 Instance Metadata Service는 Azure Resource Manager를 사용하여 만든 인스턴스만 지원합니다.
1. 잠시 전에 Azure Resource Manager를 통해 내 Virtual Machine을 만들었습니다. 컴퓨팅 메타데이터 정보가 왜 표시되지 않나요?
   * 2016년 9월 이후에 생성된 VM의 경우 컴퓨팅 메타데이터를 표시하려면 [태그](../../azure-resource-manager/management/tag-resources.md)를 추가하세요. 이전 Vm (9 월 2016 일 이전에 만들어짐)의 경우 VM 인스턴스에 확장 또는 데이터 디스크를 추가/제거 하 여 메타 데이터를 새로 고칩니다.
1. 새 버전용으로 채워진 데이터 중 일부만 표시됩니다.
   * 2016년 9월 이후에 생성된 VM의 경우 컴퓨팅 메타데이터를 표시하려면 [태그](../../azure-resource-manager/management/tag-resources.md)를 추가하세요. 이전 Vm (9 월 2016 일 이전에 만들어짐)의 경우 VM 인스턴스에 확장 또는 데이터 디스크를 추가/제거 하 여 메타 데이터를 새로 고칩니다.
1. 오류가 발생 하는 이유는 무엇 인가요 `500 Internal Server Error` `410 Resource Gone` ?
   * 지 수 백오프 시스템 또는 [일시적인 오류 처리](/azure/architecture/best-practices/transient-faults)에 설명 된 다른 방법에 따라 요청을 다시 시도 하세요. 문제가 지속 되 면 VM에 대 한 Azure Portal에서 지원 문제를 만듭니다.
1. 가상 머신 확장 집합 인스턴스에 대해이 작업을 수행 하나요?
   * 예를 들어 확장 집합 인스턴스에는 Metadata service를 사용할 수 있습니다.
1. Virtual Machine Scale Sets에서 태그를 업데이트 했지만 단일 인스턴스 Vm과 달리 인스턴스에 표시 되지 않나요?
   * 현재 크기 집합에 대 한 태그는 다시 부팅, 이미지로 다시 설치 또는 인스턴스에 대 한 디스크 변경과 같이 VM에만 표시 됩니다.
1. 서비스를 호출하는 데 요청 시간이 초과됩니다.
   * 메타 데이터 호출은 VM의 기본 네트워크 카드에 할당 된 기본 IP 주소에서 이루어져야 합니다. 또한 경로를 변경한 경우 VM의 로컬 라우팅 테이블에 169.254.169.254/32 주소에 대 한 경로가 있어야 합니다.
   * <details>
        <summary>라우팅 테이블을 확인 하는 중</summary>

        1. 로컬 라우팅 테이블을 덤프 하 고 IMDS 항목을 찾습니다 (예:).
            ```console
            > route print
            IPv4 Route Table
            ===========================================================================
            Active Routes:
            Network Destination        Netmask          Gateway       Interface  Metric
                      0.0.0.0          0.0.0.0      172.16.69.1      172.16.69.7     10
                    127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
                    127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
              127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
                168.63.129.16  255.255.255.255      172.16.69.1      172.16.69.7     11
              169.254.169.254  255.255.255.255      172.16.69.1      172.16.69.7     11
            ... (continues) ...
            ```
        1. 에 대 한 경로가 있는지 확인 하 `169.254.169.254` 고 해당 네트워크 인터페이스 (예:)를 확인 합니다. `172.16.69.7`
        1. 인터페이스 구성을 덤프 하 고, MAC (물리적) 주소를 확인 하 여 라우팅 테이블에서 참조 된 항목에 해당 하는 인터페이스를 찾습니다.
            ```console
            > ipconfig /all
            ... (continues) ...
            Ethernet adapter Ethernet:

               Connection-specific DNS Suffix  . : xic3mnxjiefupcwr1mcs1rjiqa.cx.internal.cloudapp.net
               Description . . . . . . . . . . . : Microsoft Hyper-V Network Adapter
               Physical Address. . . . . . . . . : 00-0D-3A-E5-1C-C0
               DHCP Enabled. . . . . . . . . . . : Yes
               Autoconfiguration Enabled . . . . : Yes
               Link-local IPv6 Address . . . . . : fe80::3166:ce5a:2bd5:a6d1%3(Preferred)
               IPv4 Address. . . . . . . . . . . : 172.16.69.7(Preferred)
               Subnet Mask . . . . . . . . . . . : 255.255.255.0
            ... (continues) ...
            ```
        1. 인터페이스가 VM의 기본 NIC 및 기본 IP에 해당 하는지 확인 합니다. Azure Portal에서 네트워크 구성을 확인 하거나 [Azure CLI](/cli/azure/vm/nic?view=azure-cli-latest#az-vm-nic-show)를 조회 하 여 기본 NIC/IP를 찾을 수 있습니다. 공용 및 개인 Ip (및 CLI를 사용 하는 경우 MAC 주소)를 확인 합니다. PowerShell CLI 예제:
            ```powershell
            $ResourceGroup = '<Resource_Group>'
            $VmName = '<VM_Name>'
            $NicNames = az vm nic list --resource-group $ResourceGroup --vm-name $VmName | ConvertFrom-Json | Foreach-Object { $_.id.Split('/')[-1] }
            foreach($NicName in $NicNames)
            {
                $Nic = az vm nic show --resource-group $ResourceGroup --vm-name $VmName --nic $NicName | ConvertFrom-Json
                Write-Host $NicName, $Nic.primary, $Nic.macAddress
            }
            # Output: wintest767 True 00-0D-3A-E5-1C-C0
            ```
        1. 일치 하지 않는 경우 기본 NIC/i p의 대상이 되도록 라우팅 테이블을 업데이트 합니다.
    </details>

## <a name="support-and-feedback"></a>지원 및 피드백

에서 사용자 의견 및 의견을 제출 https://feedback.azure.com 합니다.

서비스에 대한 지원을 받으려면 Azure Portal에서 긴 다시 시도 후 메타데이터 응답을 받을 수 없는 VM에 대한 지원 문제를 만듭니다.
의 문제 유형을 사용 `Management` 하 고 범주를 선택 `Instance Metadata Service` 합니다.

![인스턴스 메타데이터 지원](./media/instance-metadata-service/InstanceMetadata-support.png "스크린샷: Instance Metadata Service에 문제가 있을 때 지원 사례 열기")

## <a name="next-steps"></a>다음 단계

다음에 대해 자세히 알아봅니다.
1.  [VM의 액세스 토큰 획득](../../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md).
2.  [Scheduled Events](scheduled-events.md)
