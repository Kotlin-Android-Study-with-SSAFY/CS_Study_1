### IPC

![image 12](https://github.com/user-attachments/assets/bc60f6ce-06eb-4a65-88fd-b3af482d97f4)

- 위 그림을 보면 process는 완전히 독립된 실행객체
- 다른 프로세스의 영향을 받지 않는다는 장점이 있음
- 그러나 별도의 설비 없이는 서로 통신이 어렵다는 문제가 있음
- 이를 위해서 커널 영역에서 **IPC(Inter Process Communication: 내부 프로세스간 통신)**를 제공하고, 프로세스는 커널이 제공하는 IPC 설비를 이용해서 **프로세스간 통신**을 할 수 있게 됨

---

### IPC의 2가지 표준(System V IPC와 POSIX IPC)

- **System V IPC(오래된 버전)**
    - 초기 UNIX 시스템(System V 계열)에서 도입된 전통적인 IPC 방식
    - 오랜 역사를 가진만큼 이기종간 코드 호환성을 확실히 보장함
    - 그러나, API가 오래되었으며, 함수명도 명확하지 않아서 다소 복잡하고 일관성이 떨어짐
- **POSIX IPC(비교적 최근)**
    - IEEE와 ISO 표준에 맞춰 보다 현대적인 IPC 매커니즘으로 설계됨
    - System V IPC의 단점을 보완하여, API의 일관성과 사용 편의성을 높임
    - 함수 명명법과 동작 방식이 표준화되어 있어, 프로그래머가 보다 직관적이고 일관된 방식으로 IPC를 구현할 수 있음

---

### IPC의 필요성(독립된 프로세스들의 필요성)

- **데이터 공유**
    - 프로세스들은 서로 다른 작업을 수행하지만, **결과나 상태 정보를 주고받아** 협력할 필요가 있음
- **모듈화와 확장성**
    - 복잡한 작업을 여러 개의 독립된 프로세스로 분할하면 **유지보수와 확장**이 용이해짐
- **동시성 제어**
    - 병렬 처리나 멀티스레딩 환경에서 작업을 효율적으로 분배하고 자원 경쟁을 최소화 → **연산 및 작업 속도 향상**

---

### IPC의 두 가지 주요 모델

- IPC는 주로 **공유 메모리(Shared Memory)**와 **메시지 전달(Message Passing)**으로 구분됨

- **공유 메모리(Shared Memory)** - 여러 프로세스가 동일한(공유하는) 메모리 영역을 공유하여 데이터를 교환
    
    ![image 13](https://github.com/user-attachments/assets/57a43b01-5cdc-4607-984a-a46d99e48739)

    - 프로세스가 공유 메모리 할당을 커널에 요청 시 커널은 해당 프로세스에 메모리 공간을 할당
    - 공유 메모리가 각 프로세스에게 첨부(mapping)하는 방식으로 작동
    - 공유 메모리 영역의 생성, 할당, 접근 권한 설정, 해제 등은 시스템 콜을 통해 커널이 담당 → **공유 메모리 영역의 초기 설정과 관리를 커널이 담당**
    - 
    ![image 14](https://github.com/user-attachments/assets/4f60e66a-8775-4e1f-830d-98e05e388a3e)
    
    - 통신은 **사용자가 직접 관리**하며, 운영체제가 아닌 프로세스들이 제어 → 중계자 없이 곧바로 메모리에 접근하므로 **IPC 중에서 가장 빠름**
    - 프로세스 간 Read, Write를 모두 필요로 할 때 사용(양방향)
    - 대량의 정보를 다수의 프로세스에게 베포 가능
    - **동기화 문제**: 여러 프로세스가 동시에 접근할 경우 데이터 충돌이 일어날 수 있음 → 별도의 동기화 기법(ex: 세마포어 등)을 사**용**

- **메세지 전달(Message Passing)** - 프로세스들이 메세지를 주고받으며 통신하는 방식 → 프로세스들이 공유 변수를 사용하지 않고도 데이터를 교환할 수 있음
    
    ![image 15](https://github.com/user-attachments/assets/698bdd10-800d-42a1-8ace-8f741153cdbb)

    - 주요 연산으로는 send(message), receive(message)가 있음(송수신)
    - 메세지 크기는 고정되거나 가변적일 수 있음
    - OS가 동기화를 해주기 때문에 **안전하고 동기화 문제가 없지만, 시스템 콜을 사용하기 때문에 성능 하락**(오버헤드 발생)
    - **구현 요소**
        - **통신 링크**: 두 프로세스 간의 연결을 설정하여 메세지 교환
        - **버퍼링** : 메시지 전달 시, 메세지를 일시적으로 저장하는 **메세지 큐**의 역할
            - **제로 용량**: 메세지가 큐에 저장되지 않아, 전송자(sender)가 수신자(receiver)와 동시에 rendezvous(만남)을 이뤄야 함
            - **유한 용량**:  큐의 길이가 제한되어 있어, 버퍼가 가득 차면 전송자가 대기
            - **무한 용량**: 이론적으로 무한한 큐로, 전송자가 대기하지 않음
    - **통신 방식 분류:**
        - **직접 통신 (Direct Communication)**
            - 프로세스들이 **서로의 이름을 명시적으로 지정**하여 메세지를 송수신
            - 한 쌍에는 **하나의 링크만 존재**하고, 1대 1 통신에 사용됨
            - send(P, message): P로 메시지를 보내라
            - receive(Q, message): Q로부터 메시지를 받아라.
        - 간접 통신 (Indirect Communication)
            - 메일박스(or 포트)를 통해 메세지를 송수신
            - 여러 프로세스가 하나의 메일박스를 공유할 수 있음 → 1 대 다 통신 가능
            - 메일박스 고유 ID를 통해 통신이 이뤄짐
            - send(A, message) : mailbox A 로 메시지를 보내라.
            - receive(A, message): mailbox A로부터 메시지를 받아라.

---

### IPC의 구현 방식의 범용성

- **로컬 통신**
    - 동일 시스템 내의 프로세스들이 서로 데이터를 교환하고 동기화하기 위해 IPC 메커니즘을 사용
    - 파이프, 공유 메모리, 메세지 큐, 세마포어, 시그널 등
- **원격 통신**
    - 네트워크를 통한 클라이언트-서버 모델에서도 IPC  기술을 활용해 서로 다른 시스템에 있는 프로세스간 통신
    - 소켓, RPC(원격 프로시저 호출)

---

### 주요 IPC 구현 방식

- **시그널(Signals)**
    - 한 프로세스가 다른 프로세스에 **비동기 이벤트**(ex : 종료, 인터럽트, 사용자 정의 이벤트 등)를 알리기 위한 방식
    - 간단한 알림 전송에 유용하지만, 전달할 수 있는 정보의 양 제한적
- **파이프 (Pipes)**
    
    ![image 16](https://github.com/user-attachments/assets/30e2be0f-26d5-417d-9be8-a3a825d48764)

    - 한 프로세스에서 다른 프로세스로 데이터를 순차적으로 전달하는 통로 제공
    - 종류:
        - **익명 파이프** - 주로 부모와 자식 프로세스와 같이 관련된 프로세스 간에 사용되며, 단방향 통신 제공
        - **명명 파이프(FIFO) - 이름이 부여되어 관련 없는 프로세스들 간에도 통신할 수 있으며, 양방향 통신 가능 → 이름만 알면 통신 가능**
- **소켓 (Sockets)**
    
    ![image 17](https://github.com/user-attachments/assets/72f4e001-ca17-4f65-b702-2cf4f2586fb2)

    - 네트워크 상의 통신을 위한 **엔드포인트(종착점)**로, 로컬 및 원격 프로세스 간의 데이터 전송 지원
    - TCP/IP, UDP 등의 프로토콜을 통해 신뢰성 있는 통신 or 빠른 통신을 구현할 수 있으며, 클라이언트-서버뿐만 아니라 피어 투 피어 통신에도 활용(주로는 클라이언트-서버 통신간 사용)
    
- **RPC(원격 프로시저 호출) - 중요 x**
    - 네트워크 상의 다른 주소 공간(보통 원격 시스템)에 있는 프로세스의 함수를 마치 로컬 함수 호출처럼 호출할 수 있게 해주는 메커니즘
    - 구성 요소 - 스텁(Stub)을 통해 네트워크 연결
        - 클라이언트 측 스텁 - 호출을 추상화하고 매개변수를 직렬화(marshalling)하여 서버 전송
        - 서버 측 스텁 - 수신된 매개변수를 역직렬화(unmarshalling)하여 실제 함수를 실행
    - 분산 시스템에서의 통신을 단순화하고, 다양한 아키텍처 간 데이터 표현 문제를 해결하기 위해 XDR(External Data Representation) 등의 방식 사용
    
- **메모리 맵(Memory Map) - 중요 x**
    - 공유 메모리처럼 메모리를 공유하지만, 열린 파일을 메모리에 매핑시켜서 공유하는 방식(파일 + 메모리)
    - 주로 파일로 대용량 데이터를 공유해야할 때 사용
    - 메모리 맵 파일은 파일의 크기를 바꿀 수 없음
    
- **메시지 큐 (Message Queue)**
    - 프로세스들이 메시지를 큐에 저장하고 필요할 때 꺼내어 통신할 수 있도록 함
    - **메세지 파싱**에 사용(비동기 통신)
    - 복잡하나 데이터 교환이나 우선순위 기반 메세지 처리가 필요한 상황에 활용
    - 다른 프로세스와 메세지를 교환할 때 **AMQP**을 이용
        - AMQP(Advanced Message Queuing Protocol) - 메세지 지향 미들웨어(메시지 큐)를 위한 개방형 표준 응용 계층 프로토콜
        - 기능 : 메세지 지향, 큐잉, 라우팅, 신뢰성, 보안
    - **API 차이:**
        - **System V 메시지 큐:**
            - `msgget()`으로 메시지 큐를 생성 및 접근하고, `msgsnd()`와 `msgrcv()`로 메시지를 보내고 받음
            - 메시지 큐는 커널 메모리 내에 존재하며, 프로세스 종료 후에도 명시적으로 제거하지 않으면 시스템에 남아 있을 수 있음
        - **POSIX 메시지 큐:**
            - `mq_open()`, `mq_send()`, `mq_receive()` 등의 함수를 사용하며, 이름 기반의 식별자를 통해 관리
            - 메시지 큐는 보통 파일 시스템의 일부분(예: /dev/mqueue)에 구현되어 있어, 파일 권한 모델에 따른 제어가 가능
    
- **공유 메모리(Shared Memory)**
    - 여러 프로세스가 같은 메모리 영역을 매핑하여 직접 데이터를 읽고 쓸 수 있도록 하는 방식
    - 동기화 문제 - 여러 프로세스가 동시에 같은 메모리 영역에 접근할 경우 데이터 충돌 발생 → 이를 방지하기 위해 별도의 동기화 기법(ex : 세마포어, 뮤텍스 등) 함께 사용
    - **API 차이:**
        - **System V 공유 메모리:** 키(key)를 이용해 공유 메모리 세그먼트를 생성하고, 프로세스가 이를 attach(매핑)
        - **POSIX 공유 메모리:** 이름 기반의 객체로 생성되며, `mmap()`을 통해 프로세스 주소 공간에 매핑

---

### 동기화 기법

- **세마포어(Semaphores)**
    - 여러 프로세스나 스레드나 공유 자원에 접근할 때, **동시 접근을 제어하여 경쟁 상태(race condition)를 방지**하는 동기화 기법
    - **동기화 및 상호 배제:**
        - 세마포어를 이용하면, 특정 임계 구역(critical section) 내에서 동시에 하나의 프로세스만 접근하도록 보장할 수 있음
        - **바이너리 세마포어 (Mutex) -** 0과 1의 값만 가지며, 상호 배제를 위한 역할을 함 → **사실상 한 번에 하나의 프로세스만 접근**
        - **카운팅 세마포어** - 여러 프로세스가 동시에 접근할 수 있는 최대 수를 제어하는 데 사용 → **세마포어 값을 카운팅하며 프로세스 조절, 제어**
    - **API 차이:**
        - **System V 세마포어:**
            - 여러 세마포어를 하나의 세트로 관리하며, `semget()`, `semop()`를 통해 조작
            - 복잡한 세마포어 집합 관리와 다양한 동작 옵션이 존재
        - **POSIX 세마포어:**
            - 이름 기반(named) 또는 익명(unnamed)으로 사용할 수 있으며, `sem_open()`, `sem_wait()`, `sem_post()` 등의 API를 제공
            - API가 상대적으로 단순하며, 사용이 직관적
            
- **뮤텍스 (Mutex)**
    - “Mutual Exclusion Lock”의 약자로, **임계 구역(critical section)에 한 번에 오직 하나의 프로세스나 스레드만 들어갈 수 있도록 보장**하는 도구(바이너리 세마포어의 일종)
    
- **비교 및 정리**
    - **동시 접근 제어:**
        - **세마포어:** 여러 프로세스가 동시에 일정 개수만큼 자원에 접근할 수 있도록 제어 → 카운터 값이 자원의 사용 가능 수를 나타내며, 유연한 동기화 가능
        - **뮤텍스:** 오직 하나의 프로세스만 접근할 수 있도록 제한하여, 임계 구역 내에서의 데이터 일관성 보장
    - **소유권:**
        - **세마포어:** 일반적으로 소유권 개념이 없기 때문에, 어떤 프로세스든지 wait와 signal 연산을 수행 가능
        - **뮤텍스:** 잠금을 획득한 프로세스(또는 스레드)만이 해제할 수 있도록 소유권 개념 적용
    - **사용 사례:**
        - **세마포어:** 자원 풀 관리, 생산자-소비자 문제 등에서 여러 프로세스가 제한된 자원에 접근할 때 사용
        - **뮤텍스:** 임계 구역 보호와 같이 한 번에 하나의 프로세스만 접근해야 하는 상황에서 사용

---

### 결론

- IPC는 **운영체제 내에서 여러 프로세스가 효과적으로 협력**할 수 있도록 하는 핵심 기술
- **두 가지 주요 모델(공유 메모리와 메시지 전달)**을 기반으로 다양한 구현 방식을 제공
- 각각의 방식은 데이터 교환 및 동기화의 필요성, 통신의 효율성, 그리고 시스템의 복잡성 등 여러 요소를 고려하여 선택
- 또한, 클라이언트-서버 구조에서의 통신을 위해 **시그널, 파이프, 소켓, RPC 등 여러 기법**이 활용되며, 이는 시스템 설계 시 요구사항에 따라 적절히 조합되어 사용

---

### FIFO - CLIENT

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>
#include <stdbool.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <signal.h>
#include <time.h>
#include <fcntl.h>
#include <errno.h>

// 파일 권한 및 시간 측정 관련 상수 정의
#define FPERM 0777
#define BILLION 1000000000L

// 스레드 동기화를 위한 뮤텍스와 조건 변수 초기화
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t msg_cond = PTHREAD_COND_INITIALIZER;

// 메시지 버퍼 포인터 (입력받은 메시지와 전송할 메시지)
unsigned char *get_message = NULL;
unsigned char *send_message = NULL;

// FIFO(이름있는 파이프) 파일 이름
char *fifo = "team4fifo";
// FIFO 파일 디스크립터
int fd;

// 상태를 나타내는 불리언 변수들: 각 단계별 진행 여부 관리
bool get_message_available = true;
bool make_send_message_available = false;
bool send_message_available = false;

// enter() 함수는 FIFO에 send_message를 쓰고 전송 시간을 측정합니다.
int enter(){
    int nwrite;
    struct timespec start, stop;
    double accum;
    // 전송 시작 시간 측정
    clock_gettime(CLOCK_MONOTONIC, &start);
    // FIFO에 메시지 쓰기. strlen(send_message)+1은 널 종료 문자를 포함하기 위함.
    if((nwrite = write(fd, send_message, strlen(send_message) + 1)) == -1){
        perror("message write failed");
        return -1;
    } else {
        // 전송 완료 후 시간 측정
        clock_gettime(CLOCK_MONOTONIC, &stop);
        // 전송에 걸린 시간 계산
        accum = (stop.tv_sec - start.tv_sec) + (double)(stop.tv_nsec - start.tv_nsec) / (double)BILLION;
        printf("send time: %.9f\n", accum);
        return 0;
    }
}

// 사용자의 메시지를 입력받는 스레드 함수
void *get_user_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // get_message_available 플래그가 true가 될 때까지 대기
        while(!get_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }
        // 최대 1024바이트 할당하여 메시지 입력 공간 확보
        get_message = (unsigned char *)malloc(1024);
        printf("Enter message: ");
        fgets(get_message, 1024, stdin);  // 사용자로부터 메시지 입력

        // 입력받은 메시지를 다음 단계로 넘기기 위한 플래그 설정
        make_send_message_available = true;
        get_message_available = false;
        // 다른 스레드에 상태 변경 알림
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

// 사용자 이름과 입력받은 메시지를 합쳐 전송할 메시지 형식을 만드는 스레드 함수
void *make_send_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // make_send_message_available 플래그가 true가 될 때까지 대기
        while(!(make_send_message_available)){
            pthread_cond_wait(&msg_cond, &mutex);
        }
        char *name = (char*)arg; // 인자로 전달된 사용자 이름
        // send_message 초기 할당 후 빈 문자열로 초기화
        send_message = malloc(1);
        send_message[0] = '\0'; 
        // 입력받은 메시지와 사용자 이름의 길이에 맞게 재할당
        send_message = realloc(send_message, strlen(name) + strlen(get_message) + 5);
        // 사용자 이름과 ": "를 붙인 후, 실제 입력 메시지를 추가하여 하나의 문자열로 만듦
        strcat(send_message, name);
        strcat(send_message, ": ");
        strcat(send_message, get_message);
        // 입력 메시지 메모리 해제
        free(get_message);
        get_message = NULL;

        // 다음 단계인 전송을 위한 플래그 설정
        make_send_message_available = false;
        send_message_available = true;
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

// FIFO로 전송할 메시지를 실제로 쓰는 스레드 함수
void *send_make_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // send_message_available 플래그가 true가 될 때까지 대기
        while(!send_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }

        int sig;
        // FIFO에 메시지를 쓰고, 전송 실패 시 에러 출력
        if((sig = enter()) == -1){
            perror("receive fail");
        }

        // 전송 후 send_message 메모리 해제 및 플래그 초기화
        free(send_message);
        send_message = NULL;
        send_message_available = false;
        // 사용자 입력을 다시 받을 수 있도록 플래그 설정
        get_message_available = true;
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

int main(int argc, char *argv[]){
    // FIFO 파일을 쓰기 전용, 비차단 모드로 오픈
    if((fd = open(fifo, O_WRONLY | O_NONBLOCK)) < 0){
        perror("fifo open failed");
        return -1;
    }
    // 사용자 이름을 인자로 받아야 함. (예: ./client "User Name")
    if(argc != 2){
        printf("사용법 : ./%s \"User Name\"", argv[0]);
        return -1;
    }

    // 스레드 변수 선언
    pthread_t send_thread, make_thread, get_thread;

    // 각 기능별 스레드 생성
    pthread_create(&get_thread, NULL, get_user_message, NULL);
    pthread_create(&make_thread, NULL, make_send_message, (void*)argv[1]);
    pthread_create(&send_thread, NULL, send_make_message, NULL);   

    // 생성한 스레드들이 종료될 때까지 대기
    pthread_join(get_thread, NULL);
    pthread_join(make_thread, NULL);
    pthread_join(send_thread, NULL);

    return 0;
}
```

### FIFO - SERVER

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>
#include <stdbool.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <time.h>
#include <fcntl.h>
#include <errno.h>

// 파일 권한 및 시간 측정 상수 정의
#define FPERM 0777
#define BILLION 1000000000L

// 동기화를 위한 뮤텍스와 조건 변수 초기화
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t msg_cond = PTHREAD_COND_INITIALIZER;

// FIFO(이름있는 파이프) 파일 이름 및 메시지 버퍼
char *fifo = "team4fifo";
unsigned char *message = NULL;
int fd;  // FIFO 파일 디스크립터

// 메시지 수신 단계에 대한 상태 플래그들
bool receive_message_available = true;
bool print_message_available = false;
bool save_message_available = false;

// serve() 함수는 FIFO에서 메시지를 읽고 수신 시간을 측정합니다.
int serve(){
    // 최대 1024바이트 크기의 메시지 버퍼 할당 및 초기화
    message = malloc(1024);
    memset(message, 0, 1024);

    struct timespec start, stop;
    double accum;
    // 수신 시작 시간 측정
    clock_gettime(CLOCK_MONOTONIC, &start);
    // FIFO에서 최대 1024바이트 읽기
    if(read(fd, message, 1024) < 0){
        free(message);
        message = NULL;
        return -1;
    } else {
        // 수신 완료 후 시간 측정
        clock_gettime(CLOCK_MONOTONIC, &stop);
        accum = (stop.tv_sec - start.tv_sec) + (double)(stop.tv_nsec - start.tv_nsec) / (double)BILLION;
        printf("receive time: %.9f\n", accum);
        return 0;
    }
}

// 메시지 수신 스레드: FIFO에서 메시지를 읽어와 상태 플래그를 업데이트
void *recv_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // 출력이나 저장이 진행 중이면 대기
        while(print_message_available || save_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }
        // 이전에 할당된 메시지 메모리가 있으면 해제
        if(message != NULL){
            free(message);
            message = NULL;
        }

        // FIFO에서 메시지를 읽어오는 함수 호출
        int sig = serve();

        // 메시지가 정상적으로 읽혔다면, 출력 및 저장 플래그 설정
        if(message != NULL){
            print_message_available = true;
            save_message_available = true;
        }
        // 상태 변경을 다른 스레드에 알림
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

// 메시지 출력 스레드: 수신된 메시지를 화면에 출력
void *print_comments(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // 출력할 메시지가 준비될 때까지 대기
        while(!print_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }

        // 메시지 출력
        printf("%s", message);

        // 출력 후 플래그 초기화
        print_message_available = false;
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

// 메시지 저장 스레드: 수신된 메시지를 파일에 저장
void *save_comments(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // 저장할 메시지가 준비될 때까지 대기
        while(!save_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }

        // 파일 열기 ("comments.txt"에 메시지 추가)
        FILE *comments_file;
        comments_file = fopen("comments.txt", "a");
        // 메시지를 파일에 기록
        fprintf(comments_file, "%s", message);
        fclose(comments_file);

        // 저장 후 플래그 초기화
        save_message_available = false;
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

int main(){
    // FIFO 파일 생성. 이미 존재하면 EEXIST 오류는 무시
    if(mkfifo(fifo, FPERM) == -1){
        if(errno != EEXIST)
            perror("receiver: mkfifo");
    }
    
    // FIFO 파일을 읽기/쓰기, 비차단 모드로 오픈
    if((fd = open(fifo, O_RDWR | O_NONBLOCK)) < 0)
        perror("fifo open failed");

    // "comments.txt" 파일을 초기화 (헤더 작성)
    FILE *comments_file;
    comments_file = fopen("comments.txt", "w");
    fprintf(comments_file, "- - - - - - - Comments - - - - - - -\n\n");
    fclose(comments_file);
    printf("- - - - - - - Comments - - - - - - -\n");

    // 스레드 변수 선언
    pthread_t recv_thread, print_thread, save_thread;

    // 메시지 수신, 출력, 저장 스레드 생성
    pthread_create(&recv_thread, NULL, recv_message, NULL);
    pthread_create(&print_thread, NULL, print_comments, NULL); 
    pthread_create(&save_thread, NULL, save_comments, NULL);

    // 생성된 스레드들이 종료될 때까지 대기
    pthread_join(recv_thread, NULL);
    pthread_join(print_thread, NULL);
    pthread_join(save_thread, NULL);

    return 0;
}

```

### Messing Passing - CLIENT

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>
#include <stdbool.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <signal.h>
#include <time.h>

// 메시지 큐의 키와 퍼미션, 시간 측정을 위한 상수
#define QKEY (key_t)60040
#define QPERM 0777
#define BILLION 1000000000L

// 메시지 구조체: data_type는 메시지 우선순위, message는 실제 데이터
struct message_entry{
    long data_type;
    char message[1024];
};

// 스레드 동기화를 위한 뮤텍스와 조건 변수 초기화
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t msg_cond = PTHREAD_COND_INITIALIZER;

// 사용자 입력 메시지와 전송할 메시지를 저장할 포인터 변수
unsigned char *get_message = NULL;
unsigned char *send_message = NULL;
// 전송할 메시지의 우선순위(여기서는 정수형 1로 고정)
int priority = 1;

// 각 단계별 진행 여부를 제어하는 불리언 플래그들
bool get_message_available = true;
bool make_send_message_available = false;
bool send_message_available = false;

// 메시지 큐를 생성 또는 접근하는 함수
int init_queue() {
    int qid;
    // QKEY를 이용해 메시지 큐를 생성(또는 이미 존재하면 접근)하고 QPERM 권한 설정
    if((qid = msgget(QKEY, IPC_CREAT | QPERM)) == -1){
        perror("msgget failed");
    }
    return qid;
}

// enter() 함수: 메시지 큐에 메시지를 전송하고 전송 시간을 측정
int enter(){
    // 메시지 큐 ID를 얻음
    int qid = init_queue();
    // 전송할 메시지를 담을 메시지 구조체 변수 선언
    struct message_entry send_entry;
    // 우선순위(데이터 타입) 설정
    send_entry.data_type = (long)priority;
    // send_message에 담긴 문자열을 구조체의 message 배열에 복사
    memcpy(send_entry.message, send_message, strlen(send_message));
    
    struct timespec start, stop;
    double accum;
    // 전송 시작 시간 측정
    clock_gettime(CLOCK_MONOTONIC, &start);
    // 메시지 큐에 메시지를 전송 (전송할 데이터 크기는 문자열 길이)
    if(msgsnd(qid, &send_entry, strlen(send_message), 0) == -1){
        perror("msgsnd failed");
        return -1;
    } else {
        // 전송 후 시간 측정
        clock_gettime(CLOCK_MONOTONIC, &stop);
        accum = (stop.tv_sec - start.tv_sec) + (double)(stop.tv_nsec - start.tv_nsec) / (double)BILLION;
        printf("send time: %.9f\n", accum);
        return 0;
    }
}

// 사용자로부터 메시지를 입력받는 스레드 함수
void *get_user_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // 입력 가능한 상태가 될 때까지 대기
        while(!get_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }
        // 최대 1024바이트 할당
        get_message = (unsigned char *)malloc(1024);
        printf("Enter message: ");
        // 표준 입력(stdin)으로부터 메시지 입력
        fgets(get_message,

```

### Messing Passing - SERVER

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <dirent.h>
#include <stdbool.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <time.h>

// 메시지 큐의 키와 퍼미션, 시간 측정 상수 정의
#define QKEY (key_t)60040
#define QPERM 0777
#define BILLION 1000000000L

// 메시지 구조체: data_type와 메시지 내용을 담는 배열
struct message_entry{
    long data_type;
    char message[1024];
};

// 동기화를 위한 뮤텍스와 조건 변수 초기화
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t msg_cond = PTHREAD_COND_INITIALIZER;

// 우선순위는 클라이언트와 동일하게 사용 (여기서는 별도로 사용하지 않음)
int priority = 1;
// 수신된 메시지를 저장할 포인터
unsigned char *message = NULL;

// 각 단계(수신, 출력, 파일 저장)의 진행 여부를 제어하는 플래그들
bool receive_message_available = true;
bool print_message_available = false;
bool save_message_available = false;

// 메시지 큐를 생성 또는 접근하는 함수
int init_queue() {
    int qid;
    if((qid = msgget(QKEY, IPC_CREAT | QPERM)) == -1){
        perror("msgget failed");
    }
    return qid;
}

// serve() 함수: 메시지 큐에서 메시지를 읽고 수신 시간을 측정
int serve(){
    // 수신할 메시지를 담을 구조체 변수 선언
    struct message_entry recv_entry;
    int recv_size;

    // 메시지 큐 ID 얻기
    int qid = init_queue();
    // 구조체 초기화
    memset(&recv_entry, 0, sizeof(struct message_entry));

    struct timespec start, stop;
    double accum;
    // 수신 시작 시간 측정
    clock_gettime(CLOCK_MONOTONIC, &start);
    // 메시지 큐에서 최대 1024바이트 메시지를 non-blocking 모드(IPC_NOWAIT)로 읽음
    if((recv_size = msgrcv(qid, &recv_entry, 1024, 0, IPC_NOWAIT)) == -1){
        // 메시지가 없거나 에러 발생 시 구조체 초기화 후 -1 반환
        memset(&recv_entry, 0, sizeof(struct message_entry));
        return -1;
    } else {
        // 메시지 수신 후 시간 측정
        clock_gettime(CLOCK_MONOTONIC, &stop);
        accum = (stop.tv_sec - start.tv_sec) + (double)(stop.tv_nsec - start.tv_nsec) / (double)BILLION;
        printf("receive time: %.9f\n", accum);
        // 수신된 메시지를 저장하기 위해 1024바이트 할당
        message = malloc(1024);
        memset(message, 0, 1024);
        // recv_entry 구조체에 담긴 메시지를 복사 (문자열 길이만큼)
        memcpy(message, recv_entry.message, strlen(recv_entry.message));
        memset(&recv_entry, 0, sizeof(struct message_entry));
        return 0;
    }
}

// 메시지 수신 스레드 함수: 주기적으로 메시지 큐에서 메시지를 읽어와 플래그를 설정
void *recv_message(void *arg){
    while(1){
        pthread_mutex_lock(&mutex);
        // 출력이나 저장이 진행 중이면 대기
        while(print_message_available || save_message_available){
            pthread_cond_wait(&msg_cond, &mutex);
        }
        // 이전 메시지가 있다면 메모리 해제
        if(message != NULL){
            free(message);
            message = NULL;
        }

        int sig = serve();

        // 메시지를 정상적으로 수신한 경우 출력과 저장 플래그 설정
        if(message != NULL){
            print_message_available = true;
            save_message_available = true;
        }
        // 상태 변경 알림
        pthread_cond_broadcast(&msg_cond);
        pthread_mutex_unlock(&mutex);
    }
}

// 출력 스레드 함수: 수신된 메시지를 화면에 출력
void *print_comments(void *arg){
    while(1

```

### Shared Memory - CLIENT

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <pthread.h>
#include <sys/sem.h>
#include <semaphore.h>
#include <fcntl.h>
#include <time.h>

// 공유 메모리 크기, 키, 세마포어 키, 시간 상수 등 정의
#define SHM_SIZE 1024
#define SHM_KEY2 0x60045
#define SEM_KEY2 0x60047
#define BILLION 1000000000L
#define IFLAGS (IPC_CREAT | IPC_EXCL)

// 전역 뮤텍스와 조건 변수 선언 (쓰레드 간 동기화를 위함)
pthread_mutex_t Mutex;
pthread_cond_t Cond;   // 입력 스레드와 다른 스레드 간 동기화
pthread_cond_t Cond2;  // 입력된 데이터를 파일로 저장하는 스레드 동기화
pthread_cond_t Cond3;  // 최종 공유 메모리 전송 스레드 동기화

// 입력받은 문자열과 최종 조합된 메시지를 저장할 전역 버퍼
char inputBuffer[SHM_SIZE];      // 사용자 입력을 저장
char combinedMessage[2 * SHM_SIZE]; // 사용자 이름와 입력을 합친 최종 메시지

// 세마포어 핸들 (이 코드에서는 sem_open을 통해 생성되지만, 이후 semget/semctl을 통해 재설정함)
sem_t *semaphore2;

// 시간 측정을 위한 변수
struct timespec start, stop;
double accum;

// 공유 메모리에 저장할 데이터 구조체 정의
struct shared_data {
    int flag;              // 상태 플래그 (클라이언트와 서버 간 동기화용)
    char message[SHM_SIZE]; // 공유할 메시지 버퍼
};
int flag = 0; // 내부 플래그 (조건 변수 동기화용)

// 세마포어 연산을 위한 헬퍼 함수
// 이 함수는 주어진 세마포어의 값을 value만큼 변경합니다.
void sem_change(sem_t *sem, int value) {
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = value;   // -1: P 연산(잠금), +1: V 연산(해제)
    sem_b.sem_flg = SEM_UNDO;
    semop(semaphore2, &sem_b, 1);
}

// ---------------------- 쓰레드 함수들 ----------------------

// [Thread 3: writerThread]
// 최종 조합된 메시지를 공유 메모리에 기록하는 역할을 수행
void *writerThread(struct shared_data *arg) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // 내부 플래그(flag)가 2가 될 때까지 대기 (즉, 저장 작업이 완료되어 전송할 준비가 되었을 때)
        while (flag != 2) {
            pthread_cond_wait(&Cond3, &Mutex);
        }
        // [세마포어 연산] 공유 메모리에 쓰기 전, 세마포어를 잠궈 동시 접근을 막음
        sem_change(semaphore2, -1); // 세마포어 P 연산
        // 최종 메시지를 공유 메모리의 message 필드에 복사
        strcpy(arg->message, combinedMessage);
        // 작업 후 세마포어를 해제하여 다른 프로세스가 접근할 수 있게 함
        sem_change(semaphore2, 1);  // 세마포어 V 연산

        // 전송 완료 후 공유 메모리의 flag를 1로 설정하여 서버 측에 알림
        arg->flag = 1;
        flag = 0; // 내부 플래그 초기화 (다음 입력 대기)
        // 입력 스레드가 다시 동작할 수 있도록 Cond 신호 전송
        pthread_cond_signal(&Cond);
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

// [Thread 2: savedThread]
// 사용자 입력과 사용자 이름(argv[1])를 결합하여 최종 메시지를 생성하고, 다음 단계로 flag를 올림
void *savedThread(char* argv[]) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // 내부 플래그(flag)가 1이 될 때까지 대기 (즉, 입력 스레드가 메시지를 입력했을 때)
        while (flag != 1) {
            pthread_cond_wait(&Cond2, &Mutex);
        }
        // 사용자 이름(argv[1])와 inputBuffer를 결합하여 combinedMessage 생성
        snprintf(combinedMessage, sizeof(combinedMessage), "%s:%s", argv[1], inputBuffer);
        // 생성된 메시지를 화면에 출력 (전송될 메시지 확인)
        printf("send to server : %s", combinedMessage);
        flag = 2; // 다음 단계(공유 메모리 쓰기)를 위해 flag를 2로 설정
        pthread_cond_signal(&Cond3); // writerThread를 깨워서 전송 작업 시작
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

// [Thread 1: inputThread]
// 사용자로부터 메시지를 입력받아 inputBuffer에 저장하는 역할
void *inputThread(struct shared_data *arg) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // 내부 플래그(flag)가 0이 될 때까지 대기 (즉, 이전 작업이 완료되어 입력 받을 준비가 되었을 때)
        while (flag != 0) {
            pthread_cond_wait(&Cond, &Mutex);
        }
        // 사용자에게 메시지 입력 요청
        printf("Enter message: ");
        // 사용자 입력을 inputBuffer에 저장 (최대 SHM_SIZE 크기)
        fgets(inputBuffer, sizeof(inputBuffer), stdin);
        // 입력 완료 후 flag를 1로 올려 다음 단계(메시지 결합)가 가능하도록 함
        flag = 1;
        pthread_cond_signal(&Cond2); // savedThread를 깨워 메시지 결합 시작
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

int main(int argc, char* argv[]) {
    int shmid2;
    struct shared_data *shared_memory2;
    pthread_t writerThreadId, inputThreadId, savedThreadId;

    // 세마포어 생성: "/my_semaphore2" 이름으로 생성, 초기값 1 (뮬텍스 역할)
    semaphore2 = sem_open("/my_semaphore2", O_CREAT, 0666, 1);
    pthread_mutex_init(&Mutex, NULL); // Mutex 초기화

    // 조건 변수 초기화
    pthread_cond_init(&Cond, NULL);
    pthread_cond_init(&Cond2, NULL);
    pthread_cond_init(&Cond3, NULL);

    // 공유 메모리 생성: SHM_KEY2를 사용해 크기만큼 생성
    shmid2 = shmget(SHM_KEY2, sizeof(struct shared_data), IPC_CREAT | 0666);
    if (shmid2 == -1) {
        perror("shmget");
        exit(EXIT_FAILURE);
    }
    // 공유 메모리 연결: 프로세스의 주소 공간에 attach
    shared_memory2 = (struct shared_data *)shmat(shmid2, NULL, 0);
    if ((void *)shared_memory2 == (void *)-1) {
        perror("shmat");
        exit(EXIT_FAILURE);
    }

    // 세마포어 생성 및 초기화 (System V 방식으로 semget 사용)
    semaphore2 = semget(SEM_KEY2, 1, 0);
    if (semaphore2 == -1) {
        perror("semget");
        exit(EXIT_FAILURE);
    }
    semctl(semaphore2, 0, SETVAL, 1); // 세마포어 값 1로 초기화

    // 아래 두 코드 블록은 공유 메모리와 세마포어를 종료/제거하는 코드로,
    // 프로그램 종료 시점에 호출되어야 하나, 여기서는 즉시 실행되어
    // 공유 자원을 제거하는 문제가 있으므로 실제 사용 시 주의해야 합니다.
    if (shmctl(shmid2, IPC_RMID, NULL) == -1) { // 공유 메모리 삭제
        printf("shmctl");
        exit(EXIT_FAILURE);
    }
    if (semctl(semaphore2, 0, IPC_RMID, 0) == -1) { // 세마포어 삭제
        perror("semctl");
        exit(EXIT_FAILURE);
    }
    
    // 쓰레드 생성: 사용자 입력, 메시지 결합, 공유 메모리 전송 작업을 각각의 쓰레드에서 수행
    pthread_create(&inputThreadId, NULL, inputThread, (void*)shared_memory2);
    pthread_create(&writerThreadId, NULL, writerThread, (void *)shared_memory2);
    pthread_create(&savedThreadId, NULL, savedThread, (void *)argv);
    
    // 생성된 쓰레드들이 종료될 때까지 대기
    pthread_join(savedThreadId, NULL);
    pthread_join(inputThreadId, NULL);
    pthread_join(writerThreadId, NULL);

    // 공유 메모리 분리 및 동기화 객체 소멸
    shmdt(shared_memory2);
    pthread_mutex_destroy(&Mutex);
    pthread_cond_destroy(&Cond);
    pthread_cond_destroy(&Cond2);
    pthread_cond_destroy(&Cond3);

    shmctl(shmid2, IPC_RMID, NULL);
}
```

### Shared Memory - SERVER

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>
#include <pthread.h>
#include <semaphore.h>
#include <fcntl.h>
#include <sys/sem.h>
#include <time.h>

// 정의 상수: 공유 메모리 크기, 키, 세마포어 키, 시간 측정 등
#define SHM_SIZE 1024
#define SHM_KEY 0x60044     // 1번 클라이언트용 공유 메모리 키
#define SEM_KEY 0x60046     // 1번 클라이언트용 세마포어 키
#define BILLION 1000000000L
#define SHM_KEY2 0x60045    // 2번 클라이언트용 공유 메모리 키
#define SEM_KEY2 0x60047    // 2번 클라이언트용 세마포어 키

// 서버의 버퍼들: 각 클라이언트의 공유 메모리에서 읽은 데이터를 저장
char buf[SHM_SIZE];
char buf2[SHM_SIZE];

// 파일 이름 (여기서는 파일 출력 시 사용; 실제 파일 입출력은 주석 처리된 부분 참고)
char filename[50] = "shared file";
char filename2[50] = "shared file";

// 뮤텍스: 각 클라이언트별로 독립적인 동기화를 위해 두 개 사용
pthread_mutex_t Mutex;
pthread_mutex_t Mutex2;

// 세마포어 핸들 (sem_open을 통해 생성되며, 이후 System V 방식의 semget/semctl 사용)
sem_t *semaphore;
sem_t *semaphore2;

// 내부 플래그 변수: 각 클라이언트의 상태를 나타냄 (조건 변수로 동기화)
int flag = 0;   // 1번 클라이언트용
int flag2 = 0;  // 2번 클라이언트용

// 1번 클라이언트 관련 조건 변수
pthread_cond_t Cond;
pthread_cond_t Cond2;
pthread_cond_t Cond3;
// 2번 클라이언트 관련 조건 변수
pthread_cond_t Ccond;
pthread_cond_t Ccond2;
pthread_cond_t Ccond3;

// 시간 측정을 위한 변수
struct timespec start, stop, start2, stop2;
double accum, accum2;

// 공유 메모리에 저장할 데이터 구조체 (양쪽 클라이언트 동일 구조)
struct shared_data {
    int flag;              // 공유 메모리 사용 상태를 알리는 플래그
    char message[SHM_SIZE]; // 실제 메시지를 저장
};

// 헬퍼 함수: 주어진 세마포어의 값을 value만큼 변경 (P, V 연산)
void sem_change(sem_t *sem, int value) {
    struct sembuf sem_b;
    sem_b.sem_num = 0;
    sem_b.sem_op = value;
    sem_b.sem_flg = SEM_UNDO;
    semop(semaphore, &sem_b, 1);
}

// ------------- 1번 클라이언트 관련 쓰레드 함수들 -------------

// [Thread: printThread]
// 공유 메모리에서 읽어온 데이터를 buf에 저장한 후 화면에 출력
void *printThread(struct shared_data *shared_memory) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // flag가 2가 되어야 출력할 준비가 된 상태임
        while (flag != 2) {
            pthread_cond_wait(&Cond3, &Mutex);
        }
        // 파일 입출력 대신 buf의 내용을 출력
        printf("%s", buf);
        flag = 0; // 작업 완료 후 flag 초기화
        pthread_cond_signal(&Cond); // 다음 작업(sharedThread)에게 신호 전송
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

// [Thread: makefileThread]
// 1번 클라이언트의 buf에 저장된 메시지를 파일에 쓰는 역할 (여기서는 파일 쓰기 대신 주석 처리됨)
// 실제 코드에서는 파일에 쓰고 flag를 2로 올려 출력 쓰레드가 실행하도록 함
void *makefileThread(struct shared_data *shared_memory) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // flag가 1이 될 때까지 대기
        while (flag != 1) {
            pthread_cond_wait(&Cond2, &Mutex);
        }
        // (주석 처리된 부분: buf의 내용에서 ':' 이전를 파일 이름으로 추출할 수도 있음)
        FILE *file = fopen(filename, "a");
        if (file == NULL) {
            perror("fopen");
            exit(EXIT_FAILURE);
        }
        fprintf(file, "%s", buf); // buf에 저장된 메시지를 파일에 기록
        fclose(file);
        flag = 2; // 파일 쓰기 완료 후 flag를 2로 설정하여 출력 쓰레드가 실행되도록 함
        pthread_cond_signal(&Cond3);
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

// [Thread: sharedThread]
// 1번 클라이언트의 공유 메모리에서 메시지를 읽어 buf에 복사하는 역할
void *sharedThread(struct shared_data *shared_memory) {
    while(1) {
        pthread_mutex_lock(&Mutex);
        // flag가 0일 때 (새로운 데이터 수신 가능 상태) 대기
        while (flag != 0) {
            pthread_cond_wait(&Cond, &Mutex);
        }
        // 공유 메모리의 flag가 1이면 데이터가 준비된 상태임
        if(shared_memory->flag == 1) {
            // 세마포어로 공유 메모리 접근 보호: P 연산
            sem_change(semaphore, -1);
            // 공유 메모리의 메시지를 buf에 복사
            strcpy(buf, shared_memory->message);
            // 작업 후 세마포어 해제: V 연산
            sem_change(semaphore, 1);
            // 작업 완료 후 공유 메모리의 flag를 0으로 초기화하고, 내부 flag를 1로 설정하여 다음 단계로 전환
            shared_memory->flag = 0;
            flag = 1;
        }
        pthread_cond_signal(&Cond2); // makefileThread에 신호 전송
        pthread_mutex_unlock(&Mutex);
    }
    return NULL;
}

// ------------- 2번 클라이언트 관련 쓰레드 함수들 -------------

// [Thread: printThread2]
// 2번 클라이언트의 buf2에 저장된 메시지를 파일에서 읽어 출력하는 역할
void *printThread2(struct shared_data *shared_memory2) {
    while(1) {
        pthread_mutex_lock(&Mutex2);
        // flag2가 2가 되어야 출력할 준비가 된 상태임
        while (flag2 != 2) {
            pthread_cond_wait(&Ccond3, &Mutex2);
        }
        // 파일에서 읽어온 데이터 대신 buf2의 내용을 출력
        FILE *file2 = fopen(filename2, "r");
        if (file2 == NULL) {
            perror("fopen");
            return (void*)1;
        }
        int character;
        // 실제 파일 출력 대신 buf2를 출력
        printf("%s", buf2);
        flag2 = 0; // 출력 완료 후 flag2 초기화
        pthread_cond_signal(&Ccond); // 다음 작업(sharedThread2)에게 신호 전송
        pthread_mutex_unlock(&Mutex2);
    }
    return NULL;
}

// [Thread: makefileThread2]
// 2번 클라이언트의 buf2에 저장된 메시지를 파일에 기록하는 역할
void *makefileThread2(struct shared_data *shared_memory2) {
    while(1) {
        pthread_mutex_lock(&Mutex2);
        // flag2가 1이 될 때까지 대기
        while (flag2 != 1) {
            pthread_cond_wait(&Ccond2, &Mutex2);
        }
        // (주석 처리된 부분: 파일 이름을 파싱할 수도 있음)
        FILE *file2 = fopen(filename2, "a");
        if (file2 == NULL) {
            perror("fopen");
            exit(EXIT_FAILURE);
        }
        fprintf(file2, "%s", buf2); // buf2의 내용을 파일에 기록
        fclose(file2);
        flag2 = 2; // 파일 쓰기 완료 후 flag2를 2로 설정하여 출력 쓰레드가 실행되도록 함
        pthread_cond_signal(&Ccond3);
        pthread_mutex_unlock(&Mutex2);
    }
    return NULL;
}

```

```c
// [Thread: sharedThread2]
// 2번 클라이언트의 공유 메모리에서 메시지를 읽어 buf2에 복사하는 역할
void *sharedThread2(struct shared_data *shared_memory2) {
    while(1) {
        pthread_mutex_lock(&Mutex2);
        // flag2가 0일 때 대기 (새로운 데이터 수신 가능 상태)
        while (flag2 != 0) {
            pthread_cond_wait(&Ccond, &Mutex2);
        }
        // 공유 메모리의 flag가 1이면 데이터가 준비된 상태
        if(shared_memory2->flag == 1) {
            // 세마포어로 공유 메모리 접근 보호: P 연산
            sem_change(semaphore2, -1);
            // 공유 메모리의 메시지를 buf2에 복사
            strcpy(buf2, shared_memory2->message);
            // 작업 후 세마포어 해제: V 연산
            sem_change(semaphore2, 1);
            // 작업 완료 후 공유 메모리의 flag를 0으로 초기화하고, 내부 flag2를 1로 설정하여 다음 단계로 전환
            shared_memory2->flag = 0;
            flag2 = 1;
        }
        pthread_cond_signal(&Ccond2); // makefileThread2에 신호 전송
        pthread_mutex_unlock(&Mutex2);
    }
    return NULL;
}

int main() {
    // -------------------- 1번 클라이언트 설정 --------------------
    int shmid;
    pthread_t sharedThreadId, printThreadId, makefileThreadId;
    struct shared_data *shared_memory;
    pthread_mutex_init(&Mutex, NULL); // 1번 클라이언트용 Mutex 초기화
    pthread_cond_init(&Cond, NULL);
    pthread_cond_init(&Cond2, NULL);
    pthread_cond_init(&Cond3, NULL);
    // 세마포어 생성 (sem_open 사용, 이후 semget/semctl 사용)
    semaphore = sem_open("/my_semaphore", O_CREAT, 0666, 1);
    semaphore2 = sem_open("/my_semaphore2", O_CREAT, 0666, 1);
    // 1번 클라이언트 공유 메모리 생성
    shmid = shmget(SHM_KEY, sizeof(struct shared_data), IPC_CREAT | 0666);
    if (shmid == -1) {
        perror("shmget");
        exit(EXIT_FAILURE);
    }
    // 공유 메모리 연결
    shared_memory = (struct shared_data *)shmat(shmid, NULL, 0);
    if ((void *)shared_memory == (void *)-1) {
        perror("shmat");
        exit(EXIT_FAILURE);
    }
    // System V 방식의 세마포어 생성 (세마포어 id 할당)
    semaphore = semget(SEM_KEY, 1, IPC_CREAT | 0666);
    if (semaphore == -1) {
        perror("semget");
        exit(EXIT_FAILURE);
    }

    // -------------------- 2번 클라이언트 설정 --------------------
    int shmid2;
    struct shared_data *shared_memory2;
    pthread_t sharedThreadId2, printThreadId2, makefileThreadId2;
    pthread_mutex_init(&Mutex2, NULL); // 2번 클라이언트용 Mutex 초기화
    pthread_cond_init(&Ccond, NULL);
    pthread_cond_init(&Ccond2, NULL);
    pthread_cond_init(&Ccond3, NULL);
    // 2번 클라이언트 공유 메모리 생성
    shmid2 = shmget(SHM_KEY2, sizeof(struct shared_data), IPC_CREAT | 0666);
    if (shmid2 == -1) {
        perror("shmget");
        exit(EXIT_FAILURE);
    }
    // 공유 메모리 연결
    shared_memory2 = (struct shared_data *)shmat(shmid2, NULL, 0);
    if ((void *)shared_memory2 == (void *)-1) {
        perror("shmat");
        exit(EXIT_FAILURE);
    }
    semaphore2 = semget(SEM_KEY2, 1, IPC_CREAT | 0666);
    if (semaphore2 == -1) {
        perror("semget");
        exit(EXIT_FAILURE);
    }
    // 초기 메시지 출력
    printf("- - - - - - - - - coment - - - - - - - - - \n\n");

    // 1번 클라이언트 쓰레드 생성: 공유 메모리 읽기, 파일 쓰기, 출력
    pthread_create(&sharedThreadId, NULL, sharedThread, (void*)shared_memory);
    pthread_create(&printThreadId, NULL, printThread, (void*)shared_memory);
    pthread_create(&makefileThreadId, NULL, makefileThread, (void*)shared_memory);
    // 2번 클라이언트 쓰레드 생성: 공유 메모리 읽기, 파일 쓰기, 출력
    pthread_create(&sharedThreadId2, NULL, sharedThread2, (void*)shared_memory2);
    pthread_create(&printThreadId2, NULL, printThread2, (void*)shared_memory2);
    pthread_create(&makefileThreadId2, NULL, makefileThread2, (void*)shared_memory2);
    
    // 쓰레드 종료까지 대기 (실제로는 무한 루프이므로 종료되지 않음)
    pthread_join(makefileThreadId2, NULL);
    pthread_join(printThreadId2, NULL);
    pthread_join(sharedThreadId2, NULL);
    pthread_join(makefileThreadId, NULL);
    pthread_join(printThreadId, NULL);
    pthread_join(sharedThreadId, NULL);

    // 세마포어 제거 (System V 방식)
    if (semctl(semaphore, 0, IPC_RMID, 0) == -1) {
        perror("semctl");
        exit(EXIT_FAILURE);
    }
    if (semctl(semaphore2, 0, IPC_RMID, 0) == -1) {
        perror("semctl");
        exit(EXIT_FAILURE);
    }
    // 공유 메모리 분리 및 삭제
    shmdt(shared_memory2);
    shmdt(shared_memory);
    shmctl(shmid, IPC_RMID, NULL);
    shmctl(shmid2, IPC_RMID, NULL);
}
```
