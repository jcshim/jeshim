```
import bluetooth, time
from machine import ADC
from micropython import const

_IRQ_CENTRAL_CONNECT    = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)

# 내부 온도센서(RP2040): ADC4
adc_temp = ADC(4)
conv = 3.3 / 65535

def read_cpu_temp_c():
    v = adc_temp.read_u16() * conv
    return 27.0 - (v - 0.706) / 0.001721

# 표준 Environmental Sensing Service + Temperature Characteristic
SVC       = bluetooth.UUID("0000181A-0000-1000-8000-00805F9B34FB")
CHAR = bluetooth.UUID("00002A6E-0000-1000-8000-00805F9B34FB")  # Temperature
TEMP_CHAR = bluetooth.UUID("00002A6F-0000-1000-8000-00805F9B34FB")  # notify (새로)

SERVICE = (SVC, ((TEMP_CHAR, bluetooth.FLAG_NOTIFY),),)

def adv(name="PicoW-TEMP"):
    name_b = name.encode()
    payload = bytearray(b"\x02\x01\x06")                 # flags
    payload += bytes((len(name_b)+1, 0x09)) + name_b     # name
    payload += bytes((16+1, 0x07)) + bytes(SVC)          # service uuid
    ble.gap_advertise(250_000, adv_data=payload)

def irq(event, data):
    global conn
    if event == _IRQ_CENTRAL_CONNECT:
        conn, _, _ = data
    elif event == _IRQ_CENTRAL_DISCONNECT:
        conn = None
        adv()

ble = bluetooth.BLE()
ble.active(True)
ble.irq(irq)

((temp_handle,),) = ble.gatts_register_services((SERVICE,))
conn = None
adv()

# 1초마다 온도 Notify
while True:
    if conn is not None:
        t = read_cpu_temp_c()
        msg = ("T=%.1f\n" % t).encode('utf-8')   # 예: b"T=24.6"
        try:
            ble.gatts_notify(conn, temp_handle, msg)
        except:
            pass
    time.sleep(1)
```
<img width="1204" height="947" alt="image" src="https://github.com/user-attachments/assets/da06bd32-035f-45e4-8fc3-13de6385c7a4" />

