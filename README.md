# STM32 Dam Control System

STM32F411 기반 댐 관리 시스템 소스 코드

## 주요 기능
- 수위 센서를 통한 댐 수위 모니터링
- DHT11 센서를 통한 온습도 측정
- I2C LCD를 통한 실시간 정보 표시
- 키패드를 통한 사용자 입력
- UART 통신을 통한 데이터 로깅

## 하드웨어
- MCU: STM32F411
- 센서: DHT11 (온습도)
- 디스플레이: I2C LCD
- 입력: 4x4 키패드
- 통신: UART

## 프로젝트 구조
```
├── App/
│   ├── Src/          # 애플리케이션 소스 코드
│   └── Inc/          # 애플리케이션 헤더 파일
└── Core/
    ├── Src/          # 하드웨어 초기화 코드
    └── Inc/          # 하드웨어 헤더 파일
```

## 개발 환경
- IDE: STM32CubeIDE
- HAL Library: STM32F4 HAL Driver
