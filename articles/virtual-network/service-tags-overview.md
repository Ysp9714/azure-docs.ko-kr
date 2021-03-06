---
title: Azure 서비스 태그 개요
titlesuffix: Azure Virtual Network
description: 서비스 태그에 대해 알아봅니다. 서비스 태그는 보안 규칙 생성의 복잡성을 최소화하는 데 유용합니다.
services: virtual-network
documentationcenter: na
author: allegradomel
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/30/2020
ms.author: kumud
ms.reviewer: kumud
ms.openlocfilehash: d99ed9304612d5c1d3cbdc56a76cee751f2a4d3f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966737"
---
# <a name="virtual-network-service-tags"></a>가상 네트워크 서비스 태그
<a name="network-service-tags"></a>

서비스 태그는 지정된 Azure 서비스의 IP 주소 접두사 그룹을 나타냅니다. Microsoft에서는 서비스 태그에서 압축한 주소 접두사를 관리하고 주소를 변경하는 대로 서비스 태그를 자동으로 업데이트하여 네트워크 보안 규칙을 자주 업데이트할 때 발생하는 복잡성을 최소화합니다.

서비스 태그를 사용 하 여 [네트워크 보안 그룹](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) 또는 [Azure 방화벽](https://docs.microsoft.com/azure/firewall/service-tags)에서 네트워크 액세스 제어를 정의할 수 있습니다. 보안 규칙을 만들 때 특정 IP 주소 대신 서비스 태그를 사용합니다. 규칙의 적절 한 *원본* 또는 *대상* 필드에 **microsoft.apimanagement** 와 같은 서비스 태그 이름을 지정 하 여 해당 서비스에 대 한 트래픽을 허용 하거나 거부할 수 있습니다.

퍼블릭 엔드포인트가 있는 Azure 서비스에 액세스하면서, 서비스 태그를 사용하여 네트워크 격리를 수행하고 일반 인터넷에서 Azure 리소스를 보호할 수 있습니다. 인바운드/아웃바운드 네트워크 보안 그룹 규칙을 만들어 **인터넷** 에서 들어오고 나가는 트래픽을 거부하고 **AzureCloud** 또는 특정 Azure 서비스의 다른 [사용 가능한 서비스 태그](#available-service-tags)에서 들어오고 나가는 트래픽을 허용합니다.

![서비스 태그를 사용 하 여 Azure 서비스의 네트워크 격리](./media/service-tags-overview/service_tags.png)

## <a name="available-service-tags"></a>사용 가능한 서비스 태그
다음 표에는 [네트워크 보안 그룹](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) 규칙에서 사용할 수 있는 모든 서비스 태그가 나와 있습니다.

열은 태그가 다음에 해당하는지 여부를 나타냅니다.

- 인바운드 또는 아웃바운드 트래픽을 포함하는 규칙에 적합합니다.
- [지역](https://azure.microsoft.com/regions) 범위를 지원합니다.
- [Azure Firewall](https://docs.microsoft.com/azure/firewall/service-tags) 규칙에서 사용할 수 있습니다.

기본적으로 서비스 태그는 전체 클라우드의 범위를 반영합니다. 일부 서비스 태그는 해당 IP 범위를 지정된 지역으로 제한하여 보다 자세히 제어할 수 있습니다. 예를 들어, 서비스 태그 **Storage** 는 전체 클라우드의 Azure Storage를 나타내지만 **Storage.WestUS** 는 범위를 WestUS 지역의 스토리지 IP 주소 범위로 좁힙니다. 다음 표에서는 각 서비스 태그가 이러한 지역 범위를 지원하는지 여부를 나타냅니다.  

| 태그 | 목적 | 인바운드 또는 아웃바운드를 사용할 수 있나요? | 지역 범위를 지원할 수 있나요? | Azure Firewall에서 사용할 수 있나요? |
| --- | -------- |:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **ActionGroup** | 작업 그룹입니다. | 인바운드 | 예 | 예 |
| **ApiManagement** | Azure API Management 전용 배포에 대한 관리 트래픽입니다. <br/><br/>*참고:* 이 태그는 지역별 제어 평면에 대한 Azure API Management 서비스 엔드포인트를 나타냅니다. 이렇게 하면 고객이 API Management 서비스에 구성된 API, 작업, 정책, 명명된 값에 대해 관리 작업을 수행할 수 있습니다.  | 인바운드 | 예 | 예 |
| **ApplicationInsightsAvailability** | Application Insights 가용성입니다. | 인바운드 | 예 | 예 |
| **AppConfiguration** | 앱 구성입니다. | 아웃바운드 | 예 | 예 |
| **AppService**    | Azure App Service 이 태그는 웹 앱 및 함수 앱에 대 한 아웃 바운드 보안 규칙에 권장 됩니다.  | 아웃바운드 | 예 | 예 |
| **AppServiceManagement** | App Service Environment 전용 배포에 대한 관리 트래픽입니다. | 모두 | 예 | 예 |
| **AzureActiveDirectory** | Azure Active Directory. | 아웃바운드 | 예 | 예 |
| **AzureActiveDirectoryDomainServices** | Azure Active Directory Domain Services 전용 배포에 대한 관리 트래픽 | 모두 | 예 | 예 |
| **AzureAdvancedThreatProtection** | Azure Advanced Threat Protection | 아웃바운드 | 예 | 예 |
| **AzureBackup** |Azure Backup<br/><br/>*참고:* 이 태그는 **Storage** 및 **AzureActiveDirectory** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzureBotService** | Azure Bot Service | 아웃바운드 | 예 | 예 |
| **AzureCloud** | 모든 [데이터 센터 퍼블릭 IP 주소](https://www.microsoft.com/download/details.aspx?id=56519)입니다. | 아웃바운드 | 예 | 예 |
| **AzureCognitiveSearch** | Azure Cognitive Search. <br/><br/>이 태그 또는 이 태그가 적용되는 IP 주소를 사용하여 데이터 원본에 대한 보안 액세스 권한을 인덱서에 부여할 수 있습니다. 자세한 내용은 [인덱서 연결 설명서](https://docs.microsoft.com/azure/search/search-indexer-troubleshooting#connection-errors) 를 참조 하세요. <br/><br/> *참고*: 검색 서비스의 IP는 이 서비스 태그의 IP 범위 목록에 포함되지 않으므로 데이터 원본의 IP 방화벽에 **도 추가해야 합니다**. | 인바운드 | 예 | 예 |
| **AzureConnectors** | 프로브/백 엔드 연결용 Azure Logic Apps 커넥터입니다. | 인바운드 | 예 | 예 |
| **AzureContainerRegistry** | Azure Container Registry입니다. | 아웃바운드 | 예 | 예 |
| **AzureCosmosDB** | Azure Cosmos DB | 아웃바운드 | 예 | 예 |
| **AzureDatabricks** | Azure Databricks입니다. | 모두 | 예 | 예 |
| **AzureDataExplorerManagement** | Azure Data Explorer 관리 기능입니다. | 인바운드 | 예 | 예 |
| **AzureDataLake** | Azure Data Lake Storage Gen1입니다. | 아웃바운드 | 예 | 예 |
| **AzureDevSpaces** | Azure Dev Spaces입니다. | 아웃바운드 | 예 | 예 |
| **AzureDevOps** | Azure Dev Ops.<br/><br/>*참고:이 태그는 현재 Azure Portal을 통해 구성할 수 없습니다.*| 인바운드 | 예 | 예 |
| **AzureDigitalTwins** | Azure Digital Twins.<br/><br/>*참고:* 이 태그 또는이 태그에 포함 된 IP 주소는 이벤트 경로에 대해 구성 된 끝점에 대 한 액세스를 제한 하는 데 사용할 수 있습니다. *이 태그는 현재 Azure Portal을 통해 구성할 수 없습니다.* | 인바운드 | 예 | 예 |
| **AzureEventGrid** | Azure Event Grid. | 모두 | 예 | 예 |
| **AzureFrontDoor.Frontend** <br/> **AzureFrontDoor.Backend** <br/> **AzureFrontDoor.FirstParty**  | Azure Front Door입니다. | 모두 | 예 | 예 |
| **AzureInformationProtection** | Azure Information Protection입니다.<br/><br/>*참고:* 이 태그는 **AzureActiveDirectory**, **AzureFrontDoor.Frontend** 및 **AzureFrontDoor.FirstParty** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzureIoTHub** | Azure IoT Hub입니다. | 아웃바운드 | 예 | 예 |
| **AzureKeyVault** | Azure Key Vault.<br/><br/>*참고:* 이 태그는 **AzureActiveDirectory** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzureLoadBalancer** | Azure 인프라 부하 분산 장치입니다. 이 태그는 Azure의 상태 프로브가 시작되는 [호스트의 가상 IP 주소](security-overview.md#azure-platform-considerations)(168.63.129.16)로 변환됩니다. 이는 백 엔드 리소스에 대 한 실제 트래픽이 아닌 프로브 트래픽만 포함 합니다. Azure Load Balancer를 사용하지 않는 경우 이 규칙을 재정의할 수 있습니다. | 모두 | 예 | 예 |
| **AzureMachineLearning** | Azure Machine Learning입니다. | 모두 | 예 | 예 |
| **AzureMonitor** | Log Analytics, Application Insights, AzMon 및 사용자 지정 메트릭(GIG 엔드포인트)입니다.<br/><br/>*참고:* Log Analytics의 경우 이 태그는 **Storage** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzureOpenDatasets** | Azure Open Datasets입니다.<br/><br/>*참고:* 이 태그는 **AzureFrontDoor** 및 **Storage** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzurePlatformDNS** | 기본 인프라(기본값) DNS 서비스입니다.<br/><br>이 태그를 사용하여 기본 DNS를 사용하지 않도록 설정할 수 있습니다. 이 태그를 사용할 때는 주의해야 합니다. [Azure 플랫폼 고려 사항](https://docs.microsoft.com/azure/virtual-network/security-overview#azure-platform-considerations)을 읽어 보는 것이 좋습니다. 또한 이 태그를 사용하기 전에 테스트를 수행하는 것이 좋습니다. | 아웃바운드 | 예 | 예 |
| **AzurePlatformIMDS** | 기본 인프라 서비스인 Azure IMDS(Instance Metadata Service)입니다.<br/><br/>이 태그를 사용하여 기본 IMDS를 사용하지 않도록 설정할 수 있습니다. 이 태그를 사용할 때는 주의해야 합니다. [Azure 플랫폼 고려 사항](https://docs.microsoft.com/azure/virtual-network/security-overview#azure-platform-considerations)을 읽어 보는 것이 좋습니다. 또한 이 태그를 사용하기 전에 테스트를 수행하는 것이 좋습니다. | 아웃바운드 | 예 | 예 |
| **AzurePlatformLKM** | Windows 라이선스 또는 키 관리 서비스입니다.<br/><br/>이 태그를 사용하여 라이선스에 대한 기본값을 사용하지 않도록 설정할 수 있습니다. 이 태그를 사용할 때는 주의해야 합니다. [Azure 플랫폼 고려 사항](https://docs.microsoft.com/azure/virtual-network/security-overview#azure-platform-considerations)을 읽어 보는 것이 좋습니다.  또한 이 태그를 사용하기 전에 테스트를 수행하는 것이 좋습니다. | 아웃바운드 | 예 | 예 |
| **AzureResourceManager** | Azure Resource Manager입니다. | 아웃바운드 | 예 | 예 |
| **AzureSignalR** | Azure SignalR입니다. | 아웃바운드 | 예 | 예 |
| **AzureSiteRecovery** | Azure Site Recovery입니다.<br/><br/>*참고:* 이 태그는 **AzureActiveDirectory**, **AzureKeyVault**, **EventHub**,**GuestAndHybridManagement** 및 **Storage** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **AzureTrafficManager** | Azure Traffic Manager 프로브 IP 주소입니다.<br/><br/>Traffic Manager 프로브 IP 주소에 대한 자세한 내용은 [Azure Traffic Manager FAQ](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-faqs)를 참조하세요. | 인바운드 | 예 | 예 |  
| **BatchNodeManagement** | Azure Batch 전용 배포에 대한 관리 트래픽입니다. | 모두 | 예 | 예 |
| **CognitiveServicesManagement** | Azure Cognitive Services에 대한 트래픽 주소 범위입니다. | 모두 | 예 | 예 |
| **DataFactory**  | Azure 데이터 팩터리 | 모두 | 예 | 예 |
| **DataFactoryManagement** | Azure Data Factory에 대한 관리 트래픽입니다. | 아웃바운드 | 예 | 예 |
| **Dynamics365ForMarketingEmail** | Dynamics 365 마케팅 메일 서비스의 주소 범위입니다. | 아웃바운드 | 예 | 예 |
| **EventHub** | Azure Event Hubs입니다. | 아웃바운드 | 예 | 예 |
| **GatewayManager** | Azure VPN Gateway 및 Application Gateway 전용 배포에 대한 관리 트래픽입니다. | 인바운드 | 예 | 예 |
| **GuestAndHybridManagement** | Azure Automation 및 게스트 구성입니다. | 아웃바운드 | 예 | 예 |
| **HDInsight** | Azure HDInsight. | 인바운드 | 예 | 예 |
| **인터넷** | 가상 네트워크 외부에 있으며 공용 인터넷으로 연결할 수 있는 IP 주소 공간입니다.<br/><br/>주소 범위에는 [Azure에서 소유하는 퍼블릭 IP 주소 공간](https://www.microsoft.com/download/details.aspx?id=41653)이 포함됩니다. | 모두 | 예 | 예 |
| **LogicApps** | Logic Apps입니다. | 모두 | 예 | 예 |
| **LogicAppsManagement** | Logic Apps에 대한 관리 트래픽입니다. | 인바운드 | 예 | 예 |
| **MicrosoftCloudAppSecurity** | Microsoft Cloud App Security입니다. | 아웃바운드 | 예 | 예 |
| **MicrosoftContainerRegistry** | Microsoft 컨테이너 이미지용 컨테이너 레지스트리입니다. <br/><br/>*참고:* 이 태그는 **AzureFrontDoor** 태그에 종속됩니다. | 아웃바운드 | 예 | 예 |
| **PowerQueryOnline** | 파워 쿼리 온라인입니다. | 모두 | 예 | 예 |
| **Service Bus** | 프리미엄 서비스 계층을 사용하는 Azure Service Bus 트래픽입니다. | 아웃바운드 | 예 | 예 |
| **ServiceFabric** | Azure Service Fabric입니다.<br/><br/>*참고:* 이 태그는 지역별 제어 평면에 대한 Service Fabric 서비스 엔드포인트를 나타냅니다. 이를 통해 고객은 자신의 VNET에서 Service Fabric 클러스터에 대한 관리 작업을 수행할 수 있습니다(엔드포인트 예: https://westus.servicefabric.azure.com). | 모두 | 예 | 예 |
| **Sql** | Azure SQL Database, Azure Database for MySQL, Azure Database for PostgreSQL 및 Azure Synapse Analytics가 있습니다.<br/><br/>*참고:* 이 태그는 서비스의 특정 인스턴스가 아니라 서비스를 나타냅니다. 예를 들어 태그는 특정 SQL 데이터베이스 또는 서버가 아닌 Azure SQL Database 서비스를 나타냅니다. 이 태그는 SQL Managed Instance에 적용되지 않습니다. | 아웃바운드 | 예 | 예 |
| **SqlManagement** | SQL 전용 배포에 대한 관리 트래픽입니다. | 모두 | 예 | 예 |
| **스토리지** | Azure Storage. <br/><br/>*참고:* 이 태그는 서비스의 특정 인스턴스가 아니라 서비스를 나타냅니다. 예를 들어 태그는 특정 Azure Storage 계정이 아닌 Azure Storage 서비스를 나타냅니다. | 아웃바운드 | 예 | 예 |
| **StorageSyncService** | 스토리지 동기화 서비스입니다. | 모두 | 예 | 예 |
| **WindowsVirtualDesktop** | Windows 가상 데스크톱입니다. | 모두 | 예 | 예 |
| **VirtualNetwork** | 가상 네트워크 주소 공간(가상 네트워크에 대해 정의된 모든 IP 주소 범위), 연결된 모든 온-프레미스 주소 공간 및 [피어링](virtual-network-peering-overview.md)된 가상 네트워크 또는 [가상 네트워크 게이트웨이](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%3ftoc.json)에 연결된 가상 네트워크, [사용자 정의 경로](virtual-networks-udr-overview.md)에 사용되는 [호스트의 가상 IP 주소](security-overview.md#azure-platform-considerations)입니다. 이 태그에는 기본 경로가 포함될 수도 있습니다. | 모두 | 예 | 예 |

>[!NOTE]
>클래식 배포 모델(Azure Resource Manager 이전)에서는 이전 표에 나열된 태그의 하위 세트가 지원됩니다. 이러한 태그는 다음과 같이 철자가 다릅니다.
>
>| 클래식 맞춤법 | 해당 Resource Manager 태그 |
>|---|---|
>| AZURE_LOADBALANCER | AzureLoadBalancer |
>| 인터넷 | 인터넷 |
>| VIRTUAL_NETWORK | VirtualNetwork |

> [!NOTE]
> Azure 서비스의 서비스 태그는 사용되는 특정 클라우드의 주소 접두사를 나타냅니다. 예를 들어, Azure 퍼블릭 클라우드의 **Sql** 태그 값에 해당하는 기본 IP 범위는 Azure 중국 클라우드의 기본 범위와 다릅니다.

> [!NOTE]
> Azure Storage 또는 Azure SQL Database와 같은 서비스에 [가상 네트워크 서비스 엔드포인트](virtual-network-service-endpoints-overview.md)를 구현하는 경우 Azure는 서비스의 가상 네트워크 서브넷에 [경로](virtual-networks-udr-overview.md#optional-default-routes)를 추가합니다. 경로의 주소 접두사는 해당하는 서비스 태그와 동일한 주소 접두사 또는 CIDR 범위입니다.

## <a name="service-tags-on-premises"></a>온-프레미스 서비스 태그  
온-프레미스 방화벽 구성의 일부로 포함할 현재 서비스 태그 및 범위 정보를 얻을 수 있습니다. 이 정보는 각 서비스 태그에 해당하는 IP 범위의 현재 지정 시간 목록입니다. 다음 섹션에 설명된 대로 프로그래밍 방식으로 또는 JSON 파일 다운로드를 통해 이 정보를 가져올 수 있습니다.

### <a name="use-the-service-tag-discovery-api-public-preview"></a>서비스 태그 검색 API(퍼블릭 미리 보기) 사용
IP 주소 범위 세부 정보와 함께 서비스 태그의 현재 목록을 프로그래밍 방식으로 검색할 수 있습니다.

- [REST (영문)](https://docs.microsoft.com/rest/api/virtualnetwork/servicetags/list)
- [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.network/Get-AzNetworkServiceTag?view=azps-2.8.0&viewFallbackFrom=azps-2.3.2)
- [Azure CLI](https://docs.microsoft.com/cli/azure/network?view=azure-cli-latest#az-network-list-service-tags)

> [!NOTE]
> 퍼블릭 미리 보기로 제공되는 동안 검색 API는 JSON 다운로드에서 반환된 정보보다 최신이 아닌 정보를 반환할 수 있습니다. (다음 섹션을 참조하세요.)


### <a name="discover-service-tags-by-using-downloadable-json-files"></a>다운로드 가능한 JSON 파일을 사용하여 서비스 태그 검색 
현재 서비스 태그 목록이 포함된 JSON 파일을 IP 주소 범위 정보와 함께 다운로드할 수 있습니다. 이러한 목록은 매주 업데이트되고 게시됩니다. 각 클라우드의 위치는 다음과 같습니다.

- [Azure 퍼블릭](https://www.microsoft.com/download/details.aspx?id=56519)
- [Azure 미국 정부](https://www.microsoft.com/download/details.aspx?id=57063)  
- [Azure 중국](https://www.microsoft.com/download/details.aspx?id=57062) 
- [Azure 독일](https://www.microsoft.com/download/details.aspx?id=57064)   

이러한 파일의 IP 주소 범위는 CIDR 표기법으로 되어 있습니다. 

> [!NOTE]
>이 정보의 일부는 [Azure 퍼블릭](https://www.microsoft.com/download/details.aspx?id=41653), [Azure 중국](https://www.microsoft.com/download/details.aspx?id=42064) 및 [Azure 독일](https://www.microsoft.com/download/details.aspx?id=54770)용 XML 파일에 게시되었습니다. 이러한 XML 다운로드는 2020년 6월 30일부터 더 이상 지원되지 않으며 해당 날짜 이후에는 더 이상 사용할 수 없습니다. 이전 섹션에서 설명한 대로 검색 API 또는 JSON 파일 다운로드를 사용하여 마이그레이션해야 합니다.

### <a name="tips"></a>팁 
- JSON 파일에서 증가된 *changeNumber* 값을 기록하여 한 게시에서 다음 게시까지의 업데이트를 검색할 수 있습니다. 각 하위 섹션(예: **Storage.WestUS**)에는 변경이 발생할 때마다 증가하는 고유한 *changeNumber* 가 있습니다. 하위 섹션이 변경될 때 파일의 *changeNumber* 최상위 수준이 증가합니다.
- 서비스 태그 정보를 구문 분석하는 방법에 대한 예제(예: WestUS의 스토리지에 대한 모든 주소 범위 가져오기)를 보려면 [서비스 태그 검색 API PowerShell](https://aka.ms/discoveryapi_powershell) 설명서를 참조하세요.

## <a name="next-steps"></a>다음 단계
- [네트워크 보안 그룹 만들기](tutorial-filter-network-traffic.md)에 대해 알아보세요.
