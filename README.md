---

# Data trasfer to mobile phone with blutooth using nRF52

This project is a simple BLE (Bluetooth Low Energy) counter for the nRF52 series microcontrollers, using the Bluefruit library. The program broadcasts a counter value that increments every second and sends it as a notification to any connected BLE device.

## Project Overview

This BLE counter uses a custom BLE service and characteristic to notify connected devices with an incrementing count. The counter value is updated every second and transmitted as a BLE notification, allowing the connected device to receive the updated count in real-time.

## Features

- **BLE Service (UUID 0x180A):** A custom BLE service defined for the counter functionality.
- **BLE Characteristic (UUID 0x2A56):** Notifies the connected BLE device every second with the current counter value.
- **Counter Increments:** The counter automatically increments every second.
- **Transmission Power:** Configured to the maximum level for increased BLE range.

## Requirements

- **Hardware:** Adafruit nRF52 (such as nRF52840).
- **Libraries:** [Bluefruit Library](https://github.com/adafruit/Adafruit_nRF52_Arduino) installed via Arduino Library Manager.
- **Software:** Arduino IDE with nRF52 board support installed.

## How It Works

1. **Setup BLE:** The code initializes a BLE service and characteristic. 
2. **BLE Notification:** The counter value is converted to a string and sent as a notification to connected BLE devices.
3. **Counter Logic:** The counter increments every second, updating the BLE characteristic to notify subscribers of the new value.

## Usage Instructions

1. **Install the Required Libraries:**
   - Open the Arduino Library Manager and install the **Bluefruit** library.
   - Set up your IDE to support Adafruit nRF52 boards.

2. **Upload the Code:**
   - Connect the nRF52 board to your computer, compile, and upload the code to the board.

3. **Connect to a BLE Device:**
   - Use a BLE app (e.g., nRF Connect) to find and connect to the device named `"nRF52_Count"`.
   - Once connected, subscribe to the characteristic with UUID `0x2A56` to start receiving the count updates.

## Code Explanation

### 1. Library Inclusion and BLE Setup

```cpp
#include <bluefruit.h>
```

This line includes the Bluefruit library, which provides functions for initializing and handling BLE functionality.

### 2. Setting Up BLE Service and Characteristic

```cpp
BLEService countService = BLEService(0x180A);
BLECharacteristic countChar = BLECharacteristic(0x2A56);
```

Here, a custom BLE service (`countService`) and characteristic (`countChar`) are created using unique 16-bit UUIDs:
- **Service UUID (0x180A):** Represents the counter service.
- **Characteristic UUID (0x2A56):** Represents the characteristic used for sending notifications of the counter’s value.

### 3. Counter and Timer Variables

```cpp
unsigned int count = 0;
unsigned long previousMillis = 0;
```

- `count`: Holds the current count value, starting from 0.
- `previousMillis`: Stores the last time the count was updated, helping control the 1-second interval.

### 4. `setup()` Function

```cpp
void setup() {
  Serial.begin(9600);
  while (!Serial);
```

The `setup()` function initializes Serial communication for debugging and initializes the BLE peripheral with specific configurations.

#### BLE Initialization

```cpp
  Bluefruit.begin();
  Bluefruit.setTxPower(4);    
  Bluefruit.setName("nRF52_Count");
```

- `Bluefruit.begin()`: Initializes the BLE hardware.
- `Bluefruit.setTxPower(4)`: Sets transmission power to the maximum.
- `Bluefruit.setName("nRF52_Count")`: Sets the BLE device name, making it identifiable to other devices.

#### Advertising Configuration

```cpp
  Bluefruit.Advertising.addService(countService);
  Bluefruit.Advertising.start();
```

These lines add the custom BLE service to the device’s advertising packet and start broadcasting the device to make it discoverable.

#### Configuring the Characteristic

```cpp
  countService.begin();
  countChar.setProperties(CHR_PROPS_NOTIFY);
  countChar.setPermission(SECMODE_OPEN, SECMODE_NO_ACCESS);
  countChar.setFixedLen(4);
  countChar.begin();
```

- `setProperties(CHR_PROPS_NOTIFY)`: Sets the characteristic to notify mode, allowing it to send data to connected devices automatically when updated.
- `setPermission(SECMODE_OPEN, SECMODE_NO_ACCESS)`: Sets open permissions for reading without requiring authentication.
- `setFixedLen(4)`: Sets the fixed length of the characteristic to 4 bytes to accommodate count values.
- `begin()`: Starts the characteristic and service, making them available for use.

### 5. `loop()` Function: Updating and Sending the Count

```cpp
void loop() {
  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= 1000) {
    previousMillis = currentMillis;
    count++;
    char countStr[4];
    snprintf(countStr, sizeof(countStr), "%u", count);
    countChar.notify(countStr);
    Serial.print("Count value sent: ");
    Serial.println(countStr);
  }
}
```

- **Counter Update**: Every 1 second (`currentMillis - previousMillis >= 1000`), the counter increments.
- **Conversion to String**: The counter value is converted to a string using `snprintf()`.
- **Sending Notification**: The `countChar.notify(countStr);` function sends the string to any connected BLE devices subscribed to this characteristic.

The `Serial.print()` statements output the count to the Serial Monitor for debugging purposes.

## Expected Output

Upon running, you should see output like the following in the Serial Monitor:

```
BLE setup done. Waiting for connections...
Count value sent: 1
Count value sent: 2
Count value sent: 3
...
```

Each line represents the counter value sent to the BLE-connected device every second.

## Connecting to the BLE Counter

1. Open a BLE app (e.g., **nRF Connect**).
2. Connect to `"nRF52_Count"`.
3. Subscribe to the characteristic with UUID `0x2A56`.
4. View the notifications to see the counter’s incrementing values every second.

## License

This project is open-source. You are free to use, modify, and distribute the code as needed.

---

This `README.md` provides a detailed overview of the project, setup instructions, code explanations, and usage. It should help users understand and work with your BLE counter project effectively.
