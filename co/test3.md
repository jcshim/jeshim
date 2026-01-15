```
import machine
import time

# 하드웨어 UART 0 (GP12, GP13)
uart = machine.UART(0, baudrate=9600, tx=machine.Pin(12), rx=machine.Pin(13))

def get_co_ppm(data):
    # 769 에러를 방지하기 위해 data[4], data[5]를 읽습니다.
    # data[2], [3]은 센서 상태값이므로 무시해야 합니다.
    high = data[4]
    low = data[5]
    return ((high << 8) | low) * 0.1

try:
    while True:
        if uart.any() >= 9:
            raw = uart.read(9)
            if raw[0] == 0xFF and raw[1] == 0x04:
                ppm = get_co_ppm(raw)
                print(f"현재 CO 농도: {ppm:.1f} PPM (안정적)")
        time.sleep(1)
except KeyboardInterrupt:
    print("중단됨")
```
