# 요구사항 명세서 (Requirements Specification)

**프로젝트**: STM32 Dam Control System  
**버전**: 1.0  
**작성일**: 2025-01-31

---

## 1. 시스템 요구사항 (SYS.2)

### 1.1 기능 요구사항

#### FR-01: 수위 모니터링
- 시스템은 ADC를 통해 수위 센서 값을 읽어야 한다
- 수위는 0-100% 범위로 환산되어 표시되어야 한다
- 수위 측정 주기는 1초 이하여야 한다

#### FR-02: 환경 감지
- 시스템은 DHT11 센서를 통해 온도와 습도를 측정해야 한다
- 온도 측정 범위: 0-50°C
- 습도 측정 범위: 20-90%

#### FR-03: 사용자 인터페이스
- LCD를 통해 현재 상태를 실시간으로 표시해야 한다
- 4x4 키패드를 통해 메뉴 탐색이 가능해야 한다
- 키패드 입력:
  - `1`: 수위 확인
  - `2`: 온습도 확인
  - `3`: 임계값 설정
  - `#`: 메인 메뉴 복귀

#### FR-04: 데이터 로깅
- UART를 통해 PC로 데이터를 전송해야 한다
- 로그 형식: Timestamp, WaterLevel, Temperature, Humidity
- 전송 주기: 1초

#### FR-05: 경보 기능
- 설정된 임계값 초과 시 경고를 표시해야 한다
- 경고는 LCD와 UART 모두에 출력되어야 한다

### 1.2 비기능 요구사항

#### NFR-01: 성능
- 센서 읽기 응답 시간 < 100ms
- 키패드 입력 처리 시간 < 50ms
- LCD 업데이트 주기 ≤ 500ms

#### NFR-02: 신뢰성
- 센서 읽기 실패 시 재시도 메커니즘 필요
- 이전 정상값 유지 (센서 오류 시)

#### NFR-03: 유지보수성
- 모듈화된 드라이버 구조
- 코드 주석 포함

---

## 2. 소프트웨어 요구사항 (SWE.1)

### 2.1 DHT11 드라이버 (dht11.c/h)

**SW-DHT-01**: DHT11 초기화 함수 제공
- 함수: `DHT11_Init()`
- GPIO 핀 설정 및 타이머 초기화

**SW-DHT-02**: 온습도 읽기 함수 제공
- 함수: `DHT11_Read(float* temp, float* humi)`
- 반환: 성공(1) / 실패(0)
- 타임아웃: 2초

**SW-DHT-03**: 통신 프로토콜 준수
- DHT11 데이터 프로토콜 (40bit) 구현
- 체크섬 검증 수행

### 2.2 I2C LCD 드라이버 (i2c-lcd.c/h)

**SW-LCD-01**: LCD 초기화
- 함수: `LCD_Init()`
- I2C 주소 설정 (0x27 또는 0x3F)
- 4bit 모드 초기화

**SW-LCD-02**: 문자열 출력
- 함수: `LCD_Print(char* str)`
- 함수: `LCD_SetCursor(uint8_t row, uint8_t col)`
- 함수: `LCD_Clear()`

**SW-LCD-03**: 제어 명령
- 백라이트 on/off
- 디스플레이 clear

### 2.3 키패드 드라이버 (keypad.c/h)

**SW-KEY-01**: 키패드 스캔
- 함수: `Keypad_Scan()`
- 반환: 눌린 키 값 (0-9, A-D, *, #)
- 디바운싱 처리 (50ms)

**SW-KEY-02**: 매트릭스 스캔 방식
- 4x4 매트릭스 행/열 스캔
- 동시 입력 방지

### 2.4 데이터 로거 (water_state_logger.c/h)

**SW-LOG-01**: UART 전송
- 함수: `Logger_Send(WaterState_t* state)`
- 보레이트: 115200
- 형식: CSV (Timestamp,Level,Temp,Humi)

**SW-LOG-02**: 데이터 구조체
```c
typedef struct {
    uint32_t timestamp;
    float water_level;
    float temperature;
    float humidity;
} WaterState_t;
```

### 2.5 애플리케이션 레이어 (ap.c/h)

**SW-APP-01**: 메인 루프
- 센서 읽기 → 데이터 처리 → LCD 출력 → UART 전송
- 주기: 1초

**SW-APP-02**: 상태 머신
- IDLE / MONITORING / SETTING / ALARM 상태 관리

**SW-APP-03**: 임계값 관리
- 기본값: 80%
- 사용자 설정 가능 (10-100% 범위)

---

## 3. 인터페이스 요구사항

### 3.1 하드웨어 인터페이스

| 인터페이스 | 핀 | 프로토콜 | 설정 |
|-----------|-------|---------|------|
| 수위 센서 | PA0 | ADC1 | 12bit, 연속 변환 |
| DHT11 | PC0 | 단선 통신 | 양방향 GPIO |
| LCD | PB6/PB7 | I2C | 100kHz, 7bit 주소 |
| 키패드 | PB0-PB15 | GPIO | 행: 출력, 열: 입력(풀업) |
| UART | PA9/PA10 | UART1 | 115200 8N1 |

### 3.2 소프트웨어 인터페이스

**모듈 간 함수 호출**:
```
ap.c → dht11.c: DHT11_Read()
ap.c → i2c-lcd.c: LCD_Print()
ap.c → keypad.c: Keypad_Scan()
ap.c → water_state_logger.c: Logger_Send()
ap.c → HAL: HAL_ADC_GetValue(), HAL_UART_Transmit()
```

---

## 4. 요구사항 추적 매트릭스

| 요구사항 ID | 구현 모듈 | 테스트 항목 |
|------------|----------|------------|
| FR-01 | ap.c (ADC 처리) | TC-01 |
| FR-02 | dht11.c | TC-02 |
| FR-03 | i2c-lcd.c, keypad.c | TC-03 |
| FR-04 | water_state_logger.c | TC-04 |
| FR-05 | ap.c (경보 로직) | TC-05 |
| SW-DHT-01~03 | dht11.c | TC-02 |
| SW-LCD-01~03 | i2c-lcd.c | TC-03 |
| SW-KEY-01~02 | keypad.c | TC-03 |
| SW-LOG-01~02 | water_state_logger.c | TC-04 |
| SW-APP-01~03 | ap.c | TC-01, TC-05 |

---

## 5. 제약사항

### 5.1 하드웨어 제약
- MCU: STM32F411CEU6 (512KB Flash, 128KB RAM)
- 동작 전압: 3.3V
- 최대 클럭: 100MHz

### 5.2 소프트웨어 제약
- HAL 라이브러리 사용 필수
- FreeRTOS 미사용 (베어메탈)
- 전역 변수 최소화

### 5.3 환경 제약
- 동작 온도: 0-50°C
- 실내 사용 가정
