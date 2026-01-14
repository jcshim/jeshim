
## Co test file1

```
from machine import UART, Pin
import time

# 1. 센서 하드웨어 리셋 (GP2)
# 769 고정 문제를 해결하기 위해 전원을 껐다 켭니다.
co_pwr = Pin(2, Pin.OUT)
print("센서 하드웨어 초기화 중...")
co_pwr.value(0) 
time.sleep(2)
co_pwr.value(1)
print("센서 전원 공급 완료. 예열 시작 (약 3분 권장)")

# 2. UART 설정 (하드웨어 UART0, GP12=TX, GP13=RX)
uart = UART(0, baudrate=9600, tx=Pin(12), rx=Pin(13), bits=8, parity=None, stop=1)

def get_co_data():
    if uart.any() >= 9:
        raw = uart.read(9)
        if raw[0] == 0xFF and raw[1] == 0x04:
            # 체크섬 검증
            checksum = ((~sum(raw[1:8])) & 0xFF) + 1
            if checksum == raw[8]:
                # 769ppm 여부 확인을 위한 인덱스 2, 3 사용
                ppm = (raw[2] << 8) | raw[3]
                return ppm, raw
    return None, None

# 3. 메인 루프
start_time = time.time()
while True:
    ppm, data = get_co_data()
    
    if ppm is not None:
        elapsed = time.time() - start_time
        hex_str = " ".join("{:02x}".format(x) for x in data)
        print(f"[{elapsed}초 경과] CO: {ppm} ppm | Raw: {hex_str}")
        
        # 만약 769에서 단 1이라도 변한다면 센서는 정상입니다.
        if ppm != 769:
            print(">>> 신호 변화 감지! 안정화 진행 중...")
            
    time.sleep(2)
```
