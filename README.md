## 면접? 기술?
- 면접 스터디라는 느낌보다는 CS의 각자 맡은 파트를 공부하고 발표하며 지식을 공유하고 서로 궁금한 것을 계속 질문하고 이것들을 전부 기록으로 남기는 느낌의 스터디라고 생각해주시면 편할 것 같습니다.
- Deep Dive하게 할 것이냐, 기본에 충실할 것이냐는 스터디원과 함께 만들어가는 것이라고 생각합니다.

## 모집 대상
- 정말 정말 열심히 할 각오 된 분!!
- 모집은 선착순으로 정원 10명(5명 2팀) 생각하고 있습니다.

## 진행 방식
- 주 1회 2시간
- 매주 각자 인원에게 할당된 주제를 공부 (아래 예상 진행도 참고)
    - 기술 개념(최대한 많이 그리고 깊게)
- 공부를 해서 정리한 내용을 스터디원에게 발표
- 한명의 발표가 끝나고 면접 느낌으로 스터디원과 Q&A형식으로 질의 응답
- 정리한 내용을 매주 Github ReadMe 에 업로드
    - 이에 대한 내용은 본인 블로그에 다 가져가셔도 됩니다. (타 스터디원 내용까지도 허용)
 
## 
```kotlin
import kotlin.random.Random

fun main() {
    val names = listOf("오인성", "박상윤", "안유진", "이현수", "김성민")
    val numbers = (1..5).shuffled().toMutableList() // 1~5를 랜덤하게 섞음
    
    val assignments = names.mapIndexed { index, name -> name to numbers[index] } // 이름과 숫자를 매칭
    
    assignments.forEach { (name, number) ->
        println("\${name} -> \${number}")
    }
}
```

## 예상 진행도
### **📌 1주차: 컴퓨터 구조 기초 (2025.02.27)**
1. 컴퓨터의 구성 (안유진)
2. 중앙처리장치(CPU) 작동 원리 (김성민)
3. 캐시 메모리 (이현수)
4. 고정 소수점 & 부동 소수점 (오인성)
5. 패리티 비트 & 해밍 코드 (박상윤)

### **📌 2주차: 운영체제 개념 ① (2025.03.06)**
1. 운영체제란? 
2. 프로세스 vs 스레드 
3. 프로세스 주소 공간 
4. 인터럽트(Interrupt) 
5. 시스템 콜(System Call) 

### **📌 3주차: 운영체제 개념 ② (2025.03.13)**
1. PCB와 Context Switching
2. IPC(Inter Process Communication)
3. CPU 스케줄링
4. 데드락(DeadLock)
5. Race Condition

### **📌 4주차: 운영체제 메모리 관리 (2025.03.20)**
1. 세마포어(Semaphore) & 뮤텍스(Mutex)
2. 페이징 & 세그먼테이션
3. 페이지 교체 알고리즘
4. 메모리(Memory)
5. 파일 시스템

### **📌 5주차: 데이터베이스 기초 (2025.03.27)**

1. 키(Key) 정리
2. SQL - JOIN
3. SQL Injection
4. SQL vs NoSQL
5. 정규화(Normalization)

### **📌 6주차: 네트워크 기초 (2025.04.03)**
1. OSI 7 계층
2. TCP 3 way handshake & 4 way handshake
3. TCP/IP 흐름제어 & 혼잡제어
4. UDP
5. 대칭키 & 공개키

### **📌 7주차: 네트워크 심화 & 보안 (2025.04.10)**
1. HTTP & HTTPS
2. TLS/SSL handshake
3. 로드 밸런싱(Load Balancing)
4. Blocking,Non-blocking & Synchronous,Asynchronous
5. Blocking & Non-Blocking I/O

### **📌 8주차: 소프트웨어 공학 개념 (2025.04.17)**
1. 클린코드 & 리팩토링
2. TDD(Test Driven Development)
3. 애자일(Agile)
4. 객체 지향 프로그래밍(Object-Oriented Programming)
5. 함수형 프로그래밍(Fuctional Programming)

### **📌 9주차: 소프트웨어 개발 & 배포 (2025.04.24)**
1. 데브옵스(DevOps)
2. 서드 파티(3rd party)란?
3. 마이크로서비스 아키텍처(MSA)
4. CI/CD 개념
5. 서버리스(Serverless)
