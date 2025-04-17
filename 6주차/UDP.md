## 박상윤

### UDP란?

- **User Datagram Protocol - 사용자 데이터그램 프로토콜** → 데이터를 데이터그램 단위로 처리하는 프로토콜
    - 데이터그램 : 독립적인 관계를 지니는 패킷
- 전송 계층(Transport Layer)에서 사용하는 통신 프로토콜로, IP 프로토콜 위에서 동작
- TCP가 신뢰성과 연결 보장에 중점을 둔 프로토콜이라면, UDP는 속도와 효율을 중시하는 프로토콜

---

### UDP의 특징

- **비연결형(Connectionless)**
    - 통신을 시작하기 전에 연결을 설정하지 않음
- **비신뢰성(Unreliable)**
    - 데이터 전송 성공 여부를 확인하지 않음
- **순서 보장 없음**
    - 데이터그램이 도착하는 순서를 보장하지 않음
- **오버헤드 적음**
    - TCP에 비해 헤더가 단순하여 처리 속도가 빠름
- **브로드캐스트/멀티캐스트 지원**
    - 네트워크 상 여러 클라이언트에게 동시에 데이터 전송 가능
    - 브로드캐스트 : 모든 시스템에게 데이터를 보내는 방식, 통신하고자 하는 시스템의 MAC 주소를 모르거나, 네트워크에 있는 모든 서버에게 정보를 알려야 할 때 사용
    - 멀티캐스트 : 특정 그룹을 묶어 그 그룹에게만 정보를 뿌리는 네트워크 방식

- 패킷에 순서를 부여하여 재조립을 하거나 흐름 제어 또는 혼잡 제어와 같은 기능을 처리하지 않기에 속도가 빠르며 네트워크 부하가 적음
- 그러나, 신뢰성 있는 데이터의 전송을 보장하지 못함
- **신뢰성보다는 연속성이 종요한 서비스에서 사용 → 실시간 서비스(streaming)**

---

### UDP 헤더

![image](https://github.com/user-attachments/assets/6215828a-346e-466b-9113-00c320d4cd91)

![image (1)](https://github.com/user-attachments/assets/bc109f40-4382-4297-9c08-52411da8ef66)

- UDP 헤더는 TCP 헤더에 비해 휠씬 간단하고 널널함 → **데이터 전송 자체에만 초점을 맞추고 설계**되었기 때문에 헤더에 들어있는게 없음
    - Source Port: 데이터를 생성한 애플리케이션에서 사용하는 포트 번호
    - Destination Port: 목적지 애플리케이션이 사용하는 포트 번호
    - Checksum: 중복 검사의 한 형태로, 오류 정정을 통해 공간(전자 통신)이나 시간(기억 장치) 속에서 송신된 자료의 무결성을 보호하는 단순한 방법 (TCP의 체크섬과는 다르게 UDP의 체크섬은 선택사항)
    - Length: 전체 UDP의 길이(헤더 + 데이터)

---

### UDP 통신 흐름

- 송신자는 데이터를 하나의 **데이터그램**으로 만들어 전송
- 데이터그램은 수신자의 IP와 포트를 기준으로 네트워크에 전달됨
- 수신자는 해당 포트에서 데이터 수신
- ACK(응답) 없이 단방향으로만 전송되어도 통신 가능
    
    ![image (2)](https://github.com/user-attachments/assets/4abd6750-7de0-4773-acd8-fe43722358e1)


    ![image (3)](https://github.com/user-attachments/assets/9a67d6f6-4849-4d04-9488-2da0dd0d85ea)

---

### UDP의 장단점

- **장점**
    - 속도가 빠름(오버헤드가 거의 없음)
    - 실시간 서비스에 적합
    - 구현이 단순함
    - 브로드캐스트가 가능

- **단점**
    - 패킷 손실 발생 시 재전송 없음
    - 순서 보장 없음
    - 수신 여부 확인 불가
    - 보안 취약(스푸핑, DoS 공격 가능성 있음)

---

### UDP 사용 예

- **DNS** - Domain Name System은 빠른 요청/응답이 필요하므로 UDP 사용
- **스트리밍** - YouTube, Zoom, Netflix 등의 비디오/음성 스트리밍
- **VolP** - 음성 통화(KakaoTalk, Skype, Discord)
- **게임** - 실시간 반응성이 중요한 온라인 게임들 (예: FPS 게임)
- **TFTP** - 간단한 파일 전송 프로토콜도 UDP 기반

---

### TCP와 UDP 비교

| **항목** | **TCP** | **UDP** |
| --- | --- | --- |
| **연결 설정** | 필요 | 불필요 |
| **신뢰성** | 높음 (재전송, 순서 보장) | 낮음 |
| **속도** | 상대적으로 느림 | 빠름 |
| **헤더 크기** | 20바이트 이상 | 8바이트 |
| **용도** | 웹, 이메일, 파일 전송 | 실시간 스트리밍, 게임 등 |

---

### UDP 소켓 프로그래밍 예제

- 서버 (수신자)

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;

public class UDPServer {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket(9876); // 포트 번호
        byte[] buffer = new byte[1024];

        System.out.println("서버 시작...");

        while (true) {
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
            socket.receive(packet); // 클라이언트로부터 데이터 수신

            String message = new String(packet.getData(), 0, packet.getLength());
            System.out.println("클라이언트로부터 받은 메시지: " + message);

            if (message.equalsIgnoreCase("bye")) break;
        }

        socket.close();
    }
}
```

- 클라이언트 (송신자)

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.util.Scanner;

public class UDPClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();
        InetAddress serverAddress = InetAddress.getByName("localhost");
        Scanner sc = new Scanner(System.in);

        while (true) {
            System.out.print("서버로 보낼 메시지 입력: ");
            String message = sc.nextLine();
            byte[] buffer = message.getBytes();

            DatagramPacket packet = new DatagramPacket(buffer, buffer.length, serverAddress, 9876);
            socket.send(packet); // 서버에 전송

            if (message.equalsIgnoreCase("bye")) break;
        }

        socket.close();
    }
}

```

---

**심화 참고**

 **-**  [https://inpa.tistory.com/entry/NW-🌐-아직도-모호한-TCP-UDP-개념-❓-쉽게-이해하자](https://inpa.tistory.com/entry/NW-%F0%9F%8C%90-%EC%95%84%EC%A7%81%EB%8F%84-%EB%AA%A8%ED%98%B8%ED%95%9C-TCP-UDP-%EA%B0%9C%EB%85%90-%E2%9D%93-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EC%9E%90)
