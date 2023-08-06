# DBMS

목차 

1. dbms 구조
2. 디스크 스페이스 매니저 계층
3. 인덱스 매니저 계층
4. 버퍼 매니저 계층
5. 동시성 관리
6. 장애 회복

### DBMS 구조

![Untitled](https://github.com/baryu97/DB/assets/74360026/6538def8-2235-4bd0-a98d-5693926ef69c)

DBMS의 설계는 크게 세 계층으로 구성되어 있으며, 각 계층은 API를 통해 접근하게 설계되었습니다.

첫 번째 계층인 Disk Space Manager는 디스크 상의 데이터를 4KB 단위의 페이지로 읽고 씁니다. 이 계층은 상위 계층이 데이터 저장을 위한 공간을 요청하면 페이지를 할당하고, 필요하지 않을 때는 페이지를 할당 해제하는 역할을 담당합니다. 이를 통해 효율적인 디스크 공간 관리를 지원합니다.

두 번째 계층인 Buffer Manager는 디스크의 입출력(I/O)을 줄이기 위해 디스크의 데이터를 메모리에 저장합니다. 이를 통해 DB의 데이터를 메모리에서 직접 가져와 사용할 수 있게 합니다. 

세 번째 계층인 File and Index Manager는 B+ tree를 활용하여 key 값에 따라 데이터의 저장 위치를 계산하고, 필요한 데이터를 효율적으로 찾을 수 있게 합니다.

사용자가 File and Index Manager을 통해 데이터를 찾으려고 하면, File and Index Manager은 Buffer Manager에게 페이지를 요청합니다. Buffer Manager은 메모리에 페이지가 존재하는지 확인하고 페이지를 반환합니다. 만약 메모리에 페이지가 존재하지 않는다면 Disk Space Manager을 통해 디스크로부터 페이지를 읽어오고 메모리에 페이지를 저장합니다.

동시성 관리는 DBMS의 핵심 기능 중 하나로, 여러 트랜잭션이 동시에 발생할 때 일관성을 유지하고 성능을 최적화하는 역할을 합니다. 그러나<이러한>동시성 관리는 DBMS의 다양한 요소에 영향을 미치므로, 특정 계층에서만 처리하는 것이 아니라, 필요한 곳에서 적절히 수행됩니다. 본 설계에서는 File and Index Manager와 Buffer Manager가 동시성 관리를 수행합니다. 

### Disk Space Manager

Disk Space Manager는 페이지 단위의 입출력 처리와 페이지 할당 및 해제를 관리합니다.

 데이터베이스는 전원이 꺼져도 데이터의 유실을 방지하기 위해 데이터를 디스크에 저장합니다. 그러나 디스크의 입출력 속도는 상대적으로 느리기 때문에, 성능 향상을 위해서는 이를 최소화하는 것이 중요합니다. ([디스크가 느린 이유](https://frozenpond.tistory.com/156))

Disk Space Manager는 이를 위해 '페이지 단위 입출력'이라는 방법을 사용합니다. 이는 단순히 필요한 데이터만 읽는 것이 아니라, 한 번에 큰 블록(여기서는 4KB)의 데이터를 읽어오는 방식입니다. 이 방법을 통해 한 번의 입출력으로 많은 양의 데이터를 읽어올 수 있으므로, 전체적인 입출력 작업을 줄일 수 있습니다.

 Disk Space Manager는 '미사용 페이지'를 효율적으로 관리하기 위해 연결 리스트를 사용합니다. 디스크의 첫 번째 페이지는 '헤더 페이지'로, 파일에 대한 정보(예: 첫 번째 빈 페이지 번호, 파일의 크기 등)를 담고 있습니다. 빈 페이지 리스트에서는 각 페이지의 첫 8바이트에 다음 빈 페이지의 주소를 저장하여, 이 페이지들이 연결 리스트처럼 동작하게 합니다.

이런 방식으로, 페이지가 필요할 때 Disk Space Manager는 빈 페이지 리스트에서 페이지를 하나 뽑아 그 주소를 제공합니다. 반대로, 페이지를 해제하려는 요청이 들어오면, 그 페이지를 다시 빈 페이지 리스트에 넣습니다.

이런 방식을 통해 Disk Space Manager는 효율적인 디스크 공간 관리를 지원하며, 디스크 I/O를 최소화하여 전체 DBMS의 성능을 향상시킵니다.

![Untitled](https://github.com/baryu97/DB/assets/74360026/73805496-7d00-4392-a264-ba81abc24498)

![Untitled](https://github.com/baryu97/DB/assets/74360026/924d5496-66a6-425b-b0a8-03f1d2c226a3)

### Index Manager

Index Manager는 데이터베이스 관리 시스템(DBMS)에서 데이터를 더 빠르게 찾을 수 있도록 돕는 도구입니다. 일반적으로, DBMS는 이를 위해 'hash index'와 'b-tree index'라는 두 가지 주요한 인덱스 구조를 사용합니다. hash index는 데이터를 빠르게 찾을 수 있는 장점이 있지만, 특정 범위 내의 데이터를 찾는 데는 적합하지 않습니다. 반면, b-tree index는 범위 검색을 효과적으로 수행할 수 있습니다. ([hash index와 b-tree index의 차이](https://www.sentryone.com/blog/introduction-to-b-tree-and-hash-indexes-in-postgresql#:~:text=Hash%20indexes%20are%20single%2Dcolumn,row%20in%20the%20heap%20table.)) 이러한 이유로, 저는 범위 탐색도 효율적으로 수행할 수 있는 b+tree라는 b-tree의 변형을 선택했습니다. ([b-tree란?](https://code-lab1.tistory.com/217))

b-tree은 데이서 삽입과 삭제시에 균형을 유지해서 트리의 높이가 일정하게 유지됩니다. 또한 한 노드가 여러 키를 가지고 있어 데이터 탐색시 비교적 적은 노드를 거치게 됩니다. 이러한 b-tree의 장점덕분에 I/O를 최대한 줄여야 하는 DBMS에서 b-tree은 자주 쓰입니다. 

b+tree는 b-tree와는 약간 다른 방식으로 작동합니다. b+tree에서는 모든 값이 '리프 노드'에만 저장되며, 리프 노드들은 서로 연결 리스트로 연결되어 있습니다. 중간 노드에는 인덱싱을 위한 키 값만 저장되기 때문에, 하나의 중간 노드에 더 많은 키를 저장할 수 있습니다. 이로 인해 범위 탐색을 더 효율적으로 할 수 있으며, 평균적으로 I/O를 더 줄일 수 있습니다.

노드의 데이터를 디스크에 저장하기 위해, 저는 4KB의 페이지를 사용했습니다. 이 페이지의 첫 128바이트는 페이지의 메타데이터(예: 부모 페이지 번호, 리프 노드 여부, 저장된 키의 개수 등)를 저장하는 '헤더' 부분으로 사용되며, 나머지 3968바이트는 키와 페이지 번호(중간 노드) 또는 실제 데이터(리프 노드)를 저장하는 데 사용됩니다.

![노드 헤더의 구조](https://github.com/baryu97/DB/assets/74360026/1d669994-6934-4d83-8152-e2be86d3ec8c)

노드 헤더의 구조

비트리는 트리를 균형있게 유지하기 위해 키의 개수에 따라 노드의 병합이나 쪼개기가 이루어집니다. 하지만 비쁠트리의 리프 노드의 경우 데이터의 크기가 가변적이라면 키의 개수가 적어도 노드가 포화상태일 수 있고 키의 개수가 많아도 공간이 남을수도있습니다. 그래서 리프 노드는 쪼개기와 병합의 기준을 키의 개수가 아닌 노드의 남은 공간의 양으로 정하고 남은 공간에 대한 정보를 헤더에 저장했습니다. 

b-tree는 균형을 유지하기 위해 키의 개수에 따라 노드를 분할하거나 병합하는 작업을 수행합니다. 그러나 제 b+tree의 리프 노드는 데이터의 크기가 가변적이기 때문에, 키의 개수가 적더라도 노드가 포화 상태일 수 있으며, 키의 개수가 많아도 공간이 남을 수 있습니다. 따라서 리프 노드에서는 노드의 남은 공간에 따라 노드를 분할하거나 병합하며, 이 정보는 헤더에 저장됩니다. 이러한 방식으로, b+tree는 데이터 검색과 동시에 효율적인 공간 관리를 가능하게 합니다.

### Buffer Manager

Buffer Manage는 메모리에서 데이터베이스의 페이지를 효과적으로 관리합니다. 이를 통해 디스크 I/O를 최소화하고, 데이터베이스의 성능을 향상시킬 수 있습니다.

Buffer Manager는 버퍼 풀을 활용하여 메모리 상에서 페이지의 읽기와 쓰기 작업을 처리합니다. 페이지 요청이 들어올 경우, Buffer Manager는 먼저 해당 페이지가 메모리에 있는지를 확인합니다. 메모리에 페이지가 존재하면, 디스크에서 페이지를 가져올 필요 없이 버퍼 풀에서 해당 페이지를 반환하게 됩니다. 반면, 메모리에 페이지가 없을 경우에는 디스크에서 해당 페이지를 가져온 후, 버퍼 풀에 저장하고 그 페이지를 반환합니다.

그러나 메모리의 크기는 제한적이므로, 모든 페이지를 버퍼 풀에 계속 저장해둘 수는 없습니다. 따라서 버퍼 풀이 가득 찼을 경우, 어떤 페이지를 제거할지 결정해야 합니다. 이를 위해 다양한 페이지 교체 알고리즘(LRU, LFU, FIFO 등)이 존재하는데, 본 DBMS에서는 LRU(Least Recently Used) 알고리즘을 채택하였습니다. LRU 알고리즘은 가장 최근에 사용되지 않은 페이지를 제거하는 방식으로, 이를 통해 자주 사용되는 페이지를 메모리에 유지하고, 상대적으로 사용 빈도가 낮은 페이지를 제거하여 효율적인 메모리 관리를 수행합니다.

또한, 버퍼 풀에서 발생하는 쓰기 작업이 완료된 페이지는 다시 디스크로 쓰여져야 합니다. 이 과정에서 'dirty bit'를 활용하여 쓰기 작업이 이루어진 페이지를 추적합니다. 즉, 페이지에 쓰기 작업이 일어나면 해당 페이지의 dirty 비트를 true로 설정하고, 페이지가 버퍼 풀에서 제거될 때 dirty 비트가 true인 페이지는 디스크에 쓰기 작업을 수행하게 됩니다.

### 동시성 관리

데이터베이스의 삽입, 변경 등의 연산이 여러 쓰레드에서 동시에 안정적으로 처리되게 관리하는 기능을 만들었습니다.

데이터베이스는 다양한 사용자와 응용 프로그램들로부터 동시에 수많은 요청을 받습니다. 이런 요청들이 데이터베이스에 동시에 처리될 때, 일관성 유지가 중요한 이슈가 됩니다. 이를 위해 트랜잭션과 동시성 제어라는 두 가지 핵심 개념을 사용합니다.

트랜잭션은 데이터베이스에서 하나의 논리적인 작업 단위를 말합니다. 예를 들어, 은행 계좌 간의 이체는 한 계좌에서 돈을 빼고 다른 계좌에 더하는 두 가지 작업으로 구성된 하나의 트랜잭션입니다. 모든 트랜잭션은 원자성, 일관성, 격리성, 지속성이라는 ACID 속성을 만족해야 합니다. 

1. **원자성(Atomicity)**: 원자성은 트랜잭션이 데이터베이스에 모두 반영되거나, 아니면 전혀 반영되지 않아야 한다는 것을 의미합니다. 은행 이체 예를 들면, 한 계좌에서 돈을 뺀 후 다른 계좌에 입금하는 과정 중 하나라도 실패하면 전체 트랜잭션이 취소되어야 합니다.
2. **일관성(Consistency)**: 일관성은 트랜잭션의 실행이 데이터베이스를 일관된 상태에서 또 다른 일관된 상태로 이동시켜야 한다는 것을 의미합니다. 예를 들어, 모든 계좌의 잔고 합계는 항상 일정해야 합니다.
3. **격리성(Isolation)**: 격리성은 각각의 트랜잭션이 동시에 실행되더라도 서로 영향을 미치지 않아야 한다는 것을 의미합니다. 즉, 한 트랜잭션이 다른 트랜잭션의 중간 결과를 볼 수 없어야 합니다.
4. **지속성(Durability)**: 지속성은 트랜잭션이 성공적으로 완료되면, 그 결과는 영구적으로 반영되어야 한다는 것을 의미합니다. 예를 들어, 한번 완료된 이체는 시스템 오류가 발생하더라도 취소되어서는 안됩니다.

동시성 제어는 여러 트랜잭션이 동시에 실행되는 환경에서 데이터의 일관성과 격리성을 보장하는 기법을 의미합니다. 대표적인 동시성 제어 방식에는 낙관적 동시성 제어, 비관적 동시성 제어, 다중 버전 동시성 제어 등이 있습니다.

데이터의 일관성을 보장하는 가장 단순한 접근 방식은 모든 트랜잭션이 순차적으로 실행되면 됩니다. 하지만 이런 방식은 한 번에 하나의 트랜잭션만 실행될 수 있기 때문에 성능 측면에서 효율이 떨어집니다. 때문에 '직렬 가능성'이라는 개념이 등장합니다. 데이터베이스에서 여러 연산들이 수행될 때 그 연산들이 순차적으로 수행되지 않았더라도 순차적으로 수행한 것과 결과가 동일하다면 그 연산들은 '직렬 가능'하다고 말합니다. 이는 복잡한 트랜잭션 환경에서도 데이터의 일관성을 유지할 수 있게 해주는 중요한 개념입니다.

아래의 그림은 이러한 '직렬 가능성'을 보여주는 예입니다. 그림 1은 순차적인 스케줄을 보여줍니다. 그림 2는 순차적이지 않은 스케줄을 보여주지만 그림 1과 동일한 결과를 보여줍니다. 그러나 그림 3은 a의 값이 순차적으로 실행한 것 보다 500 높으므로, 이 스케줄은 직렬 가능하지 않습니다.

![그림 1](https://github.com/baryu97/DB/assets/74360026/b9c15780-f863-452e-874d-7ab244e3e926)

그림 1

![그림 2](https://github.com/baryu97/DB/assets/74360026/74654029-b7c7-4ab0-af22-0665b619b9bd)

그림 2

![그림 3](https://github.com/baryu97/DB/assets/74360026/6eb036eb-7247-425b-8c43-da94ae2b36b1)

그림 3

제가 구현한 DBMS는 직렬 가능한 스케줄을 보장하기 위해 Two phase locking(이하 2PL) 기법을 사용하였습니다. 2PL을 이해하려면, 먼저 '락'이 무엇인지 이해해야 합니다. 락은 여러 트랜잭션들이 동일한 데이터에 접근하려 할 때 데이터의 일관성을 보장하기 위해 사용됩니다. 이때, 주로 사용되는 락에는 공유 락과 배타적 락, 두 가지가 있습니다.

공유 락은 '읽기' 작업을 수행할 때 설정되며, 공유 락이 설정된 데이터는 여러 트랜잭션들이 동시에 읽을 수 있습니다. 즉, 한 데이터에 여러 개의 공유 락이 설정될 수 있습니다. 그러나, 이 데이터에 대한 '쓰기' 작업은 배타적 락이 설정되어야 합니다. 이는 읽기와 달리 쓰기 작업을 수행하는 트랜잭션은 해당 데이터에 대한 유일한 접근 권한을 가져야 하기 때문입니다. 따라서, 한 번에 한 트랜잭션만이 배타적 락을 설정할 수 있습니다.

2PL은 트랜잭션이 진행되는 동안 락을 설정하는 단계와 락을 해제하는 단계, 두 단계로 나누어져 있습니다. 첫 번째 단계에서는 락을 얻을 수만 있고, 두 번째 단계에서는 락을 해제할 수만 있습니다. 이렇게 구분함으로써, 트랜잭션 도중에 락을 해제하고 다시 얻는 경우를 막아 직렬 가능성을 보장합니다.

제 DBMS에서 2PL을 구현하기 위해서는 락과 데이터를 관리하는 '락 테이블'과 트랜잭션과 락을 관리하는 '트랜잭션 테이블'이 필요합니다. 락 테이블은 데이터를 페이지 단위로 잠그며, 페이지 번호를 해시 키로 사용하는 해시 테이블 형태로 구성되어 있습니다. 그리고 각 페이지에 설정된 락들은 연결 리스트로 연결되어 있습니다.

데이터에 락을 설정하려면, 해당 데이터가 포함된 페이지 번호를 통해 락 연결 리스트를 찾고, 새로운 락을 리스트의 마지막에 추가합니다. 이 락은 트랜잭션 테이블의 트랜잭션 락 리스트에도 저장됩니다. 트랜잭션이 종료되면, 트랜잭션 락 리스트를 따라가면서 설정된 락을 락 테이블에서 제거합니다.

마지막으로, 잠금을 생성할 때마다 해당 잠금이 실행 가능한 상태인지 검사합니다. 만약 실행 가능한 상태가 아니라면, 해당 잠금을 가진 스레드는 '잠자기' 상태로 전환됩니다. 이후 다른 잠금이 해제되면, 해제되는 잠금과 연관된 잠금 중 실행 가능한 상태로 바뀐 잠금이 있는지 확인합니다. 만약 그런 잠금이 있다면, 그 잠금을 가진 스레드를 깨워 다시 작업을 진행하게 합니다. 이와 같이 잠금의 설정과 해제를 통해 데이터의 일관성을 유지하면서 동시에 여러 트랜잭션을 처리할 수 있습니다.

2PL은 트랜잭션의 직렬 가능성을 보장하지만, 두 개 이상의 트랜잭션이 서로를 기다리는 데드락이 발생할 수 있습니다. 데드락이란, 트랜잭션들이 서로의 작업이 끝나기를 기다리면서 진행이 멈춰버리는 상황을 의미합니다.

이런 문제를 해결하기 위해, 트랜잭션 간 의존성을 나타내는 그래프를 생성합니다. 이 그래프에서는 한 트랜잭션이 다른 트랜잭션을 기다리는 상황을 간선으로 표현합니다. 만약 트랜잭션이 락을 걸고 실행할 수 없는 상태라면, 그 트랜잭션을 기다리는 다른 트랜잭션을 나타내는 간선을 그래프에 추가합니다. 이렇게 추가된 간선들 중 사이클을 형성하는 경우가 있는지 확인합니다. 사이클이 발생한다면 해당 트랜잭션을 롤백합니다.

롤백은 트랜잭션의 변경사항을 원래대로 되돌리는 과정을 의미합니다. 이는 트랜잭션의 커밋과 비슷하지만, 롤백은 진행 중이던 작업을 취소한다는 차이점이 있습니다. 롤백 과정에서도 커밋과 마찬가지로 락을 순차적으로 풀고, 다른 트랜잭션들을 깨워 다시 실행하도록 합니다.

그리고, 락 테이블의 성능을 개선하기 위해 implicit locking과 lock compression 이라는 두 가지 기법을 도입했습니다.

암시적 락(implicit locking)은 명시적으로 락을 설정하는 것이 아니라, 데이터 페이지에 트랜잭션 ID를 기록하여 동작하는 방식입니다. 특정 데이터에 아무런 트랜잭션 ID가 없거나, 이미 종료된 트랜잭션 ID가 기록되어 있다면, 해당 데이터에 쓰기 작업을 수행하고 그 결과로 자신의 트랜잭션 ID를 기록합니다. 그러나, 실행 중인 다른 트랜잭션 ID가 기록되어 있다면, 그 트랜잭션에 대한 락을 락 테이블에 추가하여 명시적으로 락을 설정합니다. 

또한, lock compression은 한 페이지 내의 여러 레코드에 걸린 동일한 유형의 락을 비트맵을 이용해 한 개의 락으로 표현하는 방법입니다. 예를 들어, 트랜잭션이 특정 페이지의 3, 4, 5, 6번 레코드에 락이 필요하다면, 이를 "00111100"과 같은 비트맵으로 표현할 수 있습니다. 

### 장애회복

장애 복구는 컴퓨터 시스템에서 발생할 수 있는 다양한 장애 상황에 대응하여 트랜잭션의 원자성과 영속성을 보장하는 중요한 프로세스입니다.

FORCE 정책과 NO STEAL 정책은 이 두 가지 특성을 보장하는 가장 간단한 방법입니다. FORCE 정책은 트랜잭션의 변경 사항이 커밋되기 전에 디스크에 반영되어 영속성을 보장합니다. 반면 NO STEAL 정책은 완료되지 않은 트랜잭션의 페이지를 디스크에 기록할 수 없도록 하여 원자성을 보장합니다. 그러나 이 두 가지 방법은 모두 시스템 성능을 저하시킵니다.

 그렇기 때문에 제 DBMS는 성능 저하를 최소화하면서 원자성과 영속성을 보장하는 방법으로 STEAL/NO-FORCE 정책을 선택하였습니다. 이 정책은 버퍼가 가득 차면 완료되지 않은 트랜잭션의 페이지를 디스크에 기록할 수 있게 하여 버퍼 관리의 유연성을 제공하고, 트랜잭션 커밋 시 변경 사항을 즉시 디스크에 기록하지 않아 I/O 작업을 최소화합니다. 그러나 이 방법은 장애 발생시 원자성과 영속성을 보장하는 것이 어렵습니다. 이를 보완하기 위해 Write-Ahead Logging(WAL) 기법을 도입하였습니다. WAL은 페이지가 디스크에 기록되기 전, 또는 트랜잭션이 완료되기 전에 로그를 먼저 디스크에 기록하는 방식으로 장애 발생시 데이터 복구를 가능하게 합니다.

장애 복구 과정은 분석, 리두, 언두의 세 단계로 진행됩니다. 분석 단계에서는 로그를 검사하여 완료된 트랜잭션과 실패한 트랜잭션을 식별합니다. 이때 완료된 트랜잭션은 롤백 또는 커밋이 완료된 트랜잭션을 의미합니다. 리두 단계에서는 가장 오래된 로그부터 가장 최신 로그까지 순서대로 모든 로그 항목을 디스크에 반영하여 데이터베이스의 상태를 장애 발생 시점으로 복원합니다. 마지막으로 언두 단계에서는 가장 최신의 로그부터 가장 오래된 로그까지 순서대로 실패한 트랜잭션의 로그 항목을 롤백하여 데이터베이스를 일관성 있는 상태로 복원합니다.
