```
from machine import Pin, I2C
import time

# 핀 설정: SDA=GP20, SCL=GP21
i2c = I2C(0, scl=Pin(21), sda=Pin(20), freq=400000)  # I2C0 사용

SHT31_ADDR = 0x44      # ADDR=GND 기준
CMD_SINGLE_SHOT_HIGH = b'\x24\x00'  # single shot, high repeatability 명령[web:28]

# (옵션) RST 핀 제어: GP22
rst = Pin(22, Pin.OUT)
rst.value(1)          # 기본은 high
time.sleep_ms(10)
# 필요하면 센서 리셋
# rst.value(0); time.sleep_ms(1); rst.value(1); time.sleep_ms(10)


def read_sht31():
    # 측정 명령 전송
    i2c.writeto(SHT31_ADDR, CMD_SINGLE_SHOT_HIGH)
    time.sleep_ms(20)  # 데이터 준비 시간 대기[web:28]

    # 6바이트 읽기: T(2) + CRC(1) + RH(2) + CRC(1)[web:28]
    data = i2c.readfrom(SHT31_ADDR, 6)
    t_raw = (data[0] << 8) | data[1]
    h_raw = (data[3] << 8) | data[4]

    # 데이터시트 공식: T = -45 + 175 * raw / 65535, RH = 100 * raw / 65535[web:28]
    temperature = -45 + (175 * t_raw / 65535.0)
    humidity = 100 * h_raw / 65535.0
    return temperature, humidity


while True:
    try:
        temp, hum = read_sht31()
        print("Temperature: %.2f C, Humidity: %.2f %%" % (temp, hum))
    except OSError as e:
        print("I2C Error:", e)
    time.sleep(1)
```

