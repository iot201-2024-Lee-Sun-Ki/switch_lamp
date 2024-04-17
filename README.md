swith_lamp
==========

### About

This is your project's README.md file. It helps users understand what your
project does, how to use it and anything else they may need to know.



https://github.com/iot201-2024-Lee-Sun-Ki/switch_lamp/assets/141006849/539df8fc-56c5-4d72-80c2-547e6957acd9

```ruby


from IO7FuPython import Device, ConfiguredDevice
import json, time, ComMgr
from machine import Pin

def handleCommand(topic, msg):
    global lastPub
    jo = json.loads(str(msg, 'utf8'))
    if "switch" in jo['d']:
        switch = jo['d']['switch']
        if switch == 'current':  # 'is' 대신 '==' 사용
            lastPub = -device.meta['pubInterval']

# WiFi 설정 및 디바이스 초기화
nic = ComMgr.startWiFi('io7lux')
device = ConfiguredDevice()
device.setUserCommand(handleCommand)  # 핸들러 설정
device.connect()

lastPub = time.ticks_ms() - device.meta['pubInterval']

# 버튼 핀 설정 (풀업 저항 사용)
button_pin_on = Pin(22, Pin.IN, Pin.PULL_UP)
button_pin_off = Pin(23, Pin.IN, Pin.PULL_UP)

# 초기 버튼 상태 설정
last_state_on = 1  # 눌리지 않은 상태 (풀업 저항 때문에 1)
last_state_off = 1

while True:
    device.loop()
    # 버튼 상태 읽기
    current_state_on = button_pin_on.value()
    current_state_off = button_pin_off.value()

    # "on" 버튼 눌림 감지
    if current_state_on != last_state_on:
        last_state_on = current_state_on
        if current_state_on == 0:
            print("Button ON pressed")
            device.publishEvent('status', json.dumps({'d': {'switch': "on"}}))
            time.sleep(0.1)  # 디바운싱을 위한 딜레이

    # "off" 버튼 눌림 감지
    if current_state_off != last_state_off:
        last_state_off = current_state_off
        if current_state_off == 0:
            print("Button OFF pressed")
            device.publishEvent('status', json.dumps({'d': {'switch': "off"}}))
            time.sleep(0.1)  # 디바운싱을 위한 딜레이

    # CPU 사용량 관리를 위해 짧게 대기
    time.sleep(0.05)

```

```python
from machine import Pin
import json
import ComMgr
from IO7FuPython import Device, ConfiguredDevice

# 릴레이를 연결할 핀 번호 설정, GPIO 15번 핀을 사용
relay_pin = Pin(15, Pin.OUT)
relay_pin.value(0)  # 초기 상태는 꺼진 상태 (0)

def handleCommand(topic, msg):
    print("Received a message on topic: {}".format(topic))
    jo = json.loads(msg.decode('utf-8'))  # 메시지 디코딩 및 JSON 파싱
    if 'lamp' in jo['d']:
        command = jo['d']['lamp']
        if command == 'on':
            relay_pin.value(1)  # 릴레이 ON
            print("Relay turned ON")
        elif command == 'off':
            relay_pin.value(0)  # 릴레이 OFF
            print("Relay turned OFF")

# WiFi 설정 및 디바이스 초기화
nic = ComMgr.startWiFi('io7lux')
device = ConfiguredDevice()
device.setUserCommand(handleCommand)  # 핸들러 설정
device.connect()

while True:
    device.loop()
    # 주기적으로 네트워크 이벤트 체크, 메시지 처리는 handleCommand 함수에서 수행됨
```

