
## **Getting Started with Arduino Boards (Wemos D1 R1 / NodeMCU) to Read DHT22 Sensor Data and Log It**

### **Step 1: Install the Arduino IDE**
If you haven’t installed the Arduino IDE yet:
1. Download the Arduino IDE from the [official website](https://www.arduino.cc/en/software).
2. Follow the installation instructions for your operating system (Windows, macOS, Linux).

---

### **Step 2: Install the ESP8266 Board Support**
For Wemos D1 R1 or NodeMCU, you need to install the ESP8266 board support in Arduino IDE:
1. **Open the Arduino IDE**.
2. **Go to File → Preferences**.
3. In the **Additional Boards Manager URLs** field, add the following URL:
   ```
   http://arduino.esp8266.com/stable/package_esp8266com_index.json
   ```
4. **Click OK** to save and close the Preferences window.
5. **Go to Tools → Board → Boards Manager**.
6. Search for `esp8266` and install **esp8266 by ESP8266 Community**.
7. Once installed, select **Wemos D1 R1** or **NodeMCU** as your board under **Tools → Board**.

---

### **Step 3: Install DHT Sensor Library**
The DHT22 sensor requires a specific library to interface with the Arduino. We will install the `DHT` library by Adafruit.

1. **Go to Sketch → Include Library → Manage Libraries**.
2. In the Library Manager window, search for `DHT sensor library`.
3. Find **DHT sensor library by Adafruit**, and click **Install**.
4. Optionally, install **Adafruit Unified Sensor** (a dependency for the DHT sensor library) if prompted.

---

### **Step 4: Wiring the DHT22 Sensor**

#### **DHT22 Pinout:**
- **VCC**: Power (3.3V or 5V depending on your board).
- **GND**: Ground.
- **DATA**: Signal pin that transmits the data.

#### **Wiring:**
1. **Connect VCC (DHT22)** to **3.3V** on the Wemos D1 R1 or NodeMCU.
2. **Connect GND (DHT22)** to **GND** on the Wemos D1 R1 or NodeMCU.
3. **Connect DATA (DHT22)** to a digital GPIO pin. For example, use **D4 (GPIO2)** on the Wemos D1 R1 or **D2 (GPIO4)** on NodeMCU.
4. **Use a 10kΩ pull-up resistor** between the DATA pin and VCC to ensure proper signal transmission.

**DHT22 Wiring Diagram:**
```
DHT22
  |-- VCC  →  3.3V (on Wemos D1 R1 / NodeMCU)
  |-- GND  →  GND (on Wemos D1 R1 / NodeMCU)
  |-- DATA → D4 (GPIO2) / D2 (GPIO4)
  |-- 10kΩ pull-up resistor → VCC (between DATA and VCC)
```

---

### **Step 5: Write Code to Read DHT22 Sensor Data**

1. **Open a new sketch** in the Arduino IDE (File → New).
2. Copy and paste the following code to read data from the DHT22 sensor:

```cpp
#include <DHT.h>

// Define the DHT sensor type and pin
// #define DHTPIN 2     // Pin connected to the DATA pin (D14 on Wemos D1 R1, D2 on NodeMCU)
#define DHTTYPE DHT11   // DHT11 sensor
// DHT Sensor
uint8_t DHTPin = D14;

DHT dht(DHTPin, DHTTYPE); // Initialize DHT sensor

void setup() {
  Serial.begin(115200);   // Start serial communication at 115200 baud
  delay(100);
  pinMode(DHTPin, INPUT);
  dht.begin();            // Initialize the DHT sensor
  delay(2000);            // Wait for sensor to stabilize
}

void loop() {
  // Read temperature and humidity values
  float temperature = dht.readTemperature();  // Read temperature in Celsius
  float humidity = dht.readHumidity();        // Read humidity

  // Check if the readings are valid
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
  } else {
    // Print temperature and humidity to Serial Monitor
    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.print(" °C  ");
    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.println(" %");
  }
  
  // Delay between readings
  delay(2000);
}
```

#### **Explanation of the Code:**
- **DHTPIN 2**: The pin connected to the DHT22 DATA pin (GPIO2 for Wemos D1 R1, GPIO4 for NodeMCU).
- `dht.begin()`: Initializes the DHT22 sensor.
- `dht.readTemperature()`: Reads the temperature in Celsius.
- `dht.readHumidity()`: Reads the humidity percentage.
- `isnan()`: Checks if the readings are valid (returns `NaN` if the sensor fails to read).

---

### **Step 6: Upload the Code to the Wemos D1 R1 or NodeMCU**
1. **Connect your Wemos D1 R1 / NodeMCU** to your computer using a USB cable.
2. **Select the correct board** under **Tools → Board** (Wemos D1 R1 or NodeMCU).
3. **Select the correct COM port** under **Tools → Port**.
4. **Click the Upload button** (the right arrow icon) to upload the code to your board.
5. Wait for the upload to complete. Once done, you will see the serial monitor output.

---

### **Step 7: View the Output on the Serial Monitor**
1. **Open the Serial Monitor** (Tools → Serial Monitor).
2. Set the baud rate to **115200** (to match the `Serial.begin(115200)` in the code).
3. You should see readings from the DHT22 sensor, similar to this:
   ```
   Temperature: 25.00 °C  Humidity: 60.00 %
   Temperature: 25.10 °C  Humidity: 59.80 %
   Temperature: 25.20 °C  Humidity: 59.60 %
   ```

---
