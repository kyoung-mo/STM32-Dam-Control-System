# 🌊 STM32 Dam Control System

STM32F411 기반 스마트 댐 관리 시스템

## 📋 프로젝트 소개

이 프로젝트는 STM32F411 마이크로컨트롤러를 사용하여 댐의 수위를 실시간으로 모니터링하고 제어하는 임베디드 시스템입니다. 수위 센서, 온습도 센서, LCD 디스플레이, 키패드를 통합하여 댐의 상태를 효과적으로 관리합니다.

## ✨ 주요 기능

- **실시간 수위 모니터링**: ADC를 통한 정밀한 수위 측정
- **환경 감지**: DHT11 센서를 통한 온도 및 습도 측정
- **시각적 피드백**: I2C LCD를 통한 실시간 데이터 표시
- **사용자 인터페이스**: 4x4 키패드를 통한 설정 및 제어
- **데이터 로깅**: UART 통신을 통한 PC 연동 및 데이터 기록
- **자동 경보**: 설정된 임계값 초과 시 경고 알림

## 🔧 하드웨어 구성

| 구성요소 | 모델/사양 | 용도 |
|---------|----------|------|
| MCU | STM32F411CEU6 | 메인 컨트롤러 |
| 온습도 센서 | DHT11 | 환경 모니터링 |
| 수위 센서 | 아날로그 수위 센서 | 물 높이 측정 |
| 디스플레이 | I2C LCD 1602/2004 | 정보 표시 |
| 입력 장치 | 4x4 매트릭스 키패드 | 사용자 입력 |
| 통신 | UART (USB-TTL) | PC 연동 |

## 📁 프로젝트 구조
```
STM32-Dam-Control-System/
├── App/                        # 애플리케이션 레이어
│   ├── Inc/                    # 헤더 파일
│   │   ├── ap.h               # 메인 애플리케이션
│   │   ├── ap_def.h           # 공통 정의
│   │   ├── dht11.h            # DHT11 드라이버
│   │   ├── i2c-lcd.h          # LCD 드라이버
│   │   ├── keypad.h           # 키패드 드라이버
│   │   └── water_state_logger.h  # 데이터 로거
│   └── Src/                    # 소스 파일
│       ├── ap.c
│       ├── dht11.c
│       ├── i2c-lcd.c
│       ├── keypad.c
│       └── water_state_logger.c
├── Core/                       # HAL 초기화 레이어
│   ├── Inc/                    # HAL 헤더 파일
│   │   ├── main.h
│   │   ├── gpio.h
│   │   ├── i2c.h
│   │   ├── tim.h
│   │   ├── usart.h
│   │   └── adc.h
│   └── Src/                    # HAL 소스 파일
│       ├── main.c
│       ├── gpio.c
│       ├── i2c.c
│       ├── tim.c
│       ├── usart.c
│       └── adc.c
├── .gitignore
└── README.md
```

## 🚀 시작하기

### 필요 사항

- **하드웨어**
  - STM32F411 개발보드
  - DHT11 온습도 센서
  - 수위 센서
  - I2C LCD
  - 4x4 키패드
  - USB-TTL 컨버터

- **소프트웨어**
  - [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
  - [ST-Link 드라이버](https://www.st.com/en/development-tools/stsw-link009.html)
  - 시리얼 터미널 (PuTTY, Tera Term 등)

### 설치 및 빌드

1. **프로젝트 클론**
```bash
   git clone https://github.com/kyoung-mo/STM32-Dam-Control-System.git
```

2. **STM32CubeIDE에서 프로젝트 열기**
   - File → Open Projects from File System
   - 클론한 폴더 선택

3. **빌드 및 플래시**
   - Project → Build Project (Ctrl+B)
   - Run → Debug (F11) 또는 Run (Ctrl+F11)

### 하드웨어 연결

#### 핀 배치

| 핀 | 기능 | 연결 |
|----|------|------|
| PA0 | ADC1 | 수위 센서 출력 |
| PA9 | USART1_TX | USB-TTL RX |
| PA10 | USART1_RX | USB-TTL TX |
| PB6 | I2C1_SCL | LCD SCL |
| PB7 | I2C1_SDA | LCD SDA |
| PC0 | DHT11_DATA | DHT11 데이터 핀 |
| PB0-PB3 | KEYPAD_ROW | 키패드 행 |
| PB12-PB15 | KEYPAD_COL | 키패드 열 |

## 💻 사용 방법

### 시작 화면
시스템 전원을 켜면 LCD에 초기화 메시지가 표시됩니다.

### 메뉴 네비게이션
- `1`: 현재 수위 확인
- `2`: 온습도 확인
- `3`: 임계값 설정
- `#`: 메인 메뉴로 돌아가기

### UART 모니터링
시리얼 터미널 설정:
- 보레이트: 115200
- 데이터 비트: 8
- 정지 비트: 1
- 패리티: None

## 📊 주요 기능 상세

### 1. 수위 모니터링
- ADC 12bit 해상도로 정밀 측정
- 0-100% 범위로 환산하여 표시
- 임계값 초과 시 경고 발생

### 2. 온습도 측정
- DHT11 센서를 통한 주기적 측정
- 온도: 0-50°C
- 습도: 20-90%

### 3. 데이터 로깅
- 1초 간격으로 UART를 통해 데이터 전송
- CSV 형식으로 로그 저장 가능
- 형식: `Timestamp,WaterLevel,Temperature,Humidity`

## 🔍 트러블슈팅

### LCD가 표시되지 않는 경우
- I2C 주소 확인 (일반적으로 0x27 또는 0x3F)
- 배선 연결 상태 확인
- 대비 조절 가변저항 조정

### DHT11 센서 읽기 오류
- 데이터 핀 연결 확인
- 풀업 저항(10kΩ) 연결 확인
- 전원 공급 확인 (3.3V 또는 5V)

### UART 통신 안 됨
- 보레이트 설정 확인
- TX/RX 핀 교차 연결 확인
- USB-TTL 드라이버 설치 확인

## 📝 개발 환경

- **IDE**: STM32CubeIDE 1.10.0 이상
- **HAL Library**: STM32F4 HAL Driver
- **빌드 도구**: ARM GCC
- **디버거**: ST-Link V2

## 🤝 기여

프로젝트에 기여하고 싶으시다면:

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 라이센스

이 프로젝트는 MIT 라이센스 하에 배포됩니다.

## 👨‍💻 개발자

- **구영모** - [GitHub](https://github.com/kyoung-mo)

## 📧 연락처

프로젝트 관련 문의: kym1290306@gmail.com

## 🙏 감사의 글

- STMicroelectronics의 HAL 라이브러리
- DHT11 센서 라이브러리 참고 자료
- I2C LCD 드라이버 커뮤니티

---

⭐ 이 프로젝트가 도움이 되었다면 Star를 눌러주세요!
