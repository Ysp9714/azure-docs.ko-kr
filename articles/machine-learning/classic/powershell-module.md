---
title: 'ML Studio (클래식): PowerShell 모듈-Azure'
description: PowerShell을 사용 하 여 Azure Machine Learning Studio (클래식) 작업 영역, 실험, 웹 서비스 등을 만들고 관리할 수 있습니다.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
author: likebupt
ms.author: keli19
ms.date: 04/25/2019
ms.openlocfilehash: f66363ec8b64dea1a076f81f4fc89bafe5ca4151
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322324"
---
# <a name="powershell-modules-for-azure-machine-learning-studio-classic"></a>Azure Machine Learning Studio 용 PowerShell 모듈 (클래식)

**적용 대상:**  ![적용 대상:](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio(클래식)  ![적용되지 않는 대상: ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


PowerShell 모듈을 사용 하 여 작업 영역, 데이터 집합 및 웹 서비스와 같은 사용자의 Studio (클래식) 리소스 및 자산을 프로그래밍 방식으로 관리할 수 있습니다.

세 가지 PowerShell 모듈을 사용 하 여 Studio (클래식) 리소스와 상호 작용할 수 있습니다.

* [Azure PowerShell Az](#az-rm)(2018년 출시) - cmdlet 이름은 다르지만 AzureRM의 모든 기능이 포함되어 있습니다.
* 2016에서 릴리스된 [AzureRM](#az-rm) PowerShell Az로 바뀜
* [Azure Machine Learning PowerShell 클래식](#classic)(2016년 출시)

이러한 PowerShell 모듈에는 몇 가지 유사점이 있지만 각각은 특정 시나리오용으로 설계 되었습니다. 이 문서에서는 PowerShelll 모듈 간의 차이점을 설명하고 선택해야 하는 모듈을 결정하는 데 도움이 되는 정보를 제공합니다.  

아래의 [지원 표](#support-table)에서 각 모듈이 지원하는 리소스를 확인하세요. 

## <a name="azure-powershell-az-and-azurerm"></a><a name="az-rm"></a> Azure PowerShell Az 및 AzureRM

현재 Azure 조작용 PowerShell 모듈로 사용되고 있는 Az는 AzureRM의 모든 이전 기능을 포함합니다. AzureRM에도 버그 수정은 계속 제공되지만 신규 cmdlet이나 기능은 제공되지 않습니다.  Az와 AzureRM은 모두 **Azure Resource Manager** 배포 모델을 사용하여 배포된 솔루션을 관리합니다. 이러한 리소스에는 Studio (클래식) 작업 영역 및 Studio (클래식) "새" 웹 서비스가 포함 됩니다. 

PowerShell 클래식은 Az 또는 AzureRM와 함께 설치 하 여 "새" 및 "클래식" 리소스 유형을 모두 포함할 수 있습니다. 그러나 Az와 AzureRM은 동시에 설치하지 않는 것이 좋습니다. Az와 AzureRM 중에 설치할 모듈을 결정해야 하는 경우 향후 모든 배포에 사용할 수 있는 Az를 선택하는 것이 좋습니다.  Az와 AzureRM에 대 한 자세한 내용과 [Azure PowerShell az를 소개 하](/powershell/azure/new-azureps-module-az)는 마이그레이션 경로에 대해 자세히 알아보세요.

Az 사용을 시작하려면 [Azure Az 설치 지침](/powershell/azure/install-az-ps)을 따르세요.

## <a name="powershell-classic"></a><a name="classic"></a> PowerShell 클래식

Studio (클래식) [PowerShell 클래식 모듈](https://aka.ms/amlps) 을 사용 하면 **클래식 배포 모델** 을 사용 하 여 배포 된 리소스를 관리할 수 있습니다. 이러한 리소스에는 Studio (클래식) 사용자 자산, "클래식" 웹 서비스 및 "클래식" 웹 서비스 끝점이 포함 됩니다.

그러나 리소스의 배포 및 관리를 간소화 하기 위해 이후의 모든 리소스에 리소스 관리자 배포 모델을 사용 하는 것이 좋습니다. 배포 모델에 대해 자세히 알아보려면 [Azure Resource Manager와 클래식 배포 비교](../../azure-resource-manager/management/deployment-models.md)를 참조하세요.

PowerShell 클래식 사용을 시작하려면 GitHub에서 [릴리스 패키지](https://github.com/hning86/azuremlps/releases)를 다운로드하고 [설치 지침](https://github.com/hning86/azuremlps/blob/master/README.md)을 따릅니다. 지침은 다운로드/압축 해제된 DLL의 차단을 해제한 다음 PowerShell 환경으로 가져오는 방법을 설명합니다.

PowerShell 클래식은 Az 또는 AzureRM와 함께 설치 하 여 "새" 및 "클래식" 리소스 유형을 모두 포함할 수 있습니다.

## <a name="powershell-support-table"></a><a name="support-table"></a> PowerShell 지원 표


| Task | **Az** |  **PowerShell 클래식** |
| --- | --- | --- |
| 작업 영역 만들기/삭제 | [리소스 관리자 템플릿](./deploy-with-resource-manager-template.md) |  |
| 작업 영역 약정 계획 관리 | [New-AzMlCommitmentPlan](/powershell/module/az.machinelearning/new-azmlcommitmentplan) | |
| 작업 영역 사용자 관리 |  | [Add-AmlWorkspaceUsers](https://github.com/hning86/azuremlps#add-amlworkspaceusers)|
| 웹 서비스 관리 | [New-AzMlWebService](/powershell/module/az.machinelearning/new-azmlwebservice) <br>("새" 웹 서비스)| [New-AmlWebService](https://github.com/hning86/azuremlps#manage-classic-web-service) <br>("클래식" 웹 서비스) |
| 웹 서비스 끝점/키 관리 |  [AzMlWebServiceKey](/powershell/module/az.machinelearning/get-azmlwebservicekey)|  [Add-AmlWebServiceEndpoint](https://github.com/hning86/azuremlps#manage-classic-web-servcie-endpoint)|
| 사용자 데이터 집합/학습 된 모델 관리| | [Get-AmlDataset](https://github.com/hning86/azuremlps#manage-user-assets-dataset-trained-model-transform) |
| 사용자 실험 관리 |  | [Start-AmlExperiment](https://github.com/hning86/azuremlps#manage-experiment) |
| 사용자 지정 모듈 관리 | | [New-AmlCustomModule](https://github.com/hning86/azuremlps#manage-custom-module) |


## <a name="next-steps"></a>다음 단계
다음 PowerShell 모듈에 대 한 전체 설명서를 참조 하세요.
* [PowerShell 클래식](https://aka.ms/amlps)
* [Azure PowerShell Az](/powershell/module/az.machinelearning/#machine_learning)