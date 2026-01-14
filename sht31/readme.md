<img width="825" height="517" alt="image" src="https://github.com/user-attachments/assets/84b97933-0d39-45d3-ab71-1c79d7043015" />
<img width="355" height="75" alt="image" src="https://github.com/user-attachments/assets/b2401bea-e36b-4cd8-a4bf-85ed2c24e0c2" />

I2C 방식으로 SHT31-DIS-B2.5KS가 rasberryPi-pico w에 다음과 같이 하드웨어로 연결된 경우, Thonnny에서 python으로 온도와 습도를 읽는 코드 작성을 부탁해.
```
[SHT31] SDA → [rasberryPi-pico w] GP20
[SHT31] SCL → [rasberryPi-pico w] GP21 
[SHT31] RST → [rasberryPi-pico w] GP22
```
[주의]
GP20/21 은 UART1 핀과도 겹치지만, MicroPython에서는 I2C로 정상 사용 가능합니다.
