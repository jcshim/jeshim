```
import machine
import time

# UART 0번 설정 (GP12, GP13)
uart = machine.UART(0, baudrate=9600, tx=machine.Pin(12), rx=machine.Pin(13))

def parse_co_ppm(data):
    # [중요] 769 에러 해결: data[4], data[5]를 정확히 참조합니다.
    high = data[4]
    low = data[5]
    return ((high << 8) | low) * 0.1

print("--- ZE07-CO 테스트 모드 ---")
print("초를 끄고 연기를 센서 근처에 가져가 보세요.")

try:
    while True:
        if uart.any() >= 9:
            raw = uart.read(9)
            if raw[0] == 0xFF and raw[1] == 0x04:
                ppm = parse_co_ppm(raw)
                # Raw 데이터를 함께 보여주어 상태를 확인합니다.
                print(f"CO 농도: {ppm:>5.1f} PPM | Raw: {raw[4]:02x} {raw[5]:02x}")
        time.sleep(0.5)
except KeyboardInterrupt:
    print("\n중단됨")
```
