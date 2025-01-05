## **Control Devices using an IoT Server**
Devices will read **temperature and humidity** data, and the **ThingsBoard rule engine** will log the data and send commands to the devices to turn on LEDs based on specific temperature thresholds using MQTT.

---

### **Step 1: Configure ThingsBoard Server (Cloud or Local)**

1. **ThingsBoard Cloud**:
   - If you're using the ThingsBoard Cloud, sign up at [ThingsBoard Cloud](https://cloud.thingsboard.io/).
   - Create a new device on the ThingsBoard Cloud platform by going to **Devices** and clicking **Add New Device**.
   - Copy the **Access Token** for the device, which will be used for authentication when sending data from the device.

2. **ThingsBoard Local Server**:
   - If you’re using a local ThingsBoard server, install it following the [ThingsBoard Installation Guide](https://thingsboard.io/docs/user-guide/install/).
   - Access the ThingsBoard UI at `http://localhost:8080` (or the IP address if deployed on a server).
   - Log in with the default credentials and create a new device. Ensure the device is assigned a **unique access token**.

---

### **Step 2: Define the Hardware Setup for the IoT Devices**

Each IoT device (e.g., Wemos D1 R1 or NodeMCU) will be set up with **three LEDs**: **Green**, **Yellow**, and **Red**. These LEDs will be used to indicate the temperature range.

#### **Wiring the LEDs:**
1. **LED Setup**:
   - **Green LED**: Turns on when the temperature is **normal** (within threshold).
   - **Yellow LED**: Turns on when the temperature is **close to threshold**.
   - **Red LED**: Turns on when the temperature is **too high** (surpasses threshold).

2. **Wiring the LEDs**:
   - Connect the **Anode (positive)** of each LED to a **GPIO pin** of the Wemos D1 R1 or NodeMCU.
   - Connect the **Cathode (negative)** of each LED to **GND** via a **330Ω resistor** to limit the current.

   Example wiring for a **Wemos D1 R1** (you can adjust pins based on your device):
   - **Green LED**: GPIO5 (D1)
   - **Yellow LED**: GPIO4 (D2)
   - **Red LED**: GPIO0 (D3)

**LED Wiring Diagram**:
```
Green LED    → GPIO5 (D1) → 330Ω resistor → GND
Yellow LED   → GPIO4 (D2) → 330Ω resistor → GND
Red LED       → GPIO0 (D3) → 330Ω resistor → GND
```

---

### **Step 3: Write Arduino Code to Read Sensor Data and Send MQTT Messages**

Now, write the code for the device that reads the **DHT22 sensor**, sends **temperature and humidity data** to ThingsBoard, and receives commands to control the LEDs.

#### **Arduino Code to Read Temperature, Send Data via MQTT, and Control LEDs**

```cpp
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

// Replace with your network credentials
const char* ssid = "your_wifi_ssid";
const char* password = "your_wifi_password";

// ThingsBoard MQTT Broker information
const char* mqtt_server = "demo.thingsboard.io";  // For ThingsBoard Cloud
const char* mqtt_user = "your_device_access_token";  // Device access token
const char* mqtt_topic = "v1/devices/me/telemetry"; // Default telemetry topic

// DHT Sensor settings
uint8_t DHTPin = D14;
#define DHTTYPE DHT11    // Define the DHT sensor type
DHT dht(DHTPin, DHTTYPE); // Initialize DHT sensor

// LED Pin settings
#define GREEN_LED_PIN 5  // GPIO5 (D1)
#define YELLOW_LED_PIN 4 // GPIO4 (D2)
#define RED_LED_PIN 0    // GPIO0 (D3)

DHT dht(DHTPIN, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

void setup() {
  Serial.begin(115200);
  delay(10);
  
  // Connect to Wi-Fi
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(1000);
    Serial.println("Connecting to WiFi...");
  }
  Serial.println("Connected to WiFi");
  
  // Initialize MQTT client
  client.setServer(mqtt_server, 1883);
  
  // Initialize DHT
  delay(100);
  pinMode(DHTPin, INPUT);
  dht.begin();
  delay(100);

  // Initialize LED pins
  pinMode(GREEN_LED_PIN, OUTPUT);
  pinMode(YELLOW_LED_PIN, OUTPUT);
  pinMode(RED_LED_PIN, OUTPUT);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Read temperature and humidity from DHT22
  float temperature = dht.readTemperature();  // Celsius
  float humidity = dht.readHumidity();

  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
  } else {
    // Prepare the telemetry data
    String payload = "{\"temperature\": " + String(temperature) + ", \"humidity\": " + String(humidity) + "}";

    // Publish the data to ThingsBoard
    client.publish(mqtt_topic, payload.c_str());

    // Print to Serial Monitor
    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.print(" °C  ");
    Serial.print("Humidity: ");
    Serial.print(humidity);
    Serial.println(" %");

    // Control LEDs based on temperature threshold
    controlLEDs(temperature);
  }
  
  delay(5000);  // Send data every 5 seconds
}

// Reconnect to ThingsBoard MQTT server if the connection is lost
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ArduinoClient", mqtt_user, NULL)) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      delay(5000);
    }
  }
}

// Control LEDs based on temperature threshold
void controlLEDs(float temperature) {
  // Define temperature thresholds (for example)
  const float greenThreshold = 25.0;   // Temperature below this will turn Green LED
  const float yellowThreshold = 30.0;  // Temperature between green and yellow will turn Yellow LED
  const float redThreshold = 35.0;     // Temperature above this will turn Red LED
  
  // Turn off all LEDs initially
  digitalWrite(GREEN_LED_PIN, LOW);
  digitalWrite(YELLOW_LED_PIN, LOW);
  digitalWrite(RED_LED_PIN, LOW);

  // Control LEDs based on temperature
  if (temperature < greenThreshold) {
    digitalWrite(GREEN_LED_PIN, HIGH);  // Green LED ON
  } else if (temperature >= greenThreshold && temperature < yellowThreshold) {
    digitalWrite(YELLOW_LED_PIN, HIGH); // Yellow LED ON
  } else if (temperature >= redThreshold) {
    digitalWrite(RED_LED_PIN, HIGH);    // Red LED ON
  }
}
```

#### **Explanation**:
- **DHT22 sensor** reads temperature and humidity.
- **MQTT** is used to send data to the ThingsBoard server.
- **LED Control**: The LEDs (Green, Yellow, and Red) are controlled based on temperature thresholds:
  - Green: Temperature below 25°C.
  - Yellow: Temperature between 25°C and 30°C.
  - Red: Temperature above 30°C.
- **Access Token**: Replace `"your_device_access_token"` with the device's access token from ThingsBoard.

---

### **Step 4: Set Up ThingsBoard Rule Engine**

1. **Login to ThingsBoard**:
   - Log in to your **ThingsBoard instance** (either local or cloud).

2. **Create a New Rule Chain**:
   - Navigate to **Rule Chains** from the left-hand menu.
   - Click **Add New Rule Chain** and name it (e.g., **Temperature Control**).
   - Open the newly created rule chain.

3. **Add Nodes to the Rule Chain**:
   - **Telemetry Node**: Add a **Telemetry Processing Node** to handle incoming telemetry data.
   - **Filter Node**: Use a **Filter Node** to filter the telemetry based on the temperature value.
   - **RPC (Remote Procedure Call) Node**: Add an **RPC Node** to send commands back to the device to control the LEDs.

4. **Configure the Rule**:
   - In the **Filter Node**, set conditions to check the temperature attribute.
     - If temperature > 30°C (Red), send an **RPC command** to turn on the Red LED.
     - If temperature > 25°C and < 30°C (Yellow), send an **RPC command** to turn on the Yellow LED.
     - If temperature < 25°C (Green), send an **RPC command** to turn on the Green LED.
   
5. **Create and Configure RPC Commands**:
   - In the **RPC Node**, define a method to send an RPC command. This method will be used to turn on or off the LEDs based on the temperature value.
   - The command will

 be sent to the devices as **MQTT messages**.

   Example:
   - RPC Command to turn on the **Green LED**: `"green_led_on"`
   - RPC Command to turn on the **Yellow LED**: `"yellow_led_on"`
   - RPC Command to turn on the **Red LED**: `"red_led_on"`

6. **Deploy the Rule Chain**:
   - Click **Deploy** to apply the rule chain.

---

### **Step 5: Monitor Data and Control LEDs**

1. **Create a Dashboard** in ThingsBoard:
   - Go to **Dashboards** and click **Add New Dashboard**.
   - Add widgets to display temperature and humidity from each device.
   - Add **RPC widgets** for controlling LEDs, allowing the user to manually turn LEDs on or off if desired.

2. **Test the System**:
   - The devices will now send telemetry data to ThingsBoard.
   - The rule engine will process the data and send commands to control the LEDs based on the temperature.
   - You should see the appropriate LED light up on the devices based on the temperature range.

---

### **Conclusion:**
You’ve successfully set up an **IoT lab** using **ThingsBoard** with **MQTT protocol** to send data from **Arduino-based devices** (e.g., Wemos D1 R1, NodeMCU) and control **LEDs** based on temperature thresholds. The **ThingsBoard Rule Engine** allows you to process telemetry and send commands to the devices, controlling the LEDs dynamically based on real-time temperature readings.