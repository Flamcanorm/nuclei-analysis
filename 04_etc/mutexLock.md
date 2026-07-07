---
상태: false
---
[Config struct](../02_Internal_Packages/pkg_catalog/config/nucleiconfig.go.md#Config%20struct) 의 `sync.Mutex`

- go에서 기본적으로 제공하는 `sync` 라이브러리에 정의된 `Mutex`는 **적응형 스핀락(Adaptive Spinlock)** 임
- **적응형 스핀락**은 기존의 **대기락(Sleep/Wait Lock)** 방식과 **스핀락(Spinlock)** 방식을 혼용한 것
	- 1. 고루틴에 도착, 락 획득 시도 -> 잠긴 상태
	- 2. 대기큐로 이동하지 않고 스핀락 시도
	- -> 앞의 작업이 끝난 경우 락 획득 성공
	- -> 여러 번 실패시 고루틴의 상태를 대기(Sleep)으로 전환
	- 3. 커널의 대기 큐에서 대기 (대기락)
	- 4. 앞의 작업이 끝나고 큐에서 고루틴을 깨움 (Wakeup)
	