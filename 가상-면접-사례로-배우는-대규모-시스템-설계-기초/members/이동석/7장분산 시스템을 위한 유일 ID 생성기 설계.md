- multi-master replication
- UUID
    - 128비트 짜리수
    - 문제의 요구사항보다 길다
    - id 시간순 정렬 불가능
    - 숫자가 아닌 값 포함됨
- ticket server
    - auto-increament기능을 가준 중앙 집중형 서버 1개만 두는 방식.
        - SPOF 문제
- twitter snowflake 접근법
    - https://en.wikipedia.org/wiki/Snowflake_ID
    - 유명한 '2038년 문제(Year 2038 problem)'는 부호가 있는 **32비트** 정수로 **초(second)** 단위를 계산할 때 발생합니다
    - 69년이 지나면 기원 시각을 바꾸거나 ID 체계를 다른 것으로 마이그레이션해야함
- 시계 동기화 방법이 좀 중요
    - NTP라고 하는 일반적 방식 존재.
        - NetworkTimeProtocol
            - 정의
                - 인터넷상의 컴퓨터들이 서로 시간을 맞추기 위해 사용하는 약속
            - 핵심개념
                - NTP는 전 세계의 표준시를 관리하는 원자시계나 GPS로부터 시간을 받아와, 이를 네트워크에 연결된 기기들에 전파합니다.
                - 계층적 구조를 가짐
                    - 0 : 원자시계, GPS 등 실제 시간의 근원지
                    - 1 : 기기에 연결된 최상위 서버.
                    - 2 : 그밑…
            - 어떻게 작동?
                - UDP 포트 123사용
                - ntp 서버에 시간 문의 후, 받은 값과 네트워크 지연 시간까지 계산하여 정밀한 자기 시계를 보정한다
            - NTP는 일반적으로 공용 [인터넷](https://en.wikipedia.org/wiki/Internet) 에서 수십 밀리초 이내의 시간 정확도를 유지할 수 있다.
            - 보안이슈
                - 누군가 시간을 임의로 조작할 수 있는가?
                    - 답 : 가능. 단 이런것들을 예방하기위한 NTS 라는 NetworkTimeSecurity기술이 등장.
                        - **NTP 인증 (NTP Authentication):**
                        서버와 클라이언트가 서로 미리 약속된 암호(Key)를 사용하여, "진짜 믿을 수 있는 서버가 보낸 시간인가?"를 확인합니다.
                        - **NTS (Network Time Security):**
                        NTP의 최신 보안 표준입니다. TLS(암호화 통신)를 사용하여 패킷이 이동 중에 변조되지 않도록 강력하게 보호합니다.
                        - **다중 서버 참조:**
                        하나의 서버만 믿지 않고 3~4개의 서로 다른 NTP 서버에서 시간을 받아와 비교합니다. 한 곳이 유독 이상한 시간을 준다면 그 서버를 무시하는 '알고리즘'이 내장되어 있습니다.

    - 참고
        - NTP
          - https://www.juniper.net/documentation/us/en/software/junos/time-mgmt/topics/concept/network-time-security.html
          - https://en.wikipedia.org/wiki/Network_Time_Protocol
         
# 시간 오차 계산하기.
- offset = [(T2 - T1) + (T3 - T4)] / 2
- https://www.ntp.org/documentation/4.2.8-series/warp/

| 클라이언트 시간 | 서버 시간 | 상황 |
|---------------|-----------|------|
| 950 | 1000 | 클라에서 서버로 보내기 시작 |
| 960 | 1010 | 서버에 요청 도착 |
| 965 | 1015 | 서버의 시간 계산 완료 후 보낼 준비 완료 |
| 975 | 1025 | 클라에 응답 도착 완료 |

**시간 오차 계산 공식**
- ((1010 - 950) + (1015 - 975)) / 2 = 50
