<aside>

# 📌 프로세스(Process)와 스레드(Thread) 개념 정리

## 1️⃣ 프로세스(Process)란?

✅ **운영체제로부터 자원을 할당받은 "자원의 단위"**
✅ **메모리에 올라와 실행되고 있는 프로그램의 독립적인 개체**
✅ **운영체제로부터 CPU 시간, 주소 공간(Code, Data, Stack, Heap) 등을 할당받음**

![image.png](attachment:b1a1188e-193e-4b13-a209-6797198d052a:image.png)

### 📌 특징

- 독립적인 메모리 공간(Code, Data, Stack, Heap)을 가짐
- Example
    
    ```java
    public class MemoryExample {
        // 🟢 Data 영역 (전역 변수 / 정적 변수)
        static int staticVar = 10;  // 정적 변수 (초기화됨, .data)
        static int uninitializedVar;  // 정적 변수 (초기화되지 않음, .bss)
    
        public static void main(String[] args) {
            // 🔵 Stack 영역 (지역 변수, 매개변수)
            int localVar = 20;  // 지역 변수 (Stack)
    
            // 🔴 Heap 영역 (동적 할당)
            Person person1 = new Person("Alice");  // Heap에 저장
            Person person2 = new Person("Bob");    // Heap에 저장
    
            person1.display();
            person2.display();
    
            // 🟢 Data 영역 (전역 변수 사용)
            System.out.println("Static Variable: " + staticVar);
    
            // 🔴 Heap 영역에 할당된 객체 삭제 (Garbage Collector가 처리)
            person1 = null;
            person2 = null;
            
            // ⚠️ GC 호출 (강제적으로 Heap 영역 정리)
            System.gc();
        }
    }
    
    // 🔵 Code 영역 (클래스, 메서드 코드 저장)
    class Person {
        // 🔴 Heap 영역 (객체의 필드가 Heap에 저장됨)
        String name;
    
        // 생성자
        public Person(String name) {
            this.name = name;
        }
    
        public void display() {
            // 🔵 Stack 영역 (지역 변수)
            String greeting = "Hello, " + name;  // Stack
            System.out.println(greeting);
        }
    }
    
    ```
    
- 기본적으로 하나 이상의 스레드를 포함
- 서로 다른 프로세스끼리는 메모리를 공유하지 않음
- 다른 프로세스와 데이터를 주고받으려면 **프로세스 간 통신(IPC: Inter-Process Communication)** 사용
    - ex) 메시지 기반 방식 : 파이프, 파일, 소켓 등
    - ex) 공유 메모리 방식 : 공유 메모리, 메모리 맵 파일 등

---

## 2️⃣ 스레드(Thread)란?

✅ **프로세스가 할당받은 자원을 이용하는 "실행 흐름의 단위"**
✅ **프로세스 내에서 동작하는 하나의 실행 경로**
✅ **프로세스 내에서 각각 Stack을 따로 할당받지만, Code, Data, Heap 영역은 공유**

![image (1).png](attachment:92171240-bae2-45e4-9102-292d0d993a96:image_(1).png)

### 📌 특징

- 하나의 프로세스는 여러 개의 스레드를 가질 수 있음 (멀티 스레딩)

```java
class MyThread extends Thread {
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " 실행 중");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread thread1 = new MyThread();
        MyThread thread2 = new MyThread();

        thread1.start();  // 실행 순서는 보장되지 않음
        thread2.start();
    }
}
```

- 같은 프로세스 내의 스레드끼리는 힙(Heap) 영역을 공유
- 스레드는 독립적인 **레지스터**와 **스택(Stack)**을 가짐
- 한 스레드가 변경한 데이터는 같은 프로세스 내 다른 스레드에도 영향을 줌

| 구분 | 프로세스(process) | 스레드(Thread) |
| --- | --- | --- |
| 정의  | 실행 중인 프로그램(운영체제로부터 독립된 메모리 할당)  | 프로세스 내에서 실행되는 작은 실행 흐름 |
| 독립성 | 독립적인 실행 단위 | 프로세스 내에서 실행되며 독립적이지 않음 |
| 메모리 | 각 프로세스는 독립적인 메모리를 가짐 | 같은 프로세스 내에서 code, data, heap 공유, stack은 개별 할당 |
| 실행 단위  | 하나의 프로세스는 기본적으로 하나의 실행 흐름(메인 스레드) | 하나의 프로세스 내에서 여러 개의 스레드 실행 가능(멀티스레딩) |
| 통신 방식 | 프로세스 간 통신이 필요 | 같은 프로세스 내에서 쉽게 공유 가능 |
| 자원 공유 | 각 프로세스는 자신의 메모리를 독립적으로 사용  | 같은 프로세스 내에서 메모리(code, data, heap)를 공유  |
| 속도 | 프로세스 간 전환 비용이 높음 | 같은 포르세스 내에서는 전환 비용이 낮음 |

---

# 🔹 멀티 프로세스와 멀티 스레드의 차이

## 1. 멀티 프로세스

> 하나의 응용프로그램을 여러 개의 프로세스로 구성하여 각 프로세스가 하나의 작업을 처리하도록 하는 방식
> 

### ✅ 장점

- **안정성**: 여러 개의 자식 프로세스 중 하나에 문제가 발생하더라도 다른 프로세스에 영향을 주지 않음.

### ❌ 단점

- **Context Switching에서의 오버헤드**
    - Context Switching 과정에서 캐시 메모리 초기화 등 무거운 작업이 필요하며, 많은 시간이 소모됨.
    - 프로세스는 독립된 메모리 영역을 할당받기 때문에 공유 메모리가 없어 Context Switching 시 캐시 데이터가 리셋됨.
- **프로세스 간 통신(IPC)이 복잡함**
    - 프로세스는 독립적인 메모리 영역을 가지므로, 변수 공유가 불가능하여 데이터를 주고받기 위한 IPC(Inter-Process Communication)가 필요함.

📌 **Context Switching이란?**

> CPU가 여러 프로세스를 번갈아 가며 실행하는 과정. 실행 중인 프로세스가 대기 상태로 변경될 때 해당 프로세스의 상태(Context)를 저장하고, 다음 프로세스의 저장된 상태를 복구하는 작업.
> 

---

## 2. 멀티 스레드

> 하나의 응용프로그램을 여러 개의 스레드로 구성하고, 각 스레드가 하나의 작업을 처리하는 방식
> 

### ✅ 장점

- **시스템 자원 소모 감소 (자원의 효율성 증대)**
    - 프로세스를 생성하여 자원을 할당하는 시스템 콜이 줄어들어 자원을 효율적으로 관리할 수 있음.
- **시스템 처리량 증가 (처리 비용 감소)**
    - 스레드 간 데이터 공유가 쉬우며, 자원 소모가 적고 Context Switching 속도가 빠름.
- **간단한 통신 방식으로 인한 응답 시간 단축**
    - 스레드는 Stack 영역을 제외한 모든 메모리를 공유하므로 통신 부담이 적음.

### ❌ 단점

- **설계가 복잡함**: 스레드 간 자원 공유로 인해 동기화 문제가 발생할 가능성이 있음.
- **디버깅이 까다로움**: 여러 스레드가 동시에 실행되므로, 특정 문제를 재현하고 해결하는 과정이 어려울 수 있음.
- **자원 공유로 인한 동기화 문제 발생 가능**: 여러 스레드가 동일한 자원을 사용하면 충돌이 발생할 수 있으며, 이를 해결하기 위한 동기화 기법(뮤텍스, 세마포어 등)이 필요함.
- count++ 연산이 여러 스레드에서 동시에 실행되며 충돌 발생
    
    ```java
    class Counter {
        private int count = 0;
    
        public void increment() {
            count++;  // 여러 스레드가 동시에 접근하면 문제가 발생
        }
    
        public int getCount() {
            return count;
        }
    }
    
    public class Main {
        public static void main(String[] args) {
            Counter counter = new Counter();
    
            Thread t1 = new Thread(() -> {
                for (int i = 0; i < 1000; i++) counter.increment();
            });
    
            Thread t2 = new Thread(() -> {
                for (int i = 0; i < 1000; i++) counter.increment();
            });
    
            t1.start();
            t2.start();
    
            try {
                t1.join();
                t2.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    
            System.out.println("최종 카운트 값: " + counter.getCount());
        }
    }
    ```
    
- 해결 : Synchronized 키워드 사용 →  한 번에 하나의 스레드만 실행할 수 있음
    
    ```java
    class Counter {
        private int count = 0;
    
        public synchronized void increment() {
            count++;  // 동기화 적용 -> 한 번에 하나의 스레드만 접근 가능
        }
    
        public int getCount() {
            return count;
        }
    }
    ```
    
- **하나의 스레드에 문제가 발생하면 전체 프로세스에 영향**: 특정 스레드에서 오류가 발생하면 같은 프로세스 내 모든 스레드가 영향을 받을 가능성이 높음.

---

## 📌 멀티 프로세스보다 멀티 스레드를 선호하는 이유

1️⃣ **자원의 효율적 사용**

- 새로운 프로세스를 생성하면 메모리를 별도로 할당받아야 하지만,
- 스레드는 같은 프로세스의 메모리를 공유하여 자원 낭비를 줄일 수 있음

2️⃣ **빠른 작업 속도**

- 프로세스 간의 전환(Context Switching)은 무겁지만,
- 스레드는 같은 프로세스 내에서 실행되므로 전환이 빠름

3️⃣ **통신 비용 절감**

- 프로세스 간 통신(IPC)은 복잡하고 비용이 많이 들지만,
- 스레드는 같은 메모리를 공유하므로 간단하게 데이터를 주고받을 수 있음

⚠️ **BUT, 멀티 스레드는 동기화 문제를 조심해야 함**

- 여러 스레드가 동시에 같은 데이터를 수정하려 하면 충돌 발생 가능
- 이를 해결하려면 **뮤텍스(Mutex)나 세마포어(Semaphore) 같은 동기화 기법**을 사용해야 함

---

## 🔹 정리

✅ **프로세스는 독립적인 실행 단위이고, 스레드는 프로세스 내에서 실행되는 작은 작업 단위**
✅ **프로세스끼리는 메모리를 공유하지 않지만, 같은 프로세스 내 스레드는 메모리를 공유함**

</aside>
