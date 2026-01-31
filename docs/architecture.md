# 아키텍처 설계서 (Architecture Design)

**프로젝트**: STM32 Dam Control System  
**버전**: 1.0  
**작성일**: 2025-01-31

---

## 1. 시스템 아키텍처

### 1.1 레이어 구조

```
┌─────────────────────────────────────────┐
│     Application Layer (App/)            │
│  - 메인 로직 (ap.c)                      │
│  - 상태 머신 관리                         │
│  - 데이터 처리                            │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────┴───────────────────────┐
│     Driver Layer (App/)                 │
│  - dht11.c: 온습도 센서                  │
│  - i2c-lcd.c: LCD 디스플레이             │
│  - keypad.c: 키 입력                     │
│  - water_state_logger.c: 로깅            │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────┴───────────────────────┐
│     HAL Layer (Core/)                   │
│  - gpio.c: 핀 제어                       │
│  - i2c.c: I2C 통신                       │
│  - tim.c: 타이머                         │
│  - usart.c: UART 통신                    │
│  - adc.c: ADC 변환                       │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────┴───────────────────────┐
│     Hardware (STM32F411)                │
└─────────────────────────────────────────┘
```

### 1.2 디렉토리 구조

```
STM32-Dam-Control-System/
│
├── App/                    # 애플리케이션 레이어
│   ├── Inc/                # 헤더 파일
│   │   ├── ap.h           # 메인 앱
│   │   ├── ap_def.h       # 공통 정의
│   │   ├── dht11.h        # DHT11 드라이버
│   │   ├── i2c-lcd.h      # LCD 드라이버
│   │   ├── keypad.h       # 키패드 드라이버
│   │   └── water_state_logger.h
│   │
│   └── Src/                # 소스 파일
│       ├── ap.c           # 메인 로직
│       ├── dht11.c
│       ├── i2c-lcd.c
│       ├── keypad.c
│       └── water_state_logger.c
│
└── Core/                   # HAL 레이어
    ├── Inc/
    │   ├── main.h         # 메인 설정
    │   ├── gpio.h         # GPIO 초기화
    │   ├── i2c.h          # I2C 설정
    │   ├── tim.h          # 타이머 설정
    │   ├── usart.h        # UART 설정
    │   └── adc.h          # ADC 설정
    │
    └── Src/
        ├── main.c
        ├── gpio.c
        ├── i2c.c
        ├── tim.c
        ├── usart.c
        └── adc.c
```

---

## 2. 모듈 설계

### 2.1 메인 애플리케이션 (ap.c/h)

**목적**: 전체 시스템 제어 및 조율

**주요 함수**:
```c
void App_Init(void);           // 모든 모듈 초기화
void App_Main(void);           // 메인 루프
void App_ProcessSensors(void); // 센서 데이터 처리
void App_UpdateLCD(void);      // LCD 업데이트
void App_CheckThreshold(void); // 임계값 검사
```

**상태 머신**:
```
IDLE → MONITORING → ALARM
  ↓         ↑
SETTING ←───┘
```

**데이터 플로우**:
```
센서 읽기 → 데이터 변환 → LCD 표시 → UART 전송 → 임계값 검사
```

### 2.2 DHT11 드라이버 (dht11.c/h)

**목적**: DHT11 온습도 센서 제어

**인터페이스**:
```c
typedef struct {
    float temperature;  // °C
    float humidity;     // %
    uint8_t valid;      // 데이터 유효성
} DHT11_Data_t;

void DHT11_Init(void);
uint8_t DHT11_Read(DHT11_Data_t* data);
```

**동작 원리**:
1. Start 신호 전송 (18ms LOW)
2. 응답 대기 (80μs LOW + 80μs HIGH)
3. 40bit 데이터 수신 (5 bytes)
4. 체크섬 검증

**타이밍 요구사항**:
- μs 단위 지연: TIM2 사용
- 전체 읽기 시간: ~20ms

### 2.3 I2C LCD 드라이버 (i2c-lcd.c/h)

**목적**: 16x2 또는 20x4 LCD 제어

**인터페이스**:
```c
void LCD_Init(void);
void LCD_Clear(void);
void LCD_SetCursor(uint8_t row, uint8_t col);
void LCD_Print(const char* str);
void LCD_PrintInt(int num);
void LCD_PrintFloat(float num, uint8_t decimals);
```

**내부 구조**:
- I2C 주소: 0x27 (또는 0x3F)
- PCF8574 I/O 확장기 사용
- 4bit 모드 통신

**초기화 시퀀스**:
```
1. 전원 대기 (50ms)
2. Function Set (8bit)
3. Function Set (4bit)
4. Display On/Off
5. Clear Display
6. Entry Mode Set
```

### 2.4 키패드 드라이버 (keypad.c/h)

**목적**: 4x4 매트릭스 키패드 스캔

**인터페이스**:
```c
char Keypad_Scan(void);  // 반환: '0'-'9', 'A'-'D', '*', '#', 0(없음)
```

**스캔 알고리즘**:
```c
for (각 행) {
    해당 행 LOW 출력
    for (각 열) {
        열 상태 읽기
        if (LOW 감지) {
            키 위치 계산
            디바운싱 (50ms)
            키 값 반환
        }
    }
    해당 행 HIGH 복원
}
```

**디바운싱**:
- 첫 감지 후 50ms 대기
- 재확인 후 키 값 반환

### 2.5 데이터 로거 (water_state_logger.c/h)

**목적**: UART를 통한 데이터 기록

**인터페이스**:
```c
typedef struct {
    uint32_t timestamp;    // ms 단위
    float water_level;     // %
    float temperature;     // °C
    float humidity;        // %
} WaterState_t;

void Logger_Init(void);
void Logger_Send(WaterState_t* state);
```

**출력 형식**:
```
Timestamp,WaterLevel,Temperature,Humidity
1234,75.3,23.5,45.2
```

---

## 3. 데이터 흐름도

### 3.1 센서 데이터 처리

```
┌──────────┐
│ ADC 변환  │ → Raw Value (0-4095)
└────┬─────┘
     │
     ↓
┌──────────────┐
│ 값 환산       │ → Water Level (0-100%)
└────┬─────────┘
     │
     ↓
┌──────────────┐
│ DHT11 읽기   │ → Temp & Humi
└────┬─────────┘
     │
     ↓
┌──────────────┐
│ 데이터 구조체 │ → WaterState_t
│ 구성         │
└────┬─────────┘
     │
     ├────→ LCD 표시
     │
     └────→ UART 전송
```

### 3.2 사용자 입력 처리

```
┌──────────┐
│ 키패드    │
│ 스캔      │
└────┬─────┘
     │
     ↓
  키 입력?
     │
     ├─ '1' → 수위 화면
     ├─ '2' → 온습도 화면
     ├─ '3' → 설정 모드
     └─ '#' → 메인 메뉴
```

---

## 4. 타이밍 다이어그램

### 4.1 메인 루프 타이밍 (1초 주기)

```
0ms     100ms   200ms   300ms   400ms   500ms   600ms   700ms   800ms   900ms   1000ms
│        │       │       │       │       │       │       │       │       │       │
├─ADC────┤
         ├─DHT11─┤ (20ms)
                 ├─LCD───┤ (50ms)
                         ├─Keypad┤ (10ms)
                                 ├─UART──┤ (5ms)
                                         ├─Check─┤
                                                 └─Wait─────────────┘
```

### 4.2 인터럽트 우선순위

| 인터럽트 | 우선순위 | 용도 |
|---------|---------|------|
| TIM2 | 0 (최고) | DHT11 타이밍 |
| ADC | 1 | 수위 센서 |
| UART1 | 2 | 데이터 전송 |
| I2C1 | 3 | LCD 통신 |

---

## 5. 메모리 사용

### 5.1 Flash 사용 예상

| 항목 | 크기 |
|-----|------|
| HAL 라이브러리 | ~50KB |
| App 코드 | ~20KB |
| 드라이버 코드 | ~15KB |
| **총합** | **~85KB / 512KB** |

### 5.2 RAM 사용 예상

| 항목 | 크기 |
|-----|------|
| 스택 | 2KB |
| HAL 버퍼 | 5KB |
| 애플리케이션 변수 | 3KB |
| **총합** | **~10KB / 128KB** |

---

## 6. 에러 처리 전략

### 6.1 센서 읽기 실패

```c
if (DHT11_Read(&data) == 0) {
    retry_count++;
    if (retry_count > 3) {
        // 이전 값 유지
        LCD_Print("Sensor Error!");
        error_flag = 1;
    }
}
```

### 6.2 통신 타임아웃

```c
if (HAL_I2C_Transmit(&hi2c1, ..., 100) != HAL_OK) {
    LCD_Init();  // 재초기화 시도
}
```

### 6.3 임계값 초과

```c
if (water_level > threshold) {
    alarm_state = 1;
    LCD_Print("!! ALARM !!");
    Logger_SendAlarm();
}
```

---

## 7. 확장성 고려사항

### 7.1 추가 가능 기능

- **다중 센서**: 여러 수위 센서 동시 모니터링
- **무선 통신**: ESP8266 연동 (UART2 사용)
- **SD 카드 로깅**: SPI 인터페이스 추가
- **릴레이 제어**: GPIO 출력으로 펌프 제어

### 7.2 모듈 독립성

각 드라이버는 독립적으로 교체 가능:
- DHT11 → DHT22 (핀 호환)
- I2C LCD → SPI LCD (인터페이스 변경)
- 4x4 키패드 → 3x4 키패드 (배열 크기만 수정)

---

## 8. 성능 분석

### 8.1 CPU 사용률 (추정)

| 작업 | 주기 | 실행 시간 | CPU % |
|-----|------|----------|--------|
| ADC 읽기 | 1s | 1ms | 0.1% |
| DHT11 읽기 | 1s | 20ms | 2% |
| LCD 업데이트 | 1s | 50ms | 5% |
| 키패드 스캔 | 100ms | 5ms | 5% |
| UART 전송 | 1s | 5ms | 0.5% |
| **총합** | | | **~13%** |

### 8.2 응답 시간

- 키 입력 → LCD 반영: < 100ms
- 센서 변화 → 경보: < 1.1s (다음 주기)
- UART 명령 수신 → 응답: < 50ms

---

## 9. 설계 결정 사항

| 결정 사항 | 근거 |
|---------|------|
| 베어메탈 (No RTOS) | 간단한 작업, RTOS 오버헤드 불필요 |
| HAL 사용 | 이식성 및 유지보수성 향상 |
| 1초 주기 | 댐 수위는 빠르게 변하지 않음 |
| CSV 로그 형식 | 엑셀/Python 분석 용이 |
| 전역변수 최소화 | 함수 인자로 전달 (재진입성) |

---

## 10. 참고 문서

- STM32F411 Reference Manual (RM0383)
- DHT11 Datasheet
- HD44780 LCD Controller Datasheet
- HAL Driver User Manual
