# Snowflake vs Sonyflake 

### Snowflake ID 알고리즘

snowflake ID는 64비트 정수로 구성된 고유 식별자입니다.

| 필드         | 비트 수 | 설명                     |
| ---------- | ---: | ---------------------- |
| Timestamp  |   41 | 기준 시점 이후의 **밀리초(ms)**  |
| Sequence   |   12 | 같은 ms 내에서 생성된 ID 구분 번호 |
| Machine ID |   10 | 생성 서버/노드 식별자           |


총합 63비트 (부호 비트 제외) 로 구성되며, 시간 기반 정렬이 가능합니다.

이때 TimeStemp는 약 69년동안 안전하게 ID 를 생성할 수 있습니다.
- 41비트로 표현할 수 있는 최대 밀리초 수는 약 2^41 ≈ 2.2 * 10¹² ms

이때 snowflake보다 다른 암호화 키보다 더 많은 비트수를 사용하기 때문에 충돌확률이 거의 0에 수렴한다.
- UUID (128bit)
- ULID
- Snowflake (timestamp + workerId + sequence)

### Sonyflake 알고리즘

sonyflake는 snowflake를 확장한 형태로 라이프 타임을 더 길계 설계한 알고리즘입니다.

| 필드         | 비트 수 | 설명                |
| ---------- | ---: | ----------------- |
| 시간         |   39 | 10ms 단위 timestamp |
| Sequence   |    8 | 같은 시간 단위 내 순번     |
| Machine ID |   16 | 노드 식별자            |

sonyflake는 타임스탬프를 밀리초가 아나ㅣ라 10ms 단위로 증가시키기 때문에, 이론상 약 174년 동안 유효 ID를 생성합니다.

Go 에는 공식 라이브러리도 제공한다고 합니다.
라이브러리를 쓰면 분산 환경에서도 자동으로 unique ID가 생성됩니다.

```go
import (
    "github.com/sony/sonyflake/v2"
    "time"
)

func main() {
    st := sonyflake.Settings{
        StartTime: time.Date(2021,1,1,0,0,0,0,time.UTC),
    }
    sf, _ := sonyflake.New(st)

    id, _ := sf.NextID()
    fmt.Println(id)  // Sonyflake 고유 ID 출력
}

```

### Sonyflake가 Snowflake보다 오래 쓰는 이유
Snowflake는 타임스탬프 비트 41개에 1ms 단위로 시간을 기록해서 약 69년까지 표현 가능.

Sonyflake는 타임스탬프 비트 39개이지만 10ms 단위로 시간을 기록해서 같은 비트로 더 긴 기간을 표현함 → 약 174년.
snowflake는 1ms 단위로 생성한다고 합니다.

즉 비트 수만 보는 게 아니라 “시간 단위 × 비트 수”로 전체 기간이 결정된다.
41bit × 1ms vs 39bit × 10ms → 후자가 더 큰 기간을 커버하게 되는 것.

### Snowflake 와 sonyflake의 장단점

Snowlfake 는 sony보다 할당된 비트수(12 bit sequence)가 더 많기때문에 서버가 트래픽이 몰려도 더많은 ID 를 커버할수 있다.

또한  10ms 단위라서 시간 정밀도가 약해 동시 생성량 한계가 더빨리옵니다.

또한, sonyflake는 비교적 최신 라이브러리라 안정성도 떨어진다고 합니다. (글로벌 표준이 아니라고합니다. 글로벌 표준은 snowlfake)
