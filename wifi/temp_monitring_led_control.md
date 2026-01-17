```
import rp2
import network
import ubinascii
import machine
import time
import socket

# 1. 온도 센서 설정 (ADC 4번 채널)
sensor_temp = machine.ADC(4)
conversion_factor = 3.3 / (65535)

# 국가 설정 및 Wi-Fi 연결
rp2.country('KR')
wlan = network.WLAN(network.STA_IF)
wlan.active(True)

ssid = 'shim'
pw = 'shimshim'
wlan.connect(ssid, pw)

# 연결 대기
timeout = 10
while timeout > 0:
    if wlan.status() < 0 or wlan.status() >= 3:
        break
    timeout -= 1
    print('Waiting for connection...')
    time.sleep(1)

if wlan.status() != 3:
    raise RuntimeError('Wi-Fi connection failed')
else:
    status = wlan.ifconfig()
    print('Connected, IP = ' + status[0])

# 내장 LED 설정
led = machine.Pin('LED', machine.Pin.OUT)

# 서버 설정 (IP 주소는 자동으로 할당된 것을 사용하도록 수정)
addr = socket.getaddrinfo('0.0.0.0', 80)[0][-1]
s = socket.socket()
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1) # 소켓 재사용 설정
s.bind(addr)
s.listen(1)

print('Listening on', addr)

# 2. 온도 읽기 함수
def get_temperature():
    reading = sensor_temp.read_u16() * conversion_factor
    # 온도 변환 공식 (Pico 데이터시트 기준: 27도일 때 0.706V)
    temperature = 27 - (reading - 0.706) / 0.001721
    return round(temperature, 2)

while True:
    try:
        cl, addr = s.accept()
        request = cl.recv(1024)
        request = str(request)
        
        # LED 제어 로직
        if request.find('/?led=on') > -1:
            led.value(1)
        if request.find('/?led=off') > -1:
            led.value(0)
            
        # 현재 온도 가져오기
        temp = get_temperature()
        
        # HTML 응답 생성 (별도 파일 없이 직접 생성)
        response = f"""
        <!DOCTYPE html>
        <html>
            <head>
                <meta charset="utf-8">
                <meta name="viewport" content="width=device-width, initial-scale=1">
                <title>Pico W Control</title>
            </head>
            <body>
                <h1>Pico W Web Server</h1>
                <p>현재 온도: <strong>{temp} &deg;C</strong></p>
                <p>LED 제어:
                    <a href="/?led=on"><button>ON</button></a>
                    <a href="/?led=off"><button>OFF</button></a>
                </p>
                <button onclick="location.reload()">온도 업데이트</button>
            </body>
        </html>
        """
        
        cl.send('HTTP/1.0 200 OK\r\nContent-type: text/html\r\n\r\n')
        cl.send(response)
        cl.close()
        
    except Exception as e:
        print('Error:', e)
        cl.close()
```
