# PC에서 wifi로 접속하여 테스트 하려면
## Thonny에서
```
1. 코드를 복사 및 공유기 이름 바꾸고, 비번 넣기
SSID = 'shim'
PASSWORD = 'shimshim'
2. 실행
예) 접속 주소: 192.168.0.33 가 나오면
```
## PC 웹 브라우저에 링크를 복사하여 붙여 넣기

```
import network
import socket
import asyncio
import time
from machine import UART, Pin, I2C

# --- [1. 하드웨어 설정] ---
# Wi-Fi 설정 (기존 연결 정보를 유지하세요)
SSID = 'shim'
PASSWORD = 'shimshim'

# I2C 설정 (SHT31: GP20=SDA, GP21=SCL)
i2c = I2C(0, sda=Pin(20), scl=Pin(21), freq=100000)
SHT31_ADDR = 0x44
sht_rst = Pin(22, Pin.OUT)

# UART 설정 (PMS7003: UART1 GP8/9 | ZE07-CO: UART0 GP12/13)
pms_uart = UART(1, baudrate=9600, tx=Pin(8), rx=Pin(9), timeout=100)
co_uart = UART(0, baudrate=9600, tx=Pin(12), rx=Pin(13), timeout=100)
pms_set = Pin(7, Pin.OUT)
pms_set.value(1) # PMS 센서 활성화

# 데이터 저장소
env_data = {
    "temp": 0.0, "hum": 0.0,
    "pm1_0": 0, "pm2_5": 0, "pm10": 0,
    "co": 0.0, "p03": 0
}

# --- [2. 센서 제어 함수들] ---
def calc_crc8(data):
    crc = 0xFF
    for byte in data:
        crc ^= byte
        for _ in range(8):
            if crc & 0x80: crc = (crc << 1) ^ 0x31
            else: crc <<= 1
            crc &= 0xFF
    return crc

async def update_sht31():
    while True:
        try:
            i2c.writeto(SHT31_ADDR, b'\x24\x00')
            await asyncio.sleep_ms(20)
            data = i2c.readfrom(SHT31_ADDR, 6)
            if data[2] == calc_crc8(data[0:2]) and data[5] == calc_crc8(data[3:5]):
                env_data["temp"] = -45 + (175 * ((data[0] << 8) | data[1]) / 65535)
                env_data["hum"] = 100 * ((data[3] << 8) | data[4]) / 65535
        except: pass
        await asyncio.sleep(2)

async def update_pms():
    while True:
        if pms_uart.any() >= 32:
            raw = pms_uart.read(32)
            if raw[0] == 0x42 and raw[1] == 0x4d:
                if sum(raw[:30]) == ((raw[30] << 8) | raw[31]):
                    env_data["pm1_0"] = (raw[10] << 8) | raw[11]
                    env_data["pm2_5"] = (raw[12] << 8) | raw[13]
                    env_data["pm10"] = (raw[14] << 8) | raw[15]
                    env_data["p03"] = (raw[16] << 8) | raw[17]
        await asyncio.sleep(0.5)

async def update_co():
    while True:
        if co_uart.any() >= 9:
            raw = co_uart.read(9)
            if raw[0] == 0xff and raw[1] == 0x04:
                # 체크섬 검증 및 농도 계산 (data[4], data[5])
                check = ((~sum(raw[1:8]) + 1) & 0xFF)
                if check == raw[8]:
                    env_data["co"] = ((raw[4] << 8) | raw[5]) * 0.1
        await asyncio.sleep(0.5)

# --- [3. 웹 서버 설정] ---
def get_html():
    return f"""
    <html>
    <head>
        <meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
        <meta http-equiv="refresh" content="3">
        <title>Pico W Air Monitor</title>
        <style>
            body {{ font-family: 'Malgun Gothic', sans-serif; background: #eceff1; color: #37474f; text-align: center; }}
            .container {{ display: flex; flex-wrap: wrap; justify-content: center; padding: 20px; }}
            .card {{ background: white; margin: 10px; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 140px; }}
            .label {{ font-size: 0.9em; color: #78909c; }}
            .value {{ font-size: 1.8em; font-weight: bold; margin: 10px 0; color: #1e88e5; }}
            .unit {{ font-size: 0.8em; color: #90a4ae; }}
        </style>
    </head>
    <body>
        <h1>실시간 종합 환경 모니터</h1>
        <div class="container">
            <div class="card"><div class="label">온도</div><div class="value">{env_data['temp']:.1f}</div><div class="unit">°C</div></div>
            <div class="card"><div class="label">습도</div><div class="value">{env_data['hum']:.1f}</div><div class="unit">%</div></div>
            <div class="card"><div class="label">CO 농도</div><div class="value">{env_data['co']:.1f}</div><div class="unit">PPM</div></div>
            <div class="card"><div class="label">PM 1.0</div><div class="value">{env_data['pm1_0']}</div><div class="unit">µg/m³</div></div>
            <div class="card"><div class="label">PM 2.5</div><div class="value">{env_data['pm2_5']}</div><div class="unit">µg/m³</div></div>
            <div class="card"><div class="label">0.3µm 입자</div><div class="value">{env_data['p03']}</div><div class="unit">pcs</div></div>
        </div>
    </body>
    </html>
    """

async def serve_client(reader, writer):
    request = await reader.read(1024)
    response = "HTTP/1.1 200 OK\r\nContent-Type: text/html\r\n\r\n" + get_html()
    writer.write(response)
    await writer.drain()
    await writer.wait_closed()

# --- [4. 메인 실행] ---
async def main():
    # SHT31 리셋
    sht_rst.low(); time.sleep_ms(10); sht_rst.high()
    
    # 서버 시작
    server = await asyncio.start_server(serve_client, "0.0.0.0", 80)
    print("시스템 가동 중... 접속 주소: 192.168.0.33")
    
    await asyncio.gather(
        server.wait_closed(),
        update_sht31(),
        update_pms(),
        update_co()
    )

try:
    asyncio.run(main())
except:
    print("중단됨")


```
