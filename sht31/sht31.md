```from machine import UART, Pin
import time

def init_co_sensor():
    uart = UART(0, baudrate=9600, tx=Pin(12), rx=Pin(13))
    print("ZE07-CO 센서 초기화")
    time.sleep(3)
    return uart

def read_co_ppm(uart):
    if uart.any():
        data = uart.read(9)
        if (len(data) == 9 and data[0] == 0xFF and 
            data[1] == 0x04 and data[8] == ((~sum(data[1:8]) + 1) % 256)):
            return ((data[3] << 8) | data[4]) / 10.0
    return None

# 메인 루프
uart = init_co_sensor()
last_ppm = 0

print("CO 모니터링 시작 (Ctrl+C 중단)")
try:
    while True:
        ppm = read_co_ppm(uart)
        if ppm is not None:  # None 체크 추가!
            #if ppm != last_ppm:
            print(f"CO: {ppm:.1f} ppm")
            last_ppm = ppm
            
            if ppm > 50:  # ppm이 None이 아닌 상태에서만 비교
                print("*** CO 경고: 환기 필요 ***")
        
        time.sleep_ms(1000)
except KeyboardInterrupt:
    print("모니터링 중단")


```

