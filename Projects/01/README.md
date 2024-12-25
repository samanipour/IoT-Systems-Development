### Building a Smart Monitoring System using ThingsBoard and MQTT Protocol

---

#### Objective:
The goal of this project is to create an IoT-based smart monitoring system where devices (e.g., sensors and actuators) send data to the ThingsBoard platform via MQTT protocol, and ThingsBoard processes the data and sends commands back to the devices based on pre-defined rules. You will also create a dashboard to visualize sensor data and control actuators interactively.

---

### Project Steps:

#### **Step 1: Setup ThingsBoard Platform**
1. **Install ThingsBoard**:
   - If you are using a local setup, download and install ThingsBoard on your local machine or server. You can find installation guides [here](https://thingsboard.io/docs/user-guide/install/).
   - Alternatively, you can sign up for a free account on the ThingsBoard Cloud platform (https://thingsboard.io/).

2. **Create an Admin Account**:
   - Once ThingsBoard is set up, log in with the default admin credentials and create an admin user.

3. **Configure Devices in ThingsBoard**:
   - In the ThingsBoard platform, create a new device. This represents the physical IoT device (sensor or actuator) in your system.
   - Select **MQTT** as the connectivity protocol when adding the device.

#### **Step 2: Set Up MQTT Communication**
1. **Install MQTT Library for Devices**:
   - For this project, we will use an Arduino ESP board (e.g., ESP8266 or ESP32) to send and receive data to/from ThingsBoard. Install the MQTT library for Arduino, such as the **PubSubClient** library.
   - In the Arduino IDE, go to **Sketch → Include Library → Manage Libraries**, then search for **PubSubClient** and install it.

2. **Device Configuration**:
   - Configure your Arduino ESP board to connect to a Wi-Fi network and ThingsBoard using MQTT.
   - Set the MQTT broker URL to ThingsBoard’s MQTT endpoint (e.g., `mqtt://thingsboard_server_ip:1883`).
   - Use the device’s **Access Token** (generated when creating the device in ThingsBoard) to authenticate the device.

#### **Step 3: Connect Devices and Send Sensor Data**
1. **Read Data from Sensors**:
   - Use simple sensors like **Temperature Sensors (e.g., DHT11, DHT22)** and **Light Sensors (e.g., LDR)**. 
   - Write code to read the sensor data (e.g., temperature, humidity) periodically.
   
2. **Send Data via MQTT**:
   - Send sensor data (e.g., temperature or humidity readings) to ThingsBoard over MQTT.
   - The data should be in a JSON format, for example:
     ```json
     {
       "temperature": 22.5,
       "humidity": 60
     }
     ```

3. **Verify Data Reception on ThingsBoard**:
   - Ensure that the data from the device is being received on the ThingsBoard platform by checking the **Device Data** in the dashboard.

#### **Step 4: Create Data Processing Rules in ThingsBoard**
1. **Rule Engine Setup**:
   - Navigate to the **Rule Chains** section in ThingsBoard and create a new **Rule Chain**.
   - Define a rule that processes the incoming sensor data and logs it.
   
2. **Logging Rule**:
   - Create a rule that logs the sensor data (e.g., store temperature and humidity data in the ThingsBoard database).

3. **Conditional Rules**:
   - Create a rule that triggers commands based on specific sensor data. For example, if the temperature exceeds 30°C, send a command to an actuator (e.g., turn on a fan or LED light).
   - Example logic for sending commands: 
     - **If temperature > 30°C, send command to turn on LED.**
     - **Else, send command to turn off LED.**

4. **Testing Rule Engine**:
   - Test the rule engine by sending sensor data and ensuring that the correct actions (commands) are triggered based on the rules.

#### **Step 5: Create a Monitoring Dashboard**
1. **Add Widgets to Dashboard**:
   - In ThingsBoard, create a new **Dashboard**.
   - Add widgets such as **Temperature Gauge**, **Humidity Gauge**, and **LED State Indicator**.
   - Configure the widgets to display real-time data for the connected devices.

2. **Customize Widgets for Device Control**:
   - Add widgets to control the devices. For example:
     - **Switch Widget** to turn the LED on/off.
     - Configure the widget to send commands to the device (e.g., toggle LED state).
   
3. **Dashboard Setup**:
   - Add a **Temperature Gauge** that displays the real-time temperature readings.
   - Add a **LED State Indicator** that shows the current state of the LED (on/off).
   
4. **Interactive Device Control**:
   - Ensure that users can interact with the dashboard widgets to change device states (e.g., toggle an LED using a button widget).

#### **Step 6: Test and Final Verification**
1. **Sensor Data Monitoring**:
   - Ensure that the sensor data is being received, processed, and logged correctly on the dashboard.
   
2. **Device Control Testing**:
   - Test the interactive widgets on the dashboard. For example, toggle the LED using the dashboard switch widget and ensure the device responds correctly.

3. **Final Demo**:
   - Demonstrate the entire system where:
     - Devices send real-time sensor data to ThingsBoard.
     - ThingsBoard processes the data and triggers actions (e.g., turning on/off an actuator).
     - The dashboard provides a visual representation of the data and allows for user interaction to control devices.

---

### Example Scenario (Using Arduino ESP Board with Sensors and Actuators):

1. **Device**: Arduino ESP32 board connected to:
   - **DHT22 Temperature Sensor** (senses temperature and humidity).
   - **LED** (to act as an actuator, which will be controlled by the temperature).

2. **Steps in the Example**:
   - **Setup**: The ESP32 board is programmed to read temperature and humidity from the DHT22 sensor.
   - **Data Sending**: Every 10 seconds, the ESP32 sends temperature and humidity data to ThingsBoard over MQTT.
   - **Processing Rule**: ThingsBoard receives the temperature data and, if the temperature exceeds 30°C, sends a command to the ESP32 to turn on the LED.
   - **Dashboard**: A dashboard is created with:
     - A **Temperature Gauge** to show the current temperature.
     - A **Switch Widget** to control the LED state (on/off).
   - **User Interaction**: The user can click the switch widget to toggle the LED on or off.

---

### Deliverables:
1. **ThingsBoard Setup**: A working ThingsBoard instance with devices connected via MQTT.
2. **Device Code**: Arduino code to read sensor data and communicate via MQTT.
3. **ThingsBoard Dashboard**: A dashboard that displays real-time sensor data and allows device control.
4. **Rule Engine Configuration**: Rules to process sensor data and control actuators based on specific conditions.

---

### Grading Criteria:
- **Basic (50%)**: Successful setup of ThingsBoard, MQTT communication, and device data logging.
- **Intermediate (70%)**: Correct implementation of rule engine logic to trigger actuator commands.
- **Advanced (90%)**: Creation of a fully interactive dashboard with data visualization and device control widgets.
- **Bonus (10%)**: Additional advanced features like adding more devices, complex rules, or integrating more sensors.

---
