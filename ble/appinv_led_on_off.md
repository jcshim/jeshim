```
import bluetooth
import time
from micropython import const
from machine import Pin

# ----------------------------
# Pico W onboard LED
# ----------------------------
led = Pin("LED", Pin.OUT)
led.value(0)

# ----------------------------
# BLE IRQ event constants
# ----------------------------
_IRQ_CENTRAL_CONNECT    = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)
_IRQ_GATTS_WRITE        = const(3)

# ----------------------------
# UUIDs (지금 사용하시던 값 그대로)
# - Service: 0x181A (Environmental Sensing)
# - Char   : 0x2A6E (Temperature)
# ※ 테스트용으로는 커스텀 UUID가 더 깔끔하지만,
#   교수님 요청대로 기존 UUID 그대로 둡니다.
# ----------------------------
SERVICE_UUID = bluetooth.UUID("0000181A-0000-1000-8000-00805F9B34FB")
CHAR_UUID    = bluetooth.UUID("00002A6E-0000-1000-8000-00805F9B34FB")

# App Inventor가 Write(응답 필요/불필요) 둘 다 보낼 수 있으니 둘 다 허용
CHAR_FLAGS = (
    bluetooth.FLAG_WRITE |
    bluetooth.FLAG_WRITE_NO_RESPONSE |
    bluetooth.FLAG_READ
)

SERVICE = (SERVICE_UUID, ((CHAR_UUID, CHAR_FLAGS),),)

# ----------------------------
# Advertising payload (name + 128-bit service uuid)
# ----------------------------
def adv_payload(name: str, service_uuid: bluetooth.UUID):
    name_b = name.encode("utf-8")

    # Flags: LE General Discoverable + BR/EDR not supported
    payload = bytearray(b"\x02\x01\x06")

    # Complete Local Name
    payload += bytes((len(name_b) + 1, 0x09)) + name_b

    # Service UUID (128-bit list) if 128-bit
    su = bytes(service_uuid)
    if len(su) == 16:
        payload += bytes((len(su) + 1, 0x07)) + su

    return payload

# ----------------------------
# Command parser: handle '1'/'0' robustly
# ----------------------------
def parse_cmd(raw: bytes) -> bytes:
    """
    raw bytes에서 '1' 또는 '0' 명령을 최대한 안정적으로 추출.
    - 공백/개행 제거
    - 중간에 \x00 등이 끼어도 첫 바이트만 사용
    """
    if raw is None:
        return b""

    s = raw.strip()        # bytes에서도 동작: 공백/개행류 제거
    if not s:
        return b""
    return s[:1]           # 첫 1바이트만 명령으로 사용 (b'1' / b'0')

# ----------------------------
# BLE peripheral class
# ----------------------------
class PicoBLE:
    def __init__(self, name="PicoW-BLE"):
        self.ble = bluetooth.BLE()
        self.ble.active(True)
        self.ble.irq(self._irq)

        ((self.char_handle,),) = self.ble.gatts_register_services((SERVICE,))
        self.conn_handle = None

        self._adv = adv_payload(name, SERVICE_UUID)
        self._advertise()
        print(f"[{name}] advertising... (send '1' or '0' from phone)")

    def _advertise(self):
        # interval_us: 250ms
        self.ble.gap_advertise(250_000, adv_data=self._adv)

    def _irq(self, event, data):
        if event == _IRQ_CENTRAL_CONNECT:
            self.conn_handle, _, _ = data
            print(">> connected:", self.conn_handle)

        elif event == _IRQ_CENTRAL_DISCONNECT:
            print(">> disconnected")
            self.conn_handle = None
            self._advertise()

        elif event == _IRQ_GATTS_WRITE:
            conn_handle, value_handle = data
            if value_handle != self.char_handle:
                return

            raw = self.ble.gatts_read(self.char_handle)

            # 디버그: 실제로 들어온 바이트를 정확히 보기
            print("RAW repr =", repr(raw), "len=", len(raw))

            cmd = parse_cmd(raw)

            if cmd == b"1":
                led.value(1)
                print("LED ON")
            elif cmd == b"0":
                led.value(0)
                print("LED OFF")
            else:
                print("Unknown command. Send '1' or '0'. cmd=", cmd, "raw=", raw)

# ----------------------------
# Main
# ----------------------------
pico_ble = PicoBLE()

while True:
    time.sleep(1)

```
