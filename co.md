```
from machine import UART, Pin
import time

# 1. 전원 제어 설정 (GP2)
power_en = Pin(2, Pin.OUT)
power_en.value(1) 

# 2. UART 설정 변경
# 하드웨어 UART0 대신 핀을 유연하게 할당하기 위해 
# 구버전에서는 UART(0, ...), 최신 펌웨어에서는 명시적 핀 지정을 사용합니다.
# 만약 여전히 에러가 난다면 UART 번호를 1로 바꿔보거나 아래 형식을 시도하세요.
try:
    # GP11을 TX로, GP12를 RX로 사용하는 UART1 설정
    uart = UART(1, baudrate=9600, tx=Pin(11), rx=Pin(12))
except ValueError:
    # 하드웨어 제약으로 실패할 경우를 대비한 대체 코드 (Pico 펌웨어 버전에 따라 다름)
    uart = UART(0, baudrate=9600, tx=Pin(12), rx=Pin(13)) # 회로 수정이 필요할 수 있음
    print("UART 설정 오류: 핀 연결을 다시 확인해주세요.")

def get_co_ppm():
    if uart.any():
        data = uart.read(9)
        if data and len(data) == 9 and data[0] == 0xFF and data[1] == 0x04:
            checksum = (~(data[1] + data[2] + data[3] + data[4] + data[5] + data[6] + data[7]) & 0xFF) + 1
            if checksum == data[8]:
                return (data[2] * 256) + data[3]
    return None

print("ZE07-CO 센서 읽기 시작...")

while True:
    co_value = get_co_ppm()
    if co_value is not None:
        print(f"현재 CO 농도: {co_value} ppm")
    time.sleep(1)
```
