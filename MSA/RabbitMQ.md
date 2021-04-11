* Persistent vs Transient
    * persistent 메시지
        * 큐에 도착하자마자 디스크에 쓰여짐
        * 가능한 한 메모리에 유지되며 메모리가 부족 상태에서만 메모리에서 제거됨
    * transient 메시지
        * 메모리가 부족 상태일 때 메모리에서 제거될 수 있도록 디스크에 기록함


영속성은 2가지 동작 방식이 있는데, 하나는 queue index 방식과 하나는 message store 이에요.
queue index 의 경우 메시지가 전달되고 승인되었는지 여부와 함께 주어진 메시지가 큐 내에서 어디에 위치하는지 관리하는데,  큐 당 하나의 큐 인덱스가 있어요.

아주 작은 메시지는 큐 인덱스에 저장하고 그 외 모든 메시지는 메시지 저장소에 작성되는데
queue_index_embed_msgs_below 로 제어 가능하고 디폴트로 직렬화된 크기가 4096 바이트 미만인 메시지 (속성 및 헤더 포함)은 큐 인덱스에 저장됩니다.


durable로 설정했으면 persistent 메시지이고. 그럼 디스크에 저장했을 테구요.  메시지 길이가 4096 바이트 미만인 메시지는 큐 인덱스로 관리 됩니다. 그런데 메모리 부족 상태에서는 디스크로 내립니다.



큐가 메모리를 확보하기 위해 메시지를 디스크로 페이지 아웃하기 시작하는 제한 비율이 있어요
vm_memory_high_watermark_paging_ratio = 0.5 이 디폴트입니다.


channel_max 값은 2047 이구요
channel_operation_timeout
default: 15000
queue_index_embed_msgs_below
큐 인덱스에 메시지가 직접 임베드되는 메시지 크기(바이트)
default: 4096
mnesia_table_loading_retry_timeout
클러스터의 mnesia 테이블이 사용 가능할 때까지 대기하는 타임아웃 시간
default: 30000


queue_master_locator => 큐 마스터 전략
* min-masters
* client-local: default
* random

* msg_store_file_size_limit
  메시지 저장소 세그먼트 파일 크기
  기존 데이터베이스가 있는 노드에서 이 값을 변경하면 데이터가 손실될 수 있어서 위험함
  default: 16777216

* RabbitMQ 노드는 최대 열린 파일 핸들 제한수에 가장 일반적으로 영향 받음
  => 대부분의 리눅스에서 디폴트 제한 값은 보통 1024 => ulimit -S -n 4096 이렇게 늘릴 수 있구요
  이 정도가 영향을 줄 수 있는 주요 conf 일 것 같네요.

=> rabbitmq docs보면 product checklist가 있어요. 거기에는 이렇게 되어 있어요
* 되도록 소비자와 발행자가 같은 노드에 연결하도록 하면 내부 노드간 트래픽을 줄일 수 있음
* 소비자가 현재 큐 마스터 노드에 연결하게 하는 것도 마찬가지로 도움이 됨


마지막으로 램노드에 대해 RabbitMQ 가이드에 이렇게 쓰여 있습니다. (정훈선임님 말이 맞아요. Ram 노드는 메타데이터를 Ram에 저장하는 것입니다. 메시지가 아니라. 메시지는 persistence message 여부에 따라 disk 저장 여부가 다르고 그건 제가 이 메신저 방에 처음 올린 내용 참고 하시면 됩니다.)
* Clusters with RAM nodes
    * RAM nodes keep their metadata only in memory.
    * However, note that since persistent queue data is always stored on disc, the performance improvements will affect only resource management (e.g. adding/removing queues, exchanges, or vhosts), but not publishing or consuming speed.
    * You should have enough disc nodes to handle your redundancy requirements, then if necessary add additional RAM nodes for scale.
        * A cluster containing only RAM nodes is fragile;
        * if the cluster stops you will not be able to start it again and will lose all data
