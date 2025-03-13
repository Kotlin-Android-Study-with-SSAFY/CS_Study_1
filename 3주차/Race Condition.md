# Race Condition

# Race Condition 이란?

- 둘 이상의 프로세스 혹은 쓰레드가 공유자원에 접근할 때 결과값이 의도와 달라질 수 있는 문제

![image](https://github.com/user-attachments/assets/f05b04a1-32ac-49a9-b394-6d711221f40c)

# Race Condition 발생 요건

1. **동시 접근 (Concurrent Access)**
    - 여러 스레드가 같은 공유 자원에 동시에 접근할 수 있을 때
        - 동시에 write 작업 또는 read, write 작업
        - 단순히 동시에 read 작업만 수행할 경우 x
2. **비결정적 실행 순서 (Non-deterministic Execution Order)**
    - 실행 순서에 따라 서로 다른 결과가 발생해야 함
    - 즉, 실행 타이밍이 바뀌면 프로그램의 동작이 달라질 수 있을 때

### ex)

```kotlin
import kotlinx.coroutines.*

var counter = 0  // 공유 변수

suspend fun increment() {
    counter++
}

fun main() = runBlocking {
    val jobs = List(100) {
        launch(Dispatchers.Default) {
            repeat(1000) {
                increment()
            }
        }
    }
    jobs.forEach { it.join() }
    println("Counter: $counter")  // 기대값: 100000, 실제값은 다를 수 있음
}
```

![image](https://github.com/user-attachments/assets/c5cd4471-be91-4b9a-b756-f5b505f2cc3d)

# 경쟁 프로세스가 직면하는 문제

## 1. Deadlock

- 둘 이상의 프로세스가 서로 상대방이 점유한 리소스를 기다리면서 **영원히 멈추는 현상**

### 1-1. Deadlock 발생 조건

- **상호 배제 (Mutual Exclusion)**: 자원은 한 번에 하나의 프로세스만 사용할 수 있음
- **점유 및 대기 (Hold and Wait)**: 프로세스가 이미 할당된 자원을 유지하면서 추가 자원을 요청함
- **비선점 (No Preemption)**: 할당된 자원을 강제로 빼앗을 수 없음
- **순환 대기 (Circular Wait)**: 프로세스 간의 자원 요청이 **순환 구조**를 가짐
- 모든 조건이 충족될 때 데드락이 발생함

### 1-2. Deadlock 예제 : 식사하는 철학자

![image](https://github.com/user-attachments/assets/18015a17-7fbb-4da0-af60-6c38eeb91806)

- 원형 테이블에 철학자들이 식사 할려고 앉아서 동시에 식사를 시작함
- 철학자들은 식사를 시작하기 위해서는 두개의 젓가락이 필요함
- 한번에 하나의 젓가락만 들수있으며 왼쪽 젓가락 먼져 들음
- 식사가 끝난다면 철학자들은 젓가락을 원래 자리에 내려놓고 식사를 마침
- 처음 동시에 모든 철학자들이 왼쪽 젓가락을 들고 오른쪽 젓가락을 들려고 할 때 오른쪽 젓가릭이 없어 식사를 시작하지 못하고 영원히 대기하는 상황이 발생 ⇒ 교착 상태(Deadlock)

## 2. 기아 상태 (Starvation)

- 하나의 프로세스(또는 스레드)가 **자원을 할당받지 못하고 무한히 대기하는 문제**
- 데드락과 달리 프로세스가 완전히 멈추는 것은 아니지만, 특정 프로세스가 우선순위가 낮아 **계속 실행 기회를 얻지 못하는 문제**

### **2-1. 기아 상태 발생 원인**

- 우선순위 스케줄링(Priority Scheduling)에서 낮은 우선순위의 프로세스가 계속 밀릴 때
- 리소스 할당이 **불공정**할 때
- **락(Lock) 또는 세마포어(Semaphore) 기반 동기화**에서 특정 프로세스가 계속 기다릴 때

## 3. 라이브락 (Livelock)

- 두 개 이상의 프로세스가 **서로를 방해하면서 무한히 상태를 변경하지만, 실제로 작업을 수행하지 못하는 문제**.
- 데드락과 달리 프로세스가 완전히 멈추는 것이 아니라 **계속해서 상태를 변경하지만 진행되지 않는 상황**.

### Livelock 예제

```kotlin
import kotlin.concurrent.thread
import java.util.concurrent.locks.ReentrantLock

val lock1 = ReentrantLock()
val lock2 = ReentrantLock()

fun process1() {
    while (true) {
        if (lock1.tryLock()) {
            println("Process 1 locked lock1")
            Thread.sleep(10)  // 잠시 대기
            if (lock2.tryLock()) {
                println("Process 1 completed")
                lock2.unlock()
                lock1.unlock()
                return  // 종료
            } else {
                println("Process 1 unlocked lock1 and retrying")
                lock1.unlock()  // 다시 시도
            }
        }
        Thread.sleep(10)  // 다시 시도
    }
}

fun process2() {
    while (true) {
        if (lock2.tryLock()) {
            println("Process 2 locked lock2")
            Thread.sleep(10)  // 잠시 대기
            if (lock1.tryLock()) {
                println("Process 2 completed")
                lock1.unlock()
                lock2.unlock()
                return  // 종료
            } else {
                println("Process 2 unlocked lock2 and retrying")
                lock2.unlock()  // 다시 시도
            }
        }
        Thread.sleep(10)  // 다시 시도
    }
}

fun main() {
    // 두 프로세스를 서로 다른 스레드에서 실행
    val thread1 = thread { process1() }
    val thread2 = thread { process2() }

    // 두 스레드가 종료될 때까지 대기
    thread1.join()
    thread2.join()
}
```

# Race Condition 예방 방법

## 1. 조건 변수(Condition Variables) 사용

- 동기화가 필요한 스레드 간의 조건을 관리할 수 있는 조건 변수(Condition Variables)를 사용하면 경쟁 조건을 방지
- 이를 통해 특정 조건이 만족될 때까지 스레드를 대기시킬 수 있음
- ex)
    
    ```kotlin
    import java.util.concurrent.locks.ReentrantLock;
    import java.util.concurrent.locks.Condition;
    
    public class RaceConditionExample {
        private static final ReentrantLock lock = new ReentrantLock();
        private static final Condition condition = lock.newCondition();
        private static int counter = 0;
    
        public static void incrementCounter() throws InterruptedException {
            lock.lock();
            try {
                // 특정 조건을 만족할 때까지 대기
                while (counter == 5) {
                    condition.await();
                }
                counter++;
                condition.signalAll();  // 다른 스레드가 대기할 수 있도록 신호
            } finally {
                lock.unlock();
            }
        }
    }
     
    ```
    

## 2. 스레드 안전한 자료구조 사용

- Java에서는 `ConcurrentHashMap`, `CopyOnWriteArrayList`와 같은 스레드 안전한 컬렉션을 제공

## 3. 원자적(Atomic) 연산 사용

- 원자적 연산(Atomic Operation)이란, 연산이 중단되지 않고 **완전하게 실행되는** 연산을 의미
- 원자적 연산은 다른 스레드가 개입할 수 없도록 보장되며, 여러 스레드가 동시에 동일한 데이터를 변경하는 상황에서 **경쟁 조건을 방지**하는 데 중요한 역할을 함

### 3-1. 원자적 연산의 특징

- **중단되지 않음:** 연산이 중간에 다른 스레드에 의해 끼어들지 않으며, 연산이 완료될 때까지 다른 스레드가 이를 중단 불가
- **단위화:** 연산이 하나의 단위로 실행되며, 그 결과는 **즉시 반영**

## 4. 트랜젝션 사용

- 트랜잭션(Transaction)은 **여러 데이터베이스 작업을 하나의 단위로 묶는 것**을 의미
- 트랜잭션 내에서 수행되는 작업들은 모두 **원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 지속성(Durability)** (ACID 속성)을 보장
- 그 중에서도 격리성(Isolation)은 트랜잭션 간에 서로 영향을 미치지 않도록 보장하는 역할을 하여 경쟁 조건을 방지하는 데 중요한 역할을 함
- 트랜잭션을 처리하는 DBMS는 **잠금(Locking) 또는 타임스탬프(Versioning)** 기법을 사용하여 동시에 여러 트랜잭션이 실행될 때 데이터의 일관성을 유지 ⇒ 병행 제어(Concurrency Control)

## 5. 락프리(Lock-Free) 알고리즘

- **락프리(Lock-Free)** 알고리즘은 **스레드가 자원에 접근할 때, 락을 사용하지 않고도 안전하게 공유 자원에 접근**할 수 있도록 해주는 알고리즘
- 락을 사용하지 않기 때문에 **성능**이 뛰어나며 교착 상태(Deadlock)나 기아(Starvation)를 방지하는 데 유리
- 프리 알고리즘은 주로 비교 후 교환(CAS, Compare-and-Swap)을 이용해 구현
- **CAS**는 두 개의 값 중 하나가 예상한 값과 같다면, 해당 값을 새 값으로 교체하는 연산, 이는 원자적이며, 다른 스레드가 동시에 값을 변경하지 않도록 보장

### 5-1. CAS (Compare-And-Swap) 설명

`CAS`는 세 가지 인수를 사용:

1. **메모리 위치:** 값을 저장할 메모리 위치를 지정
2. **기존 값:** 현재 그 메모리 위치에 저장된 값을 읽어들임
3. **새 값:** 메모리 위치에 새로운 값을 저장
- 이 과정은 **원자적**으로 실행되며, 만약 다른 스레드가 그 메모리 위치를 변경했으면 `CAS` 연산은 실패하고, 다시 시도

## 6. Mutex

- **공유 자원에 동시에 접근하지 못하도록** 보장하는 **동기화 객체**
- 주로 임계 영역(Critical Section)이라고 불리는 자원에 대한 접근을 관리하는 데 사용
1. 자원에 접근하려면 해당 뮤텍스를 **잠금(Lock)**
2. 뮤텍스가 잠겨 있으면 다른 스레드는 해당 자원에 접근하지 못하고 대기
3. 자원에 대한 작업이 끝나면 뮤텍스를 **해제(Unlock)** 하고, 대기 중인 다른 스레드가 자원에 접근

## 7. Semaphore

- **동시 실행 가능한 작업의 수를 제한**하는 **동기화 객체**
- **자원의 개수**를 관리하며, 이를 통해 여러 스레드가 자원에 접근하는 것을 제어
- **이진 세마포어(Binary Semaphore):** 값이 0 또는 1로 제한되며, 뮤텍스와 매우 유사 일반적으로 하나의 자원만 접근을 허용하는 경우 사용
- **카운팅 세마포어(Counting Semaphore):** 자원의 개수를 지정하는 세마포어로, 이 값은 자원의 수를 나타내며 여러 스레드가 자원에 동시에 접근
1. 세마포어는 **자원의 개수**를 추적하는 카운터를 유지
2. 스레드가 자원에 접근하려고 할 때, 세마포어 카운터 값을 **감소**
    - 카운터 값이 0이면, 더 이상 자원을 사용할 수 없으므로 대기
3. 작업이 끝난 후, 자원을 **반환**하면 카운터 값이 **증가**하여 대기 중인 스레드가 자원을 사용

# Mutex & Semaphore 교착상태

- Mutex와 Semaphore 모두 교착상태에 취약함
- 따라서 사용할 때에는 교착상태가 일어나지 않도록 해야함
- 교착 상태를 없애기 위한 방법은 여러 가지가 있으며, 그 중에서도 **예방(Prevention)**, **회피(Avoidance)**, **탐지(Detection)** 등의 전략이 있음
