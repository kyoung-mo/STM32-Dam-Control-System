# 🌊 STM32 Dam Control System

Smart Dam Management System based on STM32F411

## 📋 Project Overview

This project is an embedded system that uses the STM32F411 microcontroller to monitor and control dam water levels in real-time. It integrates water level sensors, temperature/humidity sensors, LCD displays, and keypads to effectively manage dam conditions.

## ✨ Key Features

- **Real-time Water Level Monitoring**: Precise water level measurement via ADC
- **Environmental Sensing**: Temperature and humidity measurement through DHT11 sensor
- **Visual Feedback**: Real-time data display via I2C LCD
- **User Interface**: Settings and control through 4x4 keypad
- **Data Logging**: PC connectivity and data recording via UART communication
- **Automatic Alarm**: Warning notifications when set thresholds are exceeded

## 🔧 Hardware Components

| Component | Model/Spec | Purpose |
|-----------|------------|---------|
| MCU | STM32F411CEU6 | Main Controller |
| Temperature/Humidity Sensor | DHT11 | Environmental Monitoring |
| Water Level Sensor | Analog Water Level Sensor | Water Height Measurement |
| Display | I2C LCD 1602/2004 | Information Display |
| Input Device | 4x4 Matrix Keypad | User Input |
| Communication | UART (USB-TTL) | PC Connectivity |

## 📁 Project Structure
```
STM32-Dam-Control-System/
├── docs/               
│   ├── README.md                      # ASPICE-based Design Documents
│   ├── requirements.md                # Requirements Specification
│   ├── architecture.md                # Architecture Design
│   ├── test-checklist.md              # Test Checklist
│   └── configuration-management.md    # Configuration Management Plan
│
├── App/                        # Application Layer
│   ├── Inc/                    # Header Files
│   │   ├── ap.h               # Main Application
│   │   ├── ap_def.h           # Common Definitions
│   │   ├── dht11.h            # DHT11 Driver
│   │   ├── i2c-lcd.h          # LCD Driver
│   │   ├── keypad.h           # Keypad Driver
│   │   └── water_state_logger.h  # Data Logger
│   └── Src/                    # Source Files
│       ├── ap.c
│       ├── dht11.c
│       ├── i2c-lcd.c
│       ├── keypad.c
│       └── water_state_logger.c
├── Core/                       # HAL Initialization Layer
│   ├── Inc/                    # HAL Header Files
│   │   ├── main.h
│   │   ├── gpio.h
│   │   ├── i2c.h
│   │   ├── tim.h
│   │   ├── usart.h
│   │   └── adc.h
│   └── Src/                    # HAL Source Files
│       ├── main.c
│       ├── gpio.c
│       ├── i2c.c
│       ├── tim.c
│       ├── usart.c
│       └── adc.c
├── .gitignore
└── README.md
```

## 🚀 Getting Started

### Prerequisites

- **Hardware**
  - STM32F411 Development Board
  - DHT11 Temperature/Humidity Sensor
  - Water Level Sensor
  - I2C LCD
  - 4x4 Keypad
  - USB-TTL Converter

- **Software**
  - [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)
  - [ST-Link Driver](https://www.st.com/en/development-tools/stsw-link009.html)
  - Serial Terminal (PuTTY, Tera Term, etc.)

### Installation and Build

1. **Clone the Project**
```bash
git clone https://github.com/kyoung-mo/STM32-Dam-Control-System.git
```

2. **Open Project in STM32CubeIDE**
   - File → Open Projects from File System
   - Select the cloned folder

3. **Build and Flash**
   - Project → Build Project (Ctrl+B)
   - Run → Debug (F11) or Run (Ctrl+F11)

### Hardware Connections

#### Pin Mapping

| Pin | Function | Connection |
|-----|----------|------------|
| PA0 | ADC1 | Water Level Sensor Output |
| PA9 | USART1_TX | USB-TTL RX |
| PA10 | USART1_RX | USB-TTL TX |
| PB6 | I2C1_SCL | LCD SCL |
| PB7 | I2C1_SDA | LCD SDA |
| PC0 | DHT11_DATA | DHT11 Data Pin |
| PB0-PB3 | KEYPAD_ROW | Keypad Rows |
| PB12-PB15 | KEYPAD_COL | Keypad Columns |

## 📊 Feature Details

### 1. Water Level Monitoring
- Precise measurement with 12-bit ADC resolution
- Conversion to 0-100% range for display
- Warning generation when threshold is exceeded

### 2. Temperature/Humidity Measurement
- Periodic measurement via DHT11 sensor
- Temperature: 0-50°C
- Humidity: 20-90%

### 3. Data Logging
- Data transmission via UART at 1-second intervals
- Logs can be saved in CSV format
- Format: `Timestamp,WaterLevel,Temperature,Humidity`

## 📚 Detailed Documentation

Detailed design documents for the project can be found in the `docs/` directory:

- [Requirements Specification](docs/requirements.md) - System/Software Requirements
- [Architecture Design](docs/architecture.md) - System Structure and Module Design
- [Test Checklist](docs/test-checklist.md) - Test Plans and Checklists
- [Configuration Management Plan](docs/configuration-management.md) - Git Operation Strategy

> 💡 These documents were created with reference to the ASPICE (Automotive SPICE) process.

## 🔍 Troubleshooting

### LCD Not Displaying
- Check I2C address (typically 0x27 or 0x3F)
- Verify wiring connections
- Adjust contrast potentiometer

### DHT11 Sensor Read Error
- Check data pin connection
- Verify pull-up resistor (10kΩ) connection
- Check power supply (3.3V or 5V)

### UART Communication Failed
- Verify baud rate settings
- Check TX/RX pin crossover connection
- Verify USB-TTL driver installation

## 📝 Development Environment

- **IDE**: STM32CubeIDE 1.10.0 or higher
- **HAL Library**: STM32F4 HAL Driver
- **Build Tool**: ARM GCC
- **Debugger**: ST-Link V2

## 🤝 Contributing

If you'd like to contribute to the project:

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is distributed under the MIT License.

## 👨‍💻 Developer

- **Youngmo Koo** - [GitHub](https://github.com/kyoung-mo)

## 📧 Contact

Project inquiries: kym11290306@gmail.com

## 🙏 Acknowledgments

- STMicroelectronics HAL Library
- DHT11 Sensor Library Reference Materials
- I2C LCD Driver Community

---

⭐ If this project was helpful, please give it a star!
