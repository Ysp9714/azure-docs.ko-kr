---
title: 조밀한 탄력적 풀의 리소스 관리
description: 많은 데이터베이스를 포함 하는 Azure SQL Database 탄력적 풀에서 계산 리소스를 관리 합니다.
services: sql-database
ms.service: sql-database
ms.subservice: elastic-pools
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: dimitri-furman
ms.author: dfurman
ms.reviewer: sstein
ms.date: 09/16/2020
ms.openlocfilehash: 1398745a485b57c3a739c25dbca77d7b21f19758
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792160"
---
# <a name="resource-management-in-dense-elastic-pools"></a>조밀한 탄력적 풀의 리소스 관리
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database [탄력적 풀](./elastic-pool-overview.md) 은 다양 한 리소스 사용으로 많은 데이터베이스를 관리 하는 비용 효율적인 솔루션입니다. 탄력적 풀의 모든 데이터베이스는 **지정 된 시간에 풀에 있는 데이터베이스의 하위 집합만 계산 리소스를 사용** 한다는 가정 하에 CPU, 메모리, 작업자 스레드, 저장소 공간, tempdb 등의 동일한 리소스 할당을 공유 합니다. 이러한 가정을 통해 탄력적 풀을 비용 효율적으로 사용할 수 있습니다. 각 개별 데이터베이스에 필요할 수 있는 모든 리소스에 대해 지불 하는 대신, 고객은 풀의 모든 데이터베이스 간에 공유 되는 훨씬 더 작은 리소스 집합을 지불 합니다.

## <a name="resource-governance"></a>리소스 거버넌스

리소스를 공유 하려면 시스템에서 리소스 사용을 신중 하 게 제어 하 여 리소스 사용량이 많은 데이터베이스가 동일한 탄력적 풀의 다른 데이터베이스에 영향을 주는 "잡음이 많은 환경" 효과를 최소화 해야 합니다. 동시에 시스템은 고가용성 및 재해 복구 (HADR), 백업 및 복원, 모니터링, 쿼리 저장소, 자동 조정 등과 같은 기능에 대 한 충분 한 리소스를 제공 하 여 안정적으로 작동 해야 합니다.

Azure SQL Database는 프로세스 수준 리소스 관리를 위한 windows [작업 개체](/windows/win32/procthread/job-objects) , 저장소 할당량 관리를 위한 windows [FSRM (파일 서버 리소스 관리자)](/windows-server/storage/fsrm/fsrm-overview) , SQL Database 내에서 리소스 관리를 구현 하는 SQL Server [Resource Governor](/sql/relational-databases/resource-governor/resource-governor) 수정 및 확장 된 버전을 포함 하 여 여러 리소스 거 버 넌 스 메커니즘을 사용 하 여 이러한 목표를 달성 합니다.

탄력적 풀의 기본 디자인 목표는 비용 효율적입니다. 이러한 이유로 시스템은 의도적으로 고객에 게 적절 한 수준의 데이터베이스를 사용 하는 풀 인 _조밀한_ 풀을 만들 수 있도록 합니다. 이와 같은 이유로 시스템은 내부 프로세스에 필요한 모든 리소스를 예약 하지는 않지만 내부 프로세스와 사용자 작업 간에 리소스를 공유할 수 있습니다.

이 접근 방식을 통해 고객은 조밀한 탄력적 풀을 사용 하 여 적절 한 성능 및 주요 비용 절감을 달성할 수 있습니다. 그러나 조밀한 풀의 여러 데이터베이스에 대 한 워크 로드가 충분히 높은 경우 리소스 경합이 매우 커집니다. 리소스 경합은 사용자 워크 로드 성능을 줄이며 내부 프로세스에 부정적인 영향을 줄 수 있습니다.

> [!IMPORTANT]
> 활성 데이터베이스가 많은 집적형 풀에서는 풀의 데이터베이스 수를 [DTU](resource-limits-dtu-elastic-pools.md) 및 [vcore](resource-limits-vcore-elastic-pools.md) 탄력적 풀에 대해 문서화 된 최대값까지 늘리는 것이 불가능할 수 있습니다.
>
> 리소스 경합 및 성능 문제를 발생 시 키 지 않고 조밀한 풀에 배치할 수 있는 데이터베이스 수는 동시 활성 데이터베이스 수 및 각 데이터베이스의 사용자 작업에의 한 리소스 사용에 따라 달라 집니다. 이 수는 사용자 작업이 변경 됨에 따라 시간이 지남에 따라 변경 될 수 있습니다.

조밀 압축 된 풀에서 리소스 경합이 발생 하는 경우 고객은 다음 작업 중 하나 이상을 선택 하 여이를 완화할 수 있습니다.

- 쿼리 작업을 조정 하 여 리소스 사용량을 줄이거나 시간이 지남에 따라 여러 데이터베이스에 걸쳐 리소스 소비를 분산 합니다.
- 일부 데이터베이스를 다른 풀로 이동 하거나 독립 실행형 데이터베이스를 만들어 풀 밀도를 줄입니다.
- 풀을 확장 하 여 더 많은 리소스를 얻습니다.

마지막 두 작업을 구현 하는 방법에 대 한 제안은이 문서의 뒷부분에 나오는 [운영 권장 사항](#operational-recommendations) 을 참조 하세요. 리소스 경합을 줄이면 사용자 작업 및 내부 프로세스의 이점을 모두 활용할 수 있으며, 시스템에서 필요한 서비스 수준을 안정적으로 유지할 수 있습니다.

## <a name="monitoring-resource-consumption"></a>리소스 소비 모니터링

리소스 경합으로 인 한 성능 저하를 방지 하려면 조밀한 탄력적 풀을 사용 하는 고객은 리소스 소비를 미리 모니터링 하 고 증가 하는 리소스 경합이 작업에 영향을 주는 경우 시기 적절 한 조치를 취합니다. 연속 모니터링은 사용자 작업의 변경, 데이터 볼륨 및 배포의 변경 내용, 풀 밀도의 변경 내용 및 Azure SQL Database 서비스의 변경으로 인해 시간이 지남에 따라 풀의 리소스 사용량이 변경 되기 때문에 중요 합니다.

Azure SQL Database은 이러한 유형의 모니터링과 관련 된 몇 가지 메트릭을 제공 합니다. 각 메트릭에 대해 권장 되는 평균 값을 초과 하면 풀의 리소스 경합이 표시 되며 앞에서 설명한 작업 중 하나를 사용 하 여 주소를 지정 해야 합니다.

|메트릭 이름|Description|권장 평균 값|
|----------|--------------------------------|------------|
|`avg_instance_cpu_percent`|기본 운영 체제에 따라 측정 된 탄력적 풀과 연결 된 SQL 프로세스의 CPU 사용률입니다. 모든 데이터베이스의 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 보기와 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `sqlserver_process_core_percent` Azure Portal에서 볼 수 있습니다. 이 값은 동일한 탄력적 풀의 모든 데이터베이스에 대해 동일 합니다.|70% 미만. 때때로 최대 90%까지 짧은 급증이 허용 될 수 있습니다.|
|`max_worker_percent`|[작업자 스레드]( https://docs.microsoft.com/sql/relational-databases/thread-and-task-architecture-guide) 사용률입니다. 풀 자체 뿐만 아니라 풀의 각 데이터베이스에 대해 제공 됩니다. 데이터베이스 수준에서 작업자 스레드 수에 대 한 제한은 다르며 풀 수준에서는 두 수준 모두에서이 메트릭을 모니터링 하는 것이 좋습니다. 모든 데이터베이스의 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 보기와 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `workers_percent` Azure Portal에서 볼 수 있습니다.|80% 미만. 최대 100%까지 증가 하면 연결 시도 및 쿼리가 실패 합니다.|
|`avg_data_io_percent`|읽기 및 쓰기 물리적 IO에 대 한 IOPS 사용률입니다. 풀 자체 뿐만 아니라 풀의 각 데이터베이스에 대해 제공 됩니다. 데이터베이스 수준의 IOPS 수에는 다른 제한이 있으므로 풀 수준에서는 두 수준에서이 메트릭을 모니터링 하는 것이 좋습니다. 모든 데이터베이스의 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 보기와 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `physical_data_read_percent` Azure Portal에서 볼 수 있습니다.|80% 미만. 때때로 최대 100%까지 짧은 급증이 허용 될 수 있습니다.|
|`avg_log_write_percent`|트랜잭션 로그 쓰기 IO의 처리량 사용률입니다. 풀 자체 뿐만 아니라 풀의 각 데이터베이스에 대해 제공 됩니다. 데이터베이스 수준에서 로그 처리량에는 여러 제한이 있으므로 풀 수준에서는 두 수준에서이 메트릭을 모니터링 하는 것이 좋습니다. 모든 데이터베이스의 [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) 보기와 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `log_write_percent` Azure Portal에서 볼 수 있습니다. 이 메트릭이 100%에 가까운 경우 모든 데이터베이스 수정 (INSERT, UPDATE, DELETE, MERGE 문, SELECT ... INTO, BULK INSERT 등) 속도가 느려집니다.|90% 미만. 때때로 최대 100%까지 짧은 급증이 허용 될 수 있습니다.|
|`oom_per_second`|탄력적 풀에서의 OOM (메모리 부족) 오류 (메모리 압력 표시기)입니다. [Sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database) 뷰에서 사용할 수 있습니다. 이 메트릭을 계산 하는 예제 쿼리는 [예제](#examples) 를 참조 하세요.|0|
|`avg_storage_percent`|탄력적 풀 내의 모든 데이터베이스에서 데이터에 사용 되는 총 저장소 공간입니다. 데이터베이스 파일에 빈 공간을 포함 하지 않습니다. 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `storage_percent` Azure Portal에서 볼 수 있습니다.|80% 미만. 데이터 증가가 없는 풀에 대해 100%의 접근 방식을 사용할 수 있습니다.|
|`avg_allocated_storage_percent`|탄력적 풀 내의 모든 데이터베이스에서 저장소의 데이터베이스 파일에 사용 되는 총 저장소 공간입니다. 데이터베이스 파일에 빈 공간을 포함 합니다. 데이터베이스의 [sys.elastic_pool_resource_stats](/sql/relational-databases/system-catalog-views/sys-elastic-pool-resource-stats-azure-sql-database) 뷰에서 사용할 수 있습니다 `master` . 이 메트릭은 [명명](../../azure-monitor/platform/metrics-supported.md#microsoftsqlserverselasticpools) 된 Azure Monitor로 내보내지고 `allocated_data_storage_percent` Azure Portal에서 볼 수 있습니다.|90% 미만. 데이터 증가가 없는 풀에 대해 100%의 접근 방식을 사용할 수 있습니다.|
|`tempdb_log_used_percent`|데이터베이스의 트랜잭션 로그 공간 사용률 `tempdb` 입니다. 한 데이터베이스에서 만든 임시 개체가 동일한 탄력적 풀의 다른 데이터베이스에 표시 되지 않더라도 `tempdb` 는 동일한 풀에 있는 모든 데이터베이스에 대 한 공유 리소스입니다. `tempdb`풀에 있는 한 데이터베이스에서 시작 된 오래 실행 되거나 분리 된 트랜잭션은 트랜잭션 로그의 상당 부분을 사용할 수 있으며 동일한 풀에 있는 다른 데이터베이스의 쿼리에 오류가 발생 합니다. [Sys.dm_db_log_space_usage](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-log-space-usage-transact-sql) 및 [sys.database_files](/sql/relational-databases/system-catalog-views/sys-database-files-transact-sql) 뷰에서 파생 됩니다. 이 메트릭은 Azure Monitor로 내보내지고 Azure Portal에서 볼 수 있습니다. 이 메트릭의 현재 값을 반환 하는 예제 쿼리는 [예제](#examples) 를 참조 하세요.|50% 미만. 간혹 최대 80%까지 급증 하는 것은 허용 됩니다.|
|||

이러한 메트릭 외에도 Azure SQL Database는 실제 리소스 거 버 넌 스 제한을 반환 하는 뷰와 리소스 풀 수준에서 리소스 사용률 통계를 반환 하는 추가 뷰 및 작업 그룹 수준을 제공 합니다.

|뷰 이름|Description|  
|-----------------|--------------------------------|  
|[sys.dm_user_db_resource_governance](/sql/relational-databases/system-dynamic-management-views/sys-dm-user-db-resource-governor-azure-sql-database)|현재 데이터베이스 또는 탄력적 풀의 리소스 거 버 넌 스 메커니즘에서 사용 되는 실제 구성 및 용량 설정을 반환 합니다.|
|[sys.dm_resource_governor_resource_pools](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-transact-sql)|현재 리소스 풀 상태, 리소스 풀의 현재 구성 및 누적 리소스 풀 통계에 대 한 정보를 반환 합니다.|
|[sys.dm_resource_governor_workload_groups](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-transact-sql)|작업 그룹의 누적 작업 그룹 통계 및 현재 구성을 반환 합니다. 이 뷰를 열에서 sys.dm_resource_governor_resource_pools와 조인 `pool_id` 하 여 리소스 풀 정보를 가져올 수 있습니다.|
|[sys.dm_resource_governor_resource_pools_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-resource-pools-history-ex-azure-sql-database)|최근 32 분에 대 한 리소스 풀 사용률 통계를 반환 합니다. 각 행은 20 초 간격을 나타냅니다. `delta_`열은 간격 동안의 각 통계에 대 한 변경 내용을 반환 합니다.|
|[sys.dm_resource_governor_workload_groups_history_ex](/sql/relational-databases/system-dynamic-management-views/sys-dm-resource-governor-workload-groups-history-ex-azure-sql-database)|최근 32 분의 작업 그룹 사용률 통계를 반환 합니다. 각 행은 20 초 간격을 나타냅니다. `delta_`열은 간격 동안의 각 통계에 대 한 변경 내용을 반환 합니다.|
|||

이러한 뷰를 사용 하 여 리소스 사용률을 모니터링 하 고 리소스 경합 문제를 거의 실시간으로 해결할 수 있습니다. 지역 복제본을 비롯 하 여 읽기 가능한 주 보조 복제본의 사용자 작업은 `SloSharedPool1` 리소스 풀 및 작업 그룹으로 분류 됩니다 `UserPrimaryGroup.DBId[N]` . 여기서는 `N` 데이터베이스 ID 값을 나타냅니다.

현재 리소스 사용률을 모니터링 하는 것 외에도 조밀한 풀을 사용 하는 고객은 별도의 데이터 저장소에서 기록 리소스 사용률 데이터를 유지할 수 있습니다. 이 데이터는 예측 분석에서 기록 및 계절 추세에 따라 리소스 사용률을 사전에 관리 하는 데 사용할 수 있습니다.

## <a name="operational-recommendations"></a>운영 권장 사항

**리소스 여유 공간** 을 확보 하세요. 리소스 경합 및 성능 저하가 발생 하는 경우에는 이전에 설명한 대로 영향을 받는 탄력적 풀에서 일부 데이터베이스를 이동 하거나 풀을 확장 해야 할 수 있습니다. 그러나 이러한 작업을 완료 하려면 추가 계산 리소스가 필요 합니다. 특히 Premium 및 중요 비즈니스용 풀의 경우 이러한 작업을 수행 하려면 이동 중인 데이터베이스의 모든 데이터 또는 풀이 확장 된 경우 탄력적 풀에 있는 모든 데이터베이스에 대 한 모든 데이터를 전송 해야 합니다. 데이터 전송은 장기 실행 되 고 리소스를 많이 사용 하는 작업입니다. 풀이 이미 높은 리소스 압력 상태에 있는 경우 완화 작업 자체가 성능을 저하 시킵니다. 극단적인 경우에는 필요한 리소스를 사용할 수 없기 때문에 데이터베이스 이동 또는 풀 확장을 통해 리소스 경합을 해결할 수 없습니다. 이 경우 영향을 받는 탄력적 풀에 대 한 쿼리 작업을 일시적으로 줄이는 것이 유일한 해결책이 될 수 있습니다.

조밀한 풀을 사용 하는 고객은 앞에서 설명한 대로 리소스 사용률 추세를 면밀 하 게 모니터링 하 고 메트릭이 권장 범위 내에 유지 되 고 탄력적 풀에 충분 한 리소스가 남아 있는 동안 완화 작업을 수행 해야 합니다.

리소스 사용률은 각 데이터베이스와 각 탄력적 풀에 대해 시간이 지남에 따라 변경 되는 여러 요소에 따라 다릅니다. 조밀한 풀에서 최적의 가격/성능 비율을 달성 하려면 지속적으로 모니터링 하 고 균형을 조정 해야 합니다 .이는 데이터베이스를 더 많이 사용 되는 풀에서 더 적게 사용 하는 풀로 이동 하 고 증가 된 워크 로드를 수용 하기 위해

**"핫" 데이터베이스를 이동 하지** 않습니다. 풀 수준의 리소스 경합이 주로 적은 수의 과도 한 데이터베이스를 사용 하 여 발생 하는 경우 이러한 데이터베이스를 더 적게 사용 되는 풀로 이동 하거나 독립 실행형 데이터베이스를 만드는 것이 좋습니다. 그러나 데이터베이스를 과도 하 게 사용 하는 동안에는이 작업을 수행 하지 않는 것이 좋습니다. 이동 작업은 이동 하는 데이터베이스와 전체 풀에 대해 성능이 더 저하 되기 때문입니다. 대신 높은 사용률 하위 측까지 기다리거나, 사용 하지 않는 데이터베이스를 풀 수준에서 리소스 압력을 완화 하기 위해 더 적게 이동 합니다. 하지만 사용률이 매우 낮은 데이터베이스를 이동 하는 것은 풀 수준에서 리소스 사용률을 별개인 하지 않기 때문에이 경우 어떠한 이점도 제공 하지 않습니다.

**"격리" 풀에 새 데이터베이스를 만듭니다** . 데이터베이스 당 테 넌 트 모델을 사용 하는 응용 프로그램과 같이 새 데이터베이스가 자주 만들어지는 시나리오에서 기존 탄력적 풀에 배치 되는 새 데이터베이스가 예기치 않은 리소스를 사용 하 고 풀의 다른 데이터베이스 및 내부 프로세스에 영향을 줄 수 있습니다. 이러한 위험을 완화 하려면 리소스를 충분 하 게 할당 하 여 별도의 "격리" 풀을 만듭니다. 아직 알 수 없는 리소스 소비 패턴의 새 데이터베이스에이 풀을 사용 합니다. 데이터베이스가이 풀에서 주 또는 월과 같은 비즈니스 주기에 대해 높게 유지 되 고 해당 리소스 소비가 알려진 경우이 추가 리소스 사용량을 수용할 수 있는 충분 한 용량이 있는 풀로 이동할 수 있습니다.

**사용 된 공간 및 할당 된 공간을 모두 모니터링** 합니다. 할당 된 풀 공간 (풀의 모든 데이터베이스에 대 한 저장소의 모든 데이터베이스 파일의 전체 크기)이 최대 풀 크기에 도달 하면 공간 부족 오류가 발생할 수 있습니다. 할당 된 공간 추세가 많고 최대 풀 크기에 도달 하기 위해 추적 되는 경우 완화 옵션은 다음과 같습니다.
- 일부 데이터베이스를 풀 외부로 이동 하 여 할당 된 총 공간 줄이기
- 파일의 빈 할당 된 공간을 줄이기 위해 데이터베이스 파일 [축소](file-space-manage.md)
- 풀 크기가 더 큰 서비스 목표로 풀 확장

풀 공간 (풀의 모든 데이터베이스에 있는 데이터의 전체 크기, 파일에 빈 공간을 포함 하지 않음) 추세를 사용 하 고 최대 풀 크기에 도달 하는 추적을 사용 하는 경우 완화 옵션은 다음과 같습니다.
- 사용 중인 전체 공간을 줄이기 위해 일부 데이터베이스를 풀 외부로 이동
- 데이터베이스 외부에서 데이터 이동 (보관) 또는 더 이상 필요 없는 데이터 삭제
- [데이터 압축](/sql/relational-databases/data-compression/data-compression) 구현
- 풀 크기가 더 큰 서비스 목표로 풀 확장

**지나치게 조밀한 서버를 사용 하지 않습니다** . Azure SQL Database는 서버당 최대 5000 개의 데이터베이스를 [지원](./resource-limits-logical-server.md) 합니다. 수천 개의 데이터베이스에서 탄력적 풀을 사용 하는 고객은 단일 서버에 여러 탄력적 풀을 배치 하는 것이 좋습니다 .이 경우 총 데이터베이스 수가 지원 되는 제한까지 포함 됩니다. 그러나 수천 개의 데이터베이스가 있는 서버는 운영 문제를 만듭니다. 서버의 모든 데이터베이스를 열거 해야 하는 작업 (예: 포털의 데이터베이스 보기)은 더 느려집니다. 서버 수준 로그인 또는 방화벽 규칙의 잘못 된 수정과 같은 운영 오류는 더 많은 데이터베이스에 영향을 줍니다. 서버를 실수로 삭제 하는 경우 삭제 된 서버에서 데이터베이스를 복구 하는 Microsoft 지원에 대 한 지원이 필요 하며, 영향을 받는 모든 데이터베이스의 작동이 중단 될 수 있습니다.

서버당 데이터베이스 수를 지원 되는 최대 개수 보다 낮게 제한 하는 것이 좋습니다. 대부분의 시나리오에서 서버당 최대 1000-2000 개의 데이터베이스를 사용 하는 것이 가장 좋습니다. 실수로 서버를 삭제 하는 가능성을 줄이려면 서버 또는 해당 리소스 그룹에 [삭제 잠금을](../../azure-resource-manager/management/lock-resources.md) 배치 하는 것이 좋습니다.

과거에는 동일한 서버의 탄력적 풀에서 데이터베이스를 이동 하는 것과 관련 된 특정 시나리오를 서버 간에 데이터베이스를 이동할 때 보다 더 빠릅니다. 현재 모든 데이터베이스는 원본 서버와 대상 서버에 관계 없이 동일한 속도로 실행 됩니다.

## <a name="examples"></a>예

### <a name="monitoring-memory-utilization"></a>메모리 사용률 모니터링

이 쿼리 `oom_per_second` 는 지난 32 분 동안 각 리소스 풀의 메트릭을 계산 합니다. 이 쿼리는 탄력적 풀의 모든 데이터베이스에서 실행할 수 있습니다.

```sql
SELECT pool_id,
       name AS resource_pool_name,
       IIF(name LIKE 'SloSharedPool%' OR name LIKE 'UserPool%', 'user', 'system') AS resource_pool_type,
       SUM(CAST(delta_out_of_memory_count AS decimal))/(SUM(duration_ms)/1000.) AS oom_per_second
FROM sys.dm_resource_governor_resource_pools_history_ex
GROUP BY pool_id, name
ORDER BY pool_id;
```

### <a name="monitoring-tempdb-log-space-utilization"></a>`tempdb`로그 공간 사용률 모니터링

이 쿼리는 `tempdb_log_used_percent` `tempdb` 최대 허용 크기를 기준으로 트랜잭션 로그의 상대 사용률을 보여 주는 메트릭의 현재 값을 반환 합니다. 이 쿼리는 탄력적 풀의 모든 데이터베이스에서 실행할 수 있습니다.

```sql
SELECT (lsu.used_log_space_in_bytes / df.log_max_size_bytes) * 100 AS tempdb_log_space_used_percent
FROM tempdb.sys.dm_db_log_space_usage AS lsu
CROSS JOIN (
           SELECT SUM(CAST(max_size AS bigint)) * 8 * 1024. AS log_max_size_bytes
           FROM tempdb.sys.database_files
           WHERE type_desc = N'LOG'
           ) AS df
;
```

## <a name="next-steps"></a>다음 단계

- 탄력적 풀에 대 한 소개는 [Azure SQL Database에서 여러 데이터베이스를 관리 하 고 크기를 조정 하는 데 도움이 되는 탄력적 풀](./elastic-pool-overview.md)을 참조 하세요.
- 리소스 사용률을 줄이기 위해 쿼리 작업을 튜닝 하는 방법에 대 한 자세한 내용은 [모니터링 및 튜닝]( https://docs.microsoft.com/azure/sql-database/sql-database-monitoring-tuning-index), [모니터링 및 성능 튜닝](./monitor-tune-overview.md)을 참조 하세요.