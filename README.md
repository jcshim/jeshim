# jeshim
Jooeun Shim, Air Quality Module

<img width="1024" height="747" alt="image" src="https://github.com/user-attachments/assets/05b509ce-eab8-41f5-9c4e-9f67ec35eee9" />
[리셋버튼 흰색]
<img width="1024" height="747" alt="image" src="https://github.com/user-attachments/assets/4ff7faad-7eb6-4086-8ac6-9195c8c09cd7" />

[RaspberryPi pico-W Data Sheet](https://datasheets.raspberrypi.com/picow/connecting-to-the-internet-with-pico-w.pdf)

<img width="1305" height="848" alt="image" src="https://github.com/user-attachments/assets/32c4f1f8-f517-4b0e-ba7e-fa0b48587ec2" />

## RaspberryPI pico-W 포함된 Air Qulity Module Test

요청하신 내용을 바탕으로 깔끔하게 정리된 **Markdown** 가이드입니다. 나중에 다시 보시거나 다른 분들께 공유하기 좋게 구조화했습니다.

---

# 🌬️ Air Quality Sensor 테스트 가이드

## 1. 하드웨어 초기화 및 연결

* **부팅 모드 진입**: Pico W의 **리셋(흰색 BOOTSEL 버튼)**을 누른 상태에서 USB 케이블을 PC에 연결합니다.
* **연결 확인**: PC에서 '딩동' 소리와 함께 탐색기에 새로운 드라이브(`RPI-RP2`)가 나타납니다.

## 2. 펌웨어 설치

* **다운로드**: [Raspberry Pi 공식 사이트](https://www.raspberrypi.com/documentation/microcontrollers/micropython.html)에서 `RPI_PICO_W_xx.uf2` 파일을 다운로드합니다.
* **복사 및 설치**: 다운로드한 파일을 탐색기의 Pico 드라이브로 복사합니다.
* **완료**: 복사가 끝나면 드라이브가 자동으로 탐색기에서 사라지며 MicroPython 설치가 완료됩니다.

## 3. 개발 환경(Thonny) 설정

1. **설치**: [Thonny 공식 홈페이지](https://thonny.org/)에서 프로그램을 다운로드하여 설치합니다.
2. **실행**: 설치된 Thonny를 실행합니다.
3. **인터프리터 선택**: 화면 오른쪽 하단에서 **MicroPython (Raspberry Pi Pico)**를 클릭하여 연결을 활성화합니다.

## 4. 첫 번째 테스트: LED 깜박이기

* **코드 생성**: Gemini에게 아래와 같이 요청합니다.
> "raspberrypi pico-w에서 Thonny로 LED 깜박이는 Python 코드 작성해줘"


* **실행**: 생성된 코드를 복사하여 Thonny 에디터에 붙여넣고, 상단의 **푸른색 실행 버튼(Play)**을 누릅니다.

## 5. 센서별 상세 테스트

[jcshim/jeshim GitHub 저장소](https://github.com/jcshim/jeshim)의 소스코드를 활용하여 테스트를 진행합니다.

### ① 온도/습도 테스트 (SHT31)

* **방법**: Thonny에서 새 파일을 만들고 소스코드를 복사한 뒤 실행합니다.
* **참고**: [온도 테스트 결과 확인](https://github.com/jcshim/jeshim/blob/main/sht31/1_%ED%85%8C%EC%8A%A4%ED%8A%B8%EA%B2%B0%EA%B3%BC.md)

### ② 일산화탄소 테스트 (ZE07-CO)

* **방법**: UART 통신을 통해 CO 농도가 정상적으로 출력되는지 확인합니다.
* **참고**: [CO 작동 성공 사례](https://github.com/jcshim/jeshim/blob/main/co/0%EC%9E%91%EB%8F%99%EC%84%B1%EA%B3%B5.md)

### ③ 미세먼지 테스트 (PMS7003)

* **방법**: 미세먼지 수치(PM1.0, PM2.5, PM10)가 실시간으로 변하는지 확인합니다.
* **참고**: [먼지 센서 작동 성공 사례](https://github.com/jcshim/jeshim/blob/main/dust/0%EC%9E%91%EB%8F%99%EC%84%B1%EA%B3%B5.md)

---

**팁**: 모든 센서가 정상 작동한다면, 앞서 작성해 드린 **통합 웹 서버 코드**를 사용하여 무선으로 모니터링해 보세요!

다음 단계로 이 가이드에 **회로 연결도(Fritzing)**나 **주의사항**을 추가해 드릴까요?
