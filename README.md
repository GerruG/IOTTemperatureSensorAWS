# IoT Project: ESP32 with AWS IoT Core, DynamoDB, and Grafana

This project demonstrates an IoT solution where an ESP32 equipped with a DHT11 sensor collects temperature and humidity data, uploads it to AWS IoT Core, stores it in DynamoDB, and visualizes it in Grafana. Additionally, data can be forwarded to an external API directly from AWS IoT Core.

---

## **Features**

- Secure communication with AWS IoT Core using mutual TLS (mTLS).
- Real-time telemetry data collection from a DHT11 sensor.
- Data storage in AWS DynamoDB for long-term analytics.
- Visualization using Grafana dashboards.
- Forwarding data to an external API via AWS IoT Core rules.

---

## **System Architecture**

```
DHT11 Sensor --> ESP32 --> AWS IoT Core --> Lambda --> DynamoDB --> Grafana
                        \
                         --> External API
```

- **ESP32**: Reads data from the DHT11 sensor and sends it to AWS IoT Core.
- **AWS IoT Core**: Manages secure communication and routing of data.
- **Lambda Function**: Processes incoming telemetry data and stores it in DynamoDB.
- **DynamoDB**: Stores sensor data for analysis.
- **Grafana**: Queries DynamoDB and visualizes the data on dashboards.
- **External API**: Receives data from AWS IoT Core for further processing.

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

### **5. Forward Data to an External API**

1. **Add an HTTP Action**:
   - In the IoT Rule, add an action to send data to an external HTTP API endpoint.
   - Specify the endpoint URL, such as `https://example.com/api/data`.
   - Set the method to `POST`.
   - Configure headers (e.g., `Content-Type: application/json`).
   - Ensure the endpoint is accessible and ready to accept incoming requests.

2. **Example Rule SQL Statement**:
   ```sql
   SELECT temperature, humidity, timestamp FROM '/telemetry'
   ```

3. **Test the Rule**:
   - Use the AWS IoT Core Test feature to publish sample data to the `/telemetry` topic.
   - Verify that the API receives the data by checking the API logs or responses.

4. **Example API Payload**:
   The payload sent to the API will look like this:
   ```json
   {
       "temperature": 25.5,
       "humidity": 60.0,
       "timestamp": "2024-01-01T12:00:00Z"
   }
   ```

5. **Monitor Logs**:
   - Use AWS CloudWatch to monitor the execution of the IoT Rule and troubleshoot any issues.

---

### **6. Set Up Grafana**

1. **Install the AWS DynamoDB Plugin:**
   - If not already installed, download and install the AWS DynamoDB plugin for Grafana.
   - Follow the Grafana documentation for plugin installation, or use the plugin marketplace in your Grafana instance.

2. **Configure AWS Credentials:**
   - Ensure that your Grafana instance has the correct AWS credentials configured. These credentials must have `dynamodb:Query` permissions for your DynamoDB table.
   - Add the credentials in one of the following ways:
     - Use an IAM role if Grafana is hosted on AWS.
     - Provide an AWS access key and secret key in Grafana's AWS configuration settings.

3. **Add DynamoDB as a Data Source:**
   - Navigate to **Configuration > Data Sources** in Grafana.
   - Click **Add data source** and select **AWS DynamoDB**.
   - Fill in the following fields:
     - **AWS Region:** Select the AWS region where your DynamoDB table is hosted.
     - **Default Table Name:** Enter `SensorData` (or the name of your table).
   - Test the connection to ensure Grafana can query DynamoDB.

4. **Query DynamoDB:**
   - Use the built-in query editor in Grafana to write queries. For example, to retrieve the latest temperature and humidity data:
     ```sql
     SELECT timestamp, temperature, humidity FROM SensorData
     WHERE timestamp >= '2024-01-01T00:00:00Z'
     ```

5. **Create Dashboards:**
   - Navigate to **Create > Dashboard** in Grafana.
   - Add a new panel with the following settings:
     - **Query Type:** Select DynamoDB.
     - **Query:** Use the query editor to define your desired data (e.g., temperature or humidity trends over time).
   - Select appropriate visualizations, such as time-series graphs, for your panels.

6. **Configure Alerts (Optional):**
   - Add thresholds for critical temperature or humidity values.
   - Configure alert notifications using Grafana’s alerting system (e.g., email, Slack).

7. **Test the Dashboard:**
   - Ensure that real-time data from your DynamoDB table is displayed in the dashboard.
   - Verify that the data updates as the ESP32 sends new telemetry.

---

### **7. Test the Full Pipeline**

1. **Verify Connectivity:**
   - Confirm the ESP32 connects to AWS IoT Core and publishes data.

2. **Check DynamoDB:**
   - Ensure telemetry data is stored correctly.

3. **Visualize Data:**
   - Verify Grafana dashboards display real-time data.

4. **Test API Forwarding:**
   - Publish sample data to the MQTT topic `/telemetry` and confirm the external API receives the data.

---

### **Optional Enhancements**

- **Local Data Buffering:** Store data locally on the ESP32 during network interruptions.
- **Enhanced Security:** Periodically rotate AWS IoT Core certificates.
- **Edge Analytics:** Perform preliminary data processing on the ESP32.

---
