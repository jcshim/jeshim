## App Inventor <- BLE -> Pico-W

### Import extension: BluetoothLE 


<img width="757" height="282" alt="image" src="https://github.com/user-attachments/assets/aab1af06-27ec-4bac-bc7a-ba954c42f7b9" />

```
import bluetooth
import aioble

_SERVICE_UUID = bluetooth.UUID("0000181A-0000-1000-8000-00805F9B34FB")
_WRITE_CHAR_UUID = bluetooth.UUID("00002A6E-0000-1000-8000-00805F9B34FB")
_READ_CHAR_UUID = bluetooth.UUID("00002A6F-0000-1000-8000-00805F9B34FB")
```

<img width="533" height="210" alt="image" src="https://github.com/user-attachments/assets/32550b5d-cbd6-4209-8e7d-1471fd1722d8" />

```
# Pico W BLE: receive '1'/'0' and control onboard LED (minimal)
import bluetooth, time
from machine import Pin
from micropython import const

_IRQ_CENTRAL_CONNECT    = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)
_IRQ_GATTS_WRITE        = const(3)

led = Pin("LED", Pin.OUT)
led.value(0)

# 커스텀 UUID (App Inventor에도 그대로 입력)
SVC  = bluetooth.UUID("0000181A-0000-1000-8000-00805F9B34FB")
CHAR = bluetooth.UUID("00002A6E-0000-1000-8000-00805F9B34FB")

SERVICE = (SVC, ((CHAR, bluetooth.FLAG_WRITE | bluetooth.FLAG_WRITE_NO_RESPONSE),),)

def adv(name="PicoW"):
    name_b = name.encode()
    payload = bytearray(b"\x02\x01\x06")                          # flags
    payload += bytes((len(name_b)+1, 0x09)) + name_b              # name
    payload += bytes((16+1, 0x07)) + bytes(SVC)                   # 128-bit svc uuid
    ble.gap_advertise(250_000, adv_data=payload)

def cmd_from(raw: bytes):
    # App Inventor가 b'1\x00', b'0\r\n'처럼 보내도 첫 글자만 사용
    raw = raw.strip()
    return raw[:1] if raw else b""

def irq(event, data):
    global conn
    if event == _IRQ_CENTRAL_CONNECT:
        conn, _, _ = data
    elif event == _IRQ_CENTRAL_DISCONNECT:
        conn = None
        adv()
    elif event == _IRQ_GATTS_WRITE:
        _, h = data
        if h == ch:
            c = cmd_from(ble.gatts_read(ch))
            if c == b"1":
                led.value(1)
            elif c == b"0":
                led.value(0)

ble = bluetooth.BLE()
ble.active(True)
ble.irq(irq)

((ch,),) = ble.gatts_register_services((SERVICE,))
conn = None

adv("PicoW-LED")

while True:
    time.sleep(1)

```
<img width="680" height="359" alt="image" src="https://github.com/user-attachments/assets/fb944022-c9a7-420f-84fc-f51469c3afa4" />

<img width="1147" height="609" alt="image" src="https://github.com/user-attachments/assets/0fb7b997-fc3f-42e0-9537-919f24f36982" />
