# 일산화탄소(CO) 센서 — ZE07-CO

## 📌 모델 정보
- **모델명**: ZE07-CO  
- **측정 대상**: 일산화탄소(CO)

---

## 🔌 통신 방식
- **통신 방식**: UART  
- **Baudrate**: 9600 bps  
- **프레임 길이**: 총 9바이트  

### 데이터 구조 예시
| 바이트 | 내용 |
|--------|------|
| Byte 0 | Start Byte = `0xFF` |
| Byte 1 | Command |
| Byte 2 | High Byte (CO 값 상위 바이트) |
| Byte 3 | Low Byte (CO 값 하위 바이트) |
| Byte 4~7 | Reserved |
| Byte 8 | Checksum |

> CO 값(ppm) = `(Byte2 << 8) | Byte3`

---

## 📍 핀 구성 및 연결

### 🔗 연결 요약

| Raspberry Pi Pico W | GPIO | Pin No | 신호 | ZE07-CO | Pin | 기능 |
|----------------------|------|---------|--------|-----------|------|---------|
| TX | GP12 | 16 | CO_RXD | RXD | 7 | Pico → 센서 데이터 전송 |
| RX | GP13 | 17 | CO_TXD | TXD | 8 | 센서 → Pico 데이터 수신 |
| +5V | VBUS / External | - | VCC | VIN | 15 | 센서 전원 |
| GND | GND | - | GND | GND | 5 / 14 | 공통 접지 |

---

## 🔁 연결 상세

```text
Raspberry Pi Pico W TX (GP12, Pin 16)  <->  ZE07-CO RXD (Pin 7)
Raspberry Pi Pico W RX (GP13, Pin 17)  <->  ZE07-CO TXD (Pin 8)
```

## CO 농도 기준

<img width="691" height="288" alt="image" src="https://github.com/user-attachments/assets/44b68304-67c9-4e66-876d-c73f2f7000a8" />
