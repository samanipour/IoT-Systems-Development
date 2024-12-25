
## **Read data from a sensors and send it to a Processing server using MQTT protocol.**
This Project includes creating ThingsBoard rules using its Rule Engine to receive and log device data, as well as monitoring the temperature and humidity from multiple devices using ThingsBoard widgets.
---

### **Step 1: Install Arduino IDE and Set Up Boards**
If you haven’t installed the Arduino IDE and board support yet, follow these instructions:

1. **Install Arduino IDE**:
   - Download from [Arduino IDE](https://www.arduino.cc/en/software) and follow installation instructions.
   
2. **Install ESP8266 Board Support**:
   - Open Arduino IDE.
   - Go to **File → Preferences**.
   - Add the following URL in the **Additional Boards Manager URLs** field:
     ```
     http://arduino.esp8266.com/stable/package_esp8266com_index.json
     ```
   - Go to **Tools → Board → Boards Manager**, search for **esp8266**, and click **Install**.
   - Select your board (**Wemos D1 R1** or **NodeMCU**) from **Tools → Board**.

3. **Install DHT Library**:
   - Go to **Sketch → Include Library → Manage Libraries**.
   - Search for `DHT sensor library` and install the **Adafruit DHT Sensor Library**.

---

### **Step 2: Set Up DHT22 Sensor Hardware**
1. **Wiring the DHT22 Sensor**:
   - **VCC (DHT22)** → **3.3V** (on Wemos D1 R1 / NodeMCU)
   - **GND (DHT22)** → **GND** (on Wemos D1 R1 / NodeMCU)
   - **DATA (DHT22)** → **GPIO2 (D4)** on Wemos D1 R1 or **GPIO4 (D2)** on NodeMCU
   - Add a **10kΩ pull-up resistor** between the **DATA** pin and **VCC**.

**Wiring Diagram:**
```
DHT22
  |-- VCC  →  3.3V (on Wemos D1 R1 / NodeMCU)
  |-- GND  →  GND (on Wemos D1 R1 / NodeMCU)
  |-- DATA → D4 (GPIO2) / D2 (GPIO4)
  |-- 10kΩ pull-up resistor → VCC (between DATA and VCC)
```

---

### **Step 3: Configure MQTT Broker (ThingsBoard)**
Before proceeding with the Arduino code, ensure that you have a ThingsBoard server set up and running. You can either set up a local server or use the ThingsBoard Cloud.

1. **Create an Account on ThingsBoard Cloud** (Optional if using local ThingsBoard server):
   - Go to [ThingsBoard Cloud](https://cloud.thingsboard.io/) and sign up.

2. **Create a Device on ThingsBoard**:
   - Login to your ThingsBoard instance (Cloud or Local).
   - Go to the **Devices** section in the sidebar and click **Add New Device**.
   - Enter a name for your device (e.g., “Temperature Sensor 1”) and click **Add**.
   - After the device is created, click on it to view the **Device Information** (especially the **Access Token**, which will be used in the Arduino code).

---

### **Step 4: Write Arduino Code to Read DHT22 and Send Data to ThingsBoard via MQTT**

1. **Install MQTT Library**:
   - Go to **Sketch → Include Library → Manage Libraries**.
   - Search for `PubSubClient` and install it (this library helps in sending data via MQTT).

2. **Write Code to Read DHT22 and Publish Data to MQTT**:
   - Open a new sketch and copy the following code to read DHT22 data and send it to ThingsBoard via MQTT.

```cpp
#include <DHT.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

// Replace with your network credentials
const char* ssid = "your_wifi_ssid";
const char* password = "your_wifi_password";

// ThingsBoard MQTT Broker information
const char* mqtt_server = "demo.thingsboard.io";  // For ThingsBoard Cloud
const char* mqtt_user = "your_device_access_token"; // Device access token
const char* mqtt_topic = "v1/devices/me/telemetry"; // Default telemetry topic

// DHT22 Sensor settings
#define DHTPIN 2         // Pin connected to DHT22 data pin (GPIO2)
#define DHTTYPE DHT22    // Define the DHT sensor type

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

  // Initialize DHT22
  dht.begin();
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

    delay(5000);  // Send data every 5 seconds
  }
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
```

#### **Explanation:**
- Replace **`your_wifi_ssid`** and **`your_wifi_password`** with your actual Wi-Fi credentials.
- Replace **`your_device_access_token`** with the **Access Token** from your ThingsBoard device.
- This code reads the **temperature** and **humidity** from the **DHT22** sensor and sends the data to ThingsBoard using MQTT.

---

### **Step 5: Upload Code and Monitor Data on ThingsBoard**

1. **Upload the Code** to your **Wemos D1 R1 or NodeMCU** using the Arduino IDE.
2. Open the **Serial Monitor** to check the connection status.
3. After the device successfully connects to Wi-Fi and MQTT broker, it will start sending the **temperature and humidity** data to ThingsBoard.

---

### **Step 6: Create ThingsBoard Rule Engine to Receive and Log Data**

1. **Login to ThingsBoard** and go to the **Rule Chains** section.
2. Click **Add New Rule Chain** and name it (e.g., **Device Data Logging**).
3. In the rule chain, add the following nodes:
   - **Telemetry processing node**: To receive and process telemetry data.
   - **Log node**: To log the telemetry data.
   
4. **Configure the Log Node** to store the incoming telemetry data, such as temperature and humidity.
   - Add a **"Log Telemetry"** node to print or log telemetry data.

5. **Deploy the Rule Chain** to apply the rule and start processing telemetry data from devices.

---

### **Step 7: Create Widgets to Monitor Devices' Temperature and Humidity**

1. **Create a Dashboard**:
   - Go to **Dashboards** in ThingsBoard and click **Add New Dashboard**.
   - Name the dashboard (e.g., **Temperature and Humidity Monitoring**).

2. **Add Widgets**:
   - Click **Add Widget** and select **Timeseries** or **Latest Value** widget for temperature and humidity.
   - **Configure the widget**:
     - Choose the device (e.g., **Temperature Sensor 1**).
     - Select **temperature** and **humidity** as the attributes for the widget.

3. **Visualize Data**:
   - The widgets will now display live data for temperature and humidity from each device connected to ThingsBoard.
   - You can add multiple widgets for different devices and monitor their readings in real-time.

---

### **Conclusion:**
You’ve successfully set up an **IoT lab** where **Arduino boards (Wemos D1 R1 or NodeMCU)** read data from the **DHT22 sensor**, send it to **ThingsBoard** using MQTT, and log the data with the **ThingsBoard Rule Engine**. Additionally, you’ve created widgets to visualize **temperature** and **humidity** data in real

-time for multiple devices.

Now you can expand the lab further by adding more devices or integrating additional sensors, and automating actions based on telemetry data!