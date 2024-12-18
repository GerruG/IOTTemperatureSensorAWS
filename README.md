# IoT Project: ESP32 with AWS IoT Core, DynamoDB, and Grafana

This project demonstrates an IoT solution where an ESP32 equipped with a DHT11 sensor collects temperature and humidity data, uploads it to AWS IoT Core, stores it in DynamoDB, and visualizes it in Grafana.

---

## **Features**

- Secure communication with AWS IoT Core using mutual TLS (mTLS).
- Real-time telemetry data collection from a DHT11 sensor.
- Data storage in AWS DynamoDB for long-term analytics.
- Visualization using Grafana dashboards.
- Integration with APIs for ESP32 telemetry data.

---

## **System Architecture**

```
DHT11 Sensor --> ESP32 --> AWS IoT Core --> Lambda --> DynamoDB --> Grafana
```

- **ESP32**: Reads data from the DHT11 sensor and sends it to AWS IoT Core.
- **AWS IoT Core**: Manages secure communication with the ESP32.
- **Lambda Function**: Processes incoming telemetry data and stores it in DynamoDB.
- **DynamoDB**: Stores sensor data for analysis.
- **Grafana**: Queries DynamoDB and visualizes the data on dashboards.

---

## **Hardware Requirements**

- ESP32 development board.
- DHT11 sensor (or similar temperature/humidity sensor).
- USB cable for programming.

---

## **Software Requirements**

- Arduino IDE or PlatformIO.
- AWS account with IoT Core and DynamoDB access.
- Grafana (local or hosted).

---

## **Getting Started**

### **1. Configure AWS IoT Core**

1. **Register a Thing:**
   - Create a Thing in AWS IoT Core.
   - Download the certificates (device certificate, private key, and Amazon Root CA).

2. **Create an IoT Policy:**
   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "iot:Connect",
           "iot:Publish",
           "iot:Subscribe",
           "iot:Receive"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

3. **Attach the Policy:**
   - Attach the policy to the device’s certificate.

4. **Define MQTT Topics:**
   - Publish: `/telemetry`
   - Subscribe: `/downlink`

---

### **2. Connect the DHT11 Sensor**

- **VCC**: Connect to ESP32’s 3.3V or 5V pin.
- **GND**: Connect to ESP32’s GND pin.
- **Data**: Connect to GPIO0 of the ESP32.

---

### **3. Program the ESP32**

1. **Update `secrets.h`:**
   Replace placeholders with your credentials and AWS IoT Core details:
   ```cpp
   const char WIFI_SSID[] = "YourWiFiSSID";
   const char WIFI_PASSWORD[] = "YourWiFiPassword";
   const char AWS_IOT_ENDPOINT[] = "YourAWSIoTEndpoint.amazonaws.com";
   ```

2. **Upload the Code:**
   - Compile and upload the provided code using Arduino IDE or PlatformIO.
   - Monitor the Serial Output for logs.

---

### **4. Configure DynamoDB**

1. **Create a Table:**
   - Table Name: `SensorData`
   - Primary Key: `timestamp` (String).

2. **Set Up a Lambda Function:**
   Use the following Python code:
   ```python
   import boto3
   import json

   def lambda_handler(event, context):
       dynamodb = boto3.resource('dynamodb')
       table = dynamodb.Table('SensorData')
       for record in event['Records']:
           message = json.loads(record['body'])
           table.put_item(Item=message)
       return {"statusCode": 200, "body": "Data written to DynamoDB"}
   ```

3. **Create an IoT Rule:**
   Forward messages from the `/telemetry` topic to the Lambda function.

---

### **5. Set Up Grafana**

1. **Add a Data Source:**
   - Install the AWS DynamoDB plugin for Grafana.
   - Configure DynamoDB as a data source.

2. **Create Dashboards:**
   - Add visualizations for temperature and humidity.
   - Configure thresholds for alerts.

---

### **6. Test the Full Pipeline**

1. **Verify Connectivity:**
   - Confirm the ESP32 connects to AWS IoT Core and publishes data.

2. **Check DynamoDB:**
   - Ensure telemetry data is stored correctly.

3. **Visualize Data:**
   - Verify Grafana dashboards display real-time data.

---

### **Optional Enhancements**

- **Local Data Buffering:** Store data locally on the ESP32 during network interruptions.
- **Enhanced Security:** Periodically rotate AWS IoT Core certificates.
- **Edge Analytics:** Perform preliminary data processing on the ESP32.

---
