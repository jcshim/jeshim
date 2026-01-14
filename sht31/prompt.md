I2C 방식으로 SHT31-DIS-B2.5KS가 rasberryPi-pico w에 다음과 같이 하드웨어로 연결된 경우, Thonnny에서 python으로 온도와 습도를 읽는 코드 작성을 부탁해.

SDA → GP20
SCL → GP21 
RST → GP22

[주의]
GP20/21 은 UART1 핀과도 겹치지만, MicroPython에서는 I2C로 정상 사용 가능합니다.
