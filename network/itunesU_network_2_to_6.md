## 2강

* 특정 프로세스를 찾기 위해서는 특정 IP(호스트) + port(프로세스)가 필요함 -> 특정 소켓을 지칭할 수 있음
* 애플리케이션 계층(HTTP 등) > 트랜스포트 계층(TCP, UDP)
* TCP : 신뢰성 있는 통신, 데이터 유실 없음, 보안 없음, 타임에 대한 개런티 없음
* UDP : 비신뢰성 통신, 데이터 유실 있음, 보안 없음, 타임에 대한 개런티 없음
* 통신하려는 상대와 같은 프로토콜로 통신해야함(TCP - TCP)
* HTTP는 어플리케이션 계층이기 때문에 멀리 떨어진 서버와 신뢰성 있는 통신을 하기 위해서는 트랜스포트 계층의 TCP 프로토콜로 먼저 서버와 소켓이 연결이 되어야한다.
* non-persistent HTTP(매 request마다 tcp 새로 연결) VS persistent HTTP(현재 버전의 default, 한번 연결된 TCP 프로토콜은 끊지 않고 지속함)
* 파이프 라인 방식 (html 내 오브젝트(이미지, 영상 등)들은 파이프라인 방식을 이용(한꺼번에 처리))
* Connection은 아래의 다섯가지 요인으로 식별, 이중 remote ip와 remote port를 제외한 나머지는 고정값

* 304 Not Modified는 서버에 요청할 때 반환될 값이 특정 date 이후로 변했는지를 체크해서 변한게 없다면 실제 데이터를 전송하지 않고 304 status code를 반환한다.

## 3강

* IP + PORT(프로세싱)
* DNS : host name과 IP 어드레스의 매핑 디렉토리, 전세계 곳곳에 분할되어 있음
* 성능상의 이유, 무수히 많은 데이터로 인해 분산되어 있고 계층화되어있다. (com DNS servers, org DNS servers … )
* DNS record : name(Host), value(IP), type, TTL(Time To Leave: 유효기간)
* type = A : 이름이 호스트네임, value가 IP address
* type = NS : name is domain, value는 authoritative name server의 호스트네임
* authoritative name server는 모두 Atype (ex. www.ha.edu, cse.ha.edu)
* dns.xxx.com 이라는 authoritative name server를 먼저 구성해야함 (아마 호스트 업체에서 대신 운영하는듯), 그리고 .com 네임서버에 등록 필요

4강

* 트랜스포터 레이어(TCP/UDP)
* Connectionless demux는 dest IP, port로 소켓을 분리, Connection demux(TCP)는 destIP, port, sourceIP, port 이렇게 4가지가 동일할때만 동일한 소켓을 열어줌.
* 멀티플렉싱은 하나의 전송로를 여러 사용자(클라이언트)가 동시에 사용해서 효율성을 높이는 것
* 여기서는 어플리케이션 레이어에서 특정 소켓을 찾아가는 과정을 뜻함, 반대로 디멀티플렉싱은 리시버에서 소켓에서 실제 프로세스를 찾아가는 과정?
* 실제로 구글에 접속할때 하나의 사용자에 하나의 소켓이 열리는게 아니라, 여러개의 쓰레드로 이뤄진 프로세스에 하나의 쓰레드와 연결됨
* DNS는 UDP 프로토콜을 사용함 —> why??
* UDP checksum은 데이터의 에러를 판단하기 위한 장치
* UDP에서 source port가 필요한 이유는 서버에서 return 해줄 주소를 알고 있어야 하므로
