---
title: Event Grid 원본으로 Azure Blob Storage
description: Azure Event Grid를 사용하여 Blob Storage 이벤트에 제공되는 속성을 설명합니다.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: a914edbb6f624617766c77b277d7ee8e6ad08bd9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87458946"
---
# <a name="azure-blob-storage-as-an-event-grid-source"></a>Event Grid 원본으로 Azure Blob Storage

이 문서에서는 Blob Storage 이벤트에 대한 속성 및 스키마를 제공합니다.이벤트 스키마에 대한 소개는 [Azure Event Grid 이벤트 스키마](event-schema.md)를 참조하세요. 또한 이벤트 원본으로 Azure Blob Storage를 사용 하는 빠른 시작 및 자습서의 목록을 제공 합니다.


>[!NOTE]
> Kind **StorageV2 (범용 v2)**, **Blockblobstorage**및 **blobstorage** 의 저장소 계정만 이벤트 통합을 지원 합니다. **Storage (범용 v1)** 는 Event Grid와의 통합을 지원 *하지* 않습니다.

## <a name="event-grid-event-schema"></a>Event Grid 이벤트 스키마

### <a name="list-of-events-for-blob-rest-apis"></a>Blob REST Api에 대 한 이벤트 목록

이러한 이벤트는 클라이언트가 Blob REST Api를 호출 하 여 blob을 만들거나 바꾸거나 삭제할 때 트리거됩니다.

> [!NOTE]
> `$logs`및 `$blobchangefeed` 컨테이너는 Event Grid와 통합 되지 않으므로 이러한 컨테이너의 작업은 이벤트를 생성 하지 않습니다. 또한 *`(abfss://URI) `* 비 계층 구조 네임 스페이스 사용 계정에 대해 dfs 끝점을 사용 하면 이벤트가 생성 되지 않지만 blob 끝점은 *`(wasb:// URI)`* 이벤트를 생성 합니다.

 |이벤트 이름 |Description|
 |----------|-----------|
 |**Microsoft.Storage.BlobCreated** |Blob을 만들거나 바꿀 때 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `PutBlob` `PutBlockList` `CopyBlob` Blob REST API에서 사용할 수 있는, 또는 작업을 사용 하는 경우 트리거됩니다.   |
 |**Microsoft.Storage.BlobDeleted** |Blob이 삭제 될 때 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `DeleteBlob` Blob REST API에서 사용할 수 있는 작업을 호출 하는 경우 트리거됩니다. |

> [!NOTE]
> 블록 Blob이 완전히 커밋된 경우에만 **Microsoft. 저장소에 생성** 된 이벤트를 트리거하도록 하려면 `CopyBlob` , 및 REST API 호출에 대 한 이벤트를 필터링 `PutBlob` `PutBlockList` 합니다. 이러한 API 호출은 데이터가 블록 Blob에 완전히 커밋된 후에만 **Microsoft. 저장소로 생성** 된 이벤트를 트리거합니다. 필터를 만드는 방법에 대 한 자세한 내용은 [Event Grid에 대 한 필터 이벤트](./how-to-filter-events.md)를 참조 하세요.

### <a name="list-of-the-events-for-azure-data-lake-storage-gen-2-rest-apis"></a>Azure Data Lake Storage Gen 2 REST Api에 대 한 이벤트 목록

이러한 이벤트는 저장소 계정에서 계층적 네임 스페이스를 사용 하도록 설정 하 고 클라이언트는 Azure Data Lake Storage Gen2 REST Api를 호출 하는 경우 트리거됩니다. Bout Azure Data Lake Storage Gen2 자세한 내용은 [Azure Data Lake Storage Gen2 소개](../storage/blobs/data-lake-storage-introduction.md)를 참조 하세요.

|이벤트 이름|Description|
|----------|-----------|
|**Microsoft.Storage.BlobCreated** | Blob을 만들거나 바꿀 때 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `CreateFile` `FlushWithClose` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 및 작업을 사용 하는 경우 트리거됩니다. |
|**Microsoft.Storage.BlobDeleted** |Blob이 삭제 될 때 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `DeleteFile` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 작업을 호출 하는 경우에도 트리거됩니다. |
|**BlobRenamed**|Blob의 이름을 바꾸면 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `RenameFile` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 작업을 사용 하는 경우 트리거됩니다.|
|**Microsoft. 저장소를 만들었습니다.**|디렉터리를 만들 때 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `CreateDirectory` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 작업을 사용 하는 경우 트리거됩니다.|
|**Microsoft. 저장소 이름 바꾸기**|디렉터리의 이름을 바꾸면 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `RenameDirectory` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 작업을 사용 하는 경우 트리거됩니다.|
|**Microsoft. 저장소 삭제**|디렉터리가 삭제 되 면 트리거됩니다. <br>특히이 이벤트는 클라이언트가 `DeleteDirectory` Azure Data Lake Storage Gen2 REST API에서 사용할 수 있는 작업을 사용 하는 경우 트리거됩니다.|

> [!NOTE]
> 블록 Blob이 완전히 커밋되는 경우에만 **Microsoft. 저장소에 생성** 된 이벤트가 트리거되도록 하려면 REST API 호출에 대 한 이벤트를 필터링 `FlushWithClose` 합니다. 이 API 호출은 데이터가 블록 Blob에 완전히 커밋된 후에만 **Microsoft. Storage. BlobCreated** 이벤트를 트리거합니다. 필터를 만드는 방법에 대 한 자세한 내용은 [Event Grid에 대 한 필터 이벤트](./how-to-filter-events.md)를 참조 하세요.

<a name="example-event"></a>
### <a name="the-contents-of-an-event-response"></a>이벤트 응답 내용

이벤트가 트리거될 때 Event Grid 서비스는 해당 이벤트에 대한 데이터를 구독 엔드포인트로 보냅니다.

이 섹션에는 각 blob 저장소 이벤트에 대 한 데이터가 어떻게 표시 되는지 예가 포함 되어 있습니다.

### <a name="microsoftstorageblobcreated-event"></a>Microsoft. 저장소. BlobCreated 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/test-container/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/new-file.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobcreated-event-data-lake-storage-gen2"></a>Microsoft 저장소. BlobCreated 이벤트 (Data Lake Storage Gen2)

Blob 저장소 계정에 계층적 네임 스페이스가 있는 경우 데이터는 이러한 변경 내용을 제외 하 고 이전 예제와 유사 하 게 표시 됩니다.

* `dataVersion`키가 값으로 설정 됩니다 `2` .

* `data.api`키가 또는 문자열로 설정 된 `CreateFile` `FlushWithClose` 경우

* `contentOffset`키가 데이터 집합에 포함 됩니다.

> [!NOTE]
> 응용 프로그램에서 작업을 사용 하 여 `PutBlockList` 새 blob을 계정에 업로드 하는 경우 데이터에 이러한 변경 내용이 포함 되지 않습니다.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/new-file.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "\"0x8D4BCC2E4835CD0\"",
    "contentType": "text/plain",
    "contentLength": 0,
    "contentOffset": 0,
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/new-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event"></a>Microsoft. Storage. BlobDeleted 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/testcontainer/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-11-07T20:09:22.5674003Z",
  "id": "4c2359fe-001e-00ba-0e04-58586806d298",
  "data": {
    "api": "DeleteBlob",
    "requestId": "4c2359fe-001e-00ba-0e04-585868000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.blob.core.windows.net/testcontainer/file-to-delete.txt",
    "sequencer": "0000000000000281000000000002F5CA",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobdeleted-event-data-lake-storage-gen2"></a>Microsoft. Storage. BlobDeleted 이벤트 (Data Lake Storage Gen2)

Blob 저장소 계정에 계층적 네임 스페이스가 있는 경우 데이터는 이러한 변경 내용을 제외 하 고 이전 예제와 유사 하 게 표시 됩니다.

* `dataVersion`키가 값으로 설정 됩니다 `2` .

* `data.api`키가 문자열로 설정 됩니다 `DeleteFile` .

* `url`키에 경로가 포함 되어 있습니다 `dfs.core.windows.net` .

> [!NOTE]
> 응용 프로그램에서 작업을 사용 `DeleteBlob` 하 여 계정에서 blob을 삭제 하면 데이터에 이러한 변경 내용이 포함 되지 않습니다.

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/file-to-delete.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
    "data": {
    "api": "DeleteFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/file-to-delete.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "2",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstorageblobrenamed-event"></a>BlobRenamed 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-file.txt",
  "eventType": "Microsoft.Storage.BlobRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameFile",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-file.txt",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-file.txt",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorycreated-event"></a>Microsoft. Addresscreated 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-new-directory",
  "eventType": "Microsoft.Storage.DirectoryCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "CreateDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-new-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectoryrenamed-event"></a>Microsoft 저장소 이름 바꾸기 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/my-renamed-directory",
  "eventType": "Microsoft.Storage.DirectoryRenamed",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "RenameDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "destinationUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-renamed-directory",
    "sourceUrl": "https://my-storage-account.dfs.core.windows.net/my-file-system/my-original-directory",
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="microsoftstoragedirectorydeleted-event"></a>Microsoft. 저장소 삭제 이벤트

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/my-storage-account",
  "subject": "/blobServices/default/containers/my-file-system/blobs/directory-to-delete",
  "eventType": "Microsoft.Storage.DirectoryDeleted",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "DeleteDirectory",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "url": "https://my-storage-account.dfs.core.windows.net/my-file-system/directory-to-delete",
    "recursive": "true", 
    "sequencer": "00000000000004420000000000028963",  
    "storageDiagnostics": {
    "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="event-properties"></a>이벤트 속성

이벤트에는 다음과 같은 최상위 데이터가 있습니다.

| 속성 | 형식 | Description |
| -------- | ---- | ----------- |
| 토픽 | 문자열 | 이벤트 원본에 대한 전체 리소스 경로입니다. 이 필드는 쓸 수 없습니다. Event Grid는 이 값을 제공합니다. |
| subject | 문자열 | 게시자가 정의한 이벤트 주체의 경로입니다. |
| eventType | 문자열 | 이 이벤트 원본에 대해 등록된 이벤트 유형 중 하나입니다. |
| eventTime | 문자열 | 공급자의 UTC 시간을 기준으로 이벤트가 생성되는 시간입니다. |
| id | 문자열 | 이벤트에 대한 고유 식별자입니다. |
| 데이터 | object | Blob Storage 이벤트 데이터입니다. |
| dataVersion | 문자열 | 데이터 개체의 스키마 버전입니다. 게시자가 스키마 버전을 정의합니다. |
| metadataVersion | 문자열 | 이벤트 메타데이터의 스키마 버전입니다. Event Grid는 최상위 속성의 스키마를 정의합니다. Event Grid는 이 값을 제공합니다. |

데이터 개체의 속성은 다음과 같습니다.

| 속성 | 형식 | 설명 |
| -------- | ---- | ----------- |
| api | 문자열 | 이벤트를 트리거하는 작업입니다. |
| clientRequestId | 문자열 | 저장소 API 작업에 대 한 클라이언트 제공 요청 id입니다. 이 id는 로그의 "클라이언트-요청 id" 필드를 사용 하 여 Azure Storage 진단 로그와 상호 연결 하는 데 사용할 수 있으며, "x-y-id" 헤더를 사용 하 여 클라이언트 요청에 제공할 수 있습니다. [로그 형식](/rest/api/storageservices/storage-analytics-log-format)을 참조하세요. |
| requestId | 문자열 | 스토리지 API 작업에 대한 서비스에서 생성된 요청 ID입니다. 로그의 "request-id-header" 필드를 사용하여 Azure Storage 진단 로그와의 상관 관계를 지정하는 데 사용할 수 있으며, 'x-ms-request-id' 헤더에서 API 호출을 시작하여 반환됩니다. [로그 형식](/rest/api/storageservices/storage-analytics-log-format)을 참조하세요. |
| eTag | 문자열 | 조건부로 작업을 수행하는 데 사용할 수 있는 값입니다. |
| contentType | 문자열 | Blob에 대해 지정된 콘텐츠 형식입니다. |
| contentLength | 정수 | Blob의 크기(바이트)입니다. |
| blobType | 문자열 | Blob의 형식입니다. 유효한 값은 "BlockBlob" 또는 "PageBlob"입니다. |
| contentOffset | number | 이벤트 트리거 응용 프로그램에서 파일에 쓰기를 완료 한 시점에 수행 된 쓰기 작업의 오프셋 (바이트)입니다. <br>계층 네임 스페이스가 있는 blob storage 계정에서 트리거되는 이벤트에 대해서만 나타납니다.|
| destinationUrl |문자열 | 작업이 완료 된 후 존재 하는 파일의 url입니다. 예를 들어 파일의 이름을 바꾸면 `destinationUrl` 속성에 새 파일 이름의 url이 포함 됩니다. <br>계층 네임 스페이스가 있는 blob storage 계정에서 트리거되는 이벤트에 대해서만 나타납니다.|
| sourceUrl |문자열 | 작업 이전에 존재 하는 파일의 url입니다. 예를 들어 파일의 이름을 바꾸면 `sourceUrl` 이름 바꾸기 작업 이전의 원본 파일 이름 url이 포함 됩니다. <br>계층 네임 스페이스가 있는 blob storage 계정에서 트리거되는 이벤트에 대해서만 나타납니다. |
| url | 문자열 | Blob에 대한 경로입니다. <br>클라이언트에서 REST API Blob을 사용 하는 경우 url의 구조는 * \<storage-account-name\> . blob.core.windows.net/ \<container-name\> / \<file-name\> *입니다. <br>클라이언트에서 Data Lake Storage REST API를 사용 하는 경우 url의 구조는 * \<storage-account-name\> . dfs.core.windows.net/ \<file-system-name\> / \<file-name\> *입니다. |
| recursive | 문자열 | `True` 모든 자식 디렉터리에서 작업을 수행 하려면 그렇지 않으면 `False` 입니다. <br>계층 네임 스페이스가 있는 blob storage 계정에서 트리거되는 이벤트에 대해서만 나타납니다. |
| sequencer | 문자열 | 특정 Blob 이름에 대한 이벤트의 논리적 순서를 나타내는 불투명 문자열 값입니다.  사용자는 표준 문자열 비교를 사용하여 동일한 Blob 이름에 대한 두 이벤트의 상대적 순서를 이해할 수 있습니다. |
| storageDiagnostics | object | 경우에 따라 Azure Storage 서비스에 의해 포함되는 진단 데이터입니다. 포함될 경우, 이벤트 소비자는 무시해야 합니다. |

## <a name="tutorials-and-how-tos"></a>자습서 및 방법
|제목  |설명  |
|---------|---------|
| [빠른 시작: Azure CLI를 사용하여 Blob Storage 이벤트를 사용자 지정 웹 엔드포인트로 라우팅](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Azure CLI를 사용하여 Blob Storage 이벤트를 WebHook로 전송하는 방법을 보여줍니다. |
| [빠른 시작: PowerShell을 사용하여 Blob Storage 이벤트를 사용자 지정 웹 엔드포인트로 라우팅](../storage/blobs/storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Azure PowerShell을 사용하여 Blob Storage 이벤트를 WebHook로 전송하는 방법을 보여줍니다. |
| [빠른 시작: Azure Portal을 사용하여 Blob Storage 이벤트 만들기 및 라우팅](blob-event-quickstart-portal.md) | 포털를 사용하여 Blob Storage 이벤트를 WebHook로 전송하는 방법을 보여줍니다. |
| [Azure CLI: Blob Storage 계정에 대한 이벤트 구독](./scripts/event-grid-cli-blob.md) | Blob Storage 계정에 대한 이벤트를 구독하는 샘플 스크립트입니다. 이벤트를 WebHook로 전송합니다. |
| [PowerShell: Blob Storage 계정에 대한 이벤트 구독](./scripts/event-grid-powershell-blob.md) | Blob Storage 계정에 대한 이벤트를 구독하는 샘플 스크립트입니다. 이벤트를 WebHook로 전송합니다. |
| [Resource Manager 템플릿: Blob Storage 및 구독 만들기](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage) | Azure Blob Storage 계정을 배포하고 해당 스토리지 계정에 대한 이벤트를 구독합니다. WebHook에 이벤트를 보냅니다. |
| [개요: Blob Storage 이벤트에 대응](../storage/blobs/storage-blob-event-overview.md) | Event Grid와 Blob Storage 통합의 개요입니다. |

## <a name="next-steps"></a>다음 단계

* Azure Event Grid에 대한 소개는 [Event Grid란?](overview.md)을 참조하세요.
* Azure Event Grid 구독을 만드는 방법에 대한 자세한 내용은 [Event Grid 구독 스키마](subscription-creation-schema.md)를 참조하세요.
* Blob Storage 이벤트를 사용하는 방법에 대한 소개는 [Blob Storage 이벤트 라우팅 - Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)를 참조하세요. 
