# Android Bluetooth

## Android BLE Connect cycle에 대해서 설명해주세요.
1. BluetoothManager를 통해서 BluetoothAdapter를 가져온다.
2. Bluetooth 관련 Permission 체크를 진행한다.
3. 체크가 완료되었다면 1번에서 가져온 BluetoothAdapter 객체에서 BluetoothLeScanner 객체를 가져옵니다.
4. BluetoothLeScanner를 통해서 주변 BLE 기기를 Scanning 합니다. (필요하다면 Scanning 과정에 특정 UUID 혹은 필요한 설정을 추가하여 기기를 Scan 합니다.)
5. 미리 정의한 ScanCallback 객체를 통해 Scanning된 BLE 주변 기기를 ScanResult 객체로 가져올 수 있습니다.
6. 특정 ScanResult 객체와 연결하기 위해 connectGatt() 함수를 호출합니다.
7. 미리 정의한 BluetoothGattCallback으로 연결 상태에 대한 정보를 얻을 수 있습니다.
8. 성공적으로 연결되었다면, BluetoothGatt 객체의 Service를 가져옵니다.
9. BluetoothGatt의 Service 객체를 통해서 특정 UUID의 Characteristic 정보를 얻을 수 있고 Characteristic 정보를 통해서 BLE 기기의 값을 읽고 쓸 수 있습니다.

<br><br>

## Bluetooth GATT란 무엇인가요?
BLE 연결을 통해서 프로필 및 데이터를 주고 받기 위한 방법을 정의한 방법론이다. GATT는 실제 데이터 전송 절차 및 데이터 형식만을 다룬다. 올바르게 작동하려면 모든 표준 BLE Profile은 GATT를 기반으로 해야 하고 준수해야 한다.

### GATT Service
GATT는 계층적으로 구성되어 있다. Service는 일반적으로 장치의 특징을 설명하는 특성 모음이다. Service 안에 장치의 일련 번호, 배터리 레벨을 나타내는 특성 등이 포함될 수 있다.

### GATT Characteristic
일반적으로 일련 번호 문자열 특성으로부터 읽거나 쓸 수 있는 의미 있는 데이터를 포함하는 Entity이다.

### GATT 특징 및 필요성
- GATT라는 계층적으로 구성되어 있는 데이터 Entity를 통해서 모든 BLE 기기 관련 데이터셋을 표준화 할 수 있다. (개발자는 기기마다 다른 데이터셋을 고민하지 않고 오직 GATT를 기반으로 통신을 구현한다.)
- 계층적으로 구성되어 있다.
- 특정 Entity의 Key를 읽고 데이터를 0 -> 1 로 변경하는 등의 작업을 통해 BLE 기기에 데이터를 변경할 수 있다.

<br><br>

## BLE와 Bluetooth Classic의 차이점

### 에너지 효율
가장 큰 차이점은 Low Energy이다. Bluetooth Classic은 음악 재생과 같이 연속적인 데이터 스트림을 전송하도록 설계되었다.  
반면에, BLE는 전력 효율성에 최적화 되어 있다. 필요한 경우 적절하게 데이터를 보내기 때문에 배터리 사용량이 낮고, 사물 인터넷에 적합하다.

### 개발 방법
Bluetooth Classic을 구현하기 위해서는 Scanning 작업을 위한 Broadcast Receiver 구혆과 페어링 및 블루투스 소켓 통신을 위한 Thread를 별도로 구현해야 한다.  
반면에, BLE 개발 방식은 다양한 객체와 콜백을 지원하기 때문에 Bluetooth Classic보다 개발 친화적이다.