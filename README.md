# 🌡️ Smart Environmental Monitor

> AI-powered IoT system for real-time environmental monitoring with intelligent anomaly detection

[![AWS](https://img.shields.io/badge/AWS-IoT_Core-FF9900?logo=amazon-aws)](https://aws.amazon.com/iot-core/)
[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

[🎥 Live Demo](#demo) • [📐 Architecture](#architecture) • [🚀 Quick Start](#quick-start) • [💡 Features](#features)

---

## 🎯 Overview

A production-ready IoT monitoring system that combines edge computing (Raspberry Pi) with cloud infrastructure (AWS) and artificial intelligence to detect environmental anomalies in real-time. Built for industrial, healthcare, and commercial applications requiring 24/7 environmental monitoring.

**Built in 4 weeks as a portfolio project demonstrating full-stack IoT architecture.**

### Why This Matters

Industrial facilities, data centers, and healthcare environments lose millions annually due to undetected environmental issues. This system provides:
- **Real-time monitoring** with sub-minute data granularity
- **AI-powered anomaly detection** using Claude/GPT models
- **Proactive alerting** before issues escalate
- **Cost-effective** cloud-native architecture (<$15/month at scale)

---

## ✨ Features

### Core Functionality
- 📊 **Multi-Sensor Data Collection**: Temperature, humidity, air quality (expandable)
- ☁️ **AWS IoT Core Integration**: Secure MQTT communication with device certificates
- 🤖 **AI Anomaly Detection**: Claude API analyzes patterns and detects unusual behavior
- 📱 **Real-Time Dashboard**: React-based visualization of live and historical data
- 🔔 **Smart Alerts**: SNS notifications when AI detects potential issues
- 💾 **Time-Series Storage**: DynamoDB for efficient sensor data storage
- 📈 **Historical Analytics**: 30-day trend analysis with AI-generated insights

### Production Features
- 🔒 **Security**: AWS IoT certificate-based authentication, encrypted data transmission
- 📉 **Cost Optimization**: Efficient data batching, lifecycle policies on S3 backups
- 🔧 **Monitoring**: CloudWatch dashboards track system health and AWS costs
- 🏗️ **Infrastructure as Code**: Full Terraform configuration included
- 🔄 **High Availability**: Multi-AZ DynamoDB, Lambda concurrency controls

---

## 🏗️ Architecture

```
┌─────────────────┐
│  Raspberry Pi   │  DHT22 (Temp/Humidity)
│  Zero 2 W       │  MQ135 (Air Quality)
│  (Edge Device)  │  Python + boto3
└────────┬────────┘
         │ MQTT (TLS 1.2)
         ▼
┌─────────────────────────────────────────┐
│           AWS IoT Core                  │
│  - Device Registry & Certificates       │
│  - Rule Engine (Route to Lambda)        │
└────────┬────────────────────────────────┘
         │
         ▼
┌─────────────────┐      ┌──────────────┐
│  Lambda (Python)│─────▶│  DynamoDB    │
│  Data Processor │      │  Time-Series │
└────────┬────────┘      └──────────────┘
         │
         ├──────────────┐
         ▼              ▼
┌─────────────────┐  ┌──────────────┐
│  Lambda (AI)    │  │     SNS      │
│  Claude API     │  │   Alerting   │
│  Anomaly Check  │  └──────────────┘
└─────────────────┘
         │
         ▼
┌─────────────────────────────────────────┐
│        React Dashboard (Amplify)        │
│  - Real-time data visualization         │
│  - AI analysis results                  │
│  - Historical trends & insights         │
└─────────────────────────────────────────┘
```

### Data Flow

1. **Edge Collection**: Raspberry Pi reads sensors every 60 seconds
2. **Secure Transport**: Publishes to AWS IoT Core via MQTT with TLS encryption
3. **Processing**: IoT Rule triggers Lambda to validate and store data in DynamoDB
4. **AI Analysis**: Separate Lambda analyzes recent data via Claude API every 5 minutes
5. **Alerting**: If anomaly detected (confidence >70%), SNS sends email/SMS
6. **Visualization**: Dashboard fetches data from DynamoDB API Gateway endpoint

---

## 🚀 Quick Start

### Prerequisites

- AWS Account with IoT Core access
- Raspberry Pi Zero 2 W (or any Pi model)
- DHT22 temperature/humidity sensor
- MQ135 air quality sensor (optional)
- Node.js 18+ (for dashboard)
- Python 3.9+
- Terraform 1.0+

### 1. Hardware Setup

**Wiring Diagram:**
```
DHT22:
  VCC  → Pi Pin 2 (5V)
  DATA → Pi Pin 7 (GPIO 4)
  GND  → Pi Pin 6 (Ground)

MQ135:
  VCC  → Pi Pin 4 (5V)
  DO   → Pi Pin 11 (GPIO 17)
  GND  → Pi Pin 9 (Ground)
```

**3D Printed Enclosure:**
- STL files in `/hardware/enclosure/`
- Print settings: 0.2mm layer height, 20% infill, PLA
- Includes mounting holes for sensors and Pi

### 2. AWS Infrastructure

```bash
# Clone repository
git clone https://github.com/TreGalloway/smart-environmental-monitor.git
cd smart-environmental-monitor

# Deploy AWS infrastructure with Terraform
cd terraform
terraform init
terraform plan
terraform apply

# Note the outputs:
# - iot_endpoint: Your AWS IoT Core endpoint
# - certificate_arn: Device certificate ARN
# - dashboard_url: React app URL
```

### 3. Raspberry Pi Configuration

```bash
# On your Raspberry Pi
cd edge-device
pip3 install -r requirements.txt

# Download IoT certificates from AWS Console
# Place in: ./certs/certificate.pem.crt
#           ./certs/private.pem.key
#           ./certs/AmazonRootCA1.pem

# Configure device
cp config.example.json config.json
# Edit config.json with your IoT endpoint

# Run sensor client
python3 sensor_client.py

# Set up as systemd service for auto-start
sudo cp smart-monitor.service /etc/systemd/system/
sudo systemctl enable smart-monitor
sudo systemctl start smart-monitor
```

### 4. Dashboard Setup

```bash
cd dashboard
npm install
npm run dev

# Build for production
npm run build
# Deploy to Amplify (configured via Terraform)
```

### 5. Test the System

```bash
# Verify data flow
aws dynamodb scan --table-name sensor-data --max-items 5

# Trigger test alert
python3 scripts/test_anomaly.py

# Check CloudWatch logs
aws logs tail /aws/lambda/sensor-processor --follow
```

---

## 💡 AI Anomaly Detection

The system uses Claude 3.5 Sonnet (via AWS Bedrock or Anthropic API) to analyze sensor data patterns:

### Detection Examples

**Temperature Spike:**
```
Input: Last 30 readings show 72°F → 89°F in 10 minutes
AI Analysis: "Rapid temperature increase detected. 
Potential causes: HVAC failure, fire risk, or sensor malfunction. 
Recommend immediate investigation."
Confidence: 92%
Alert: ✅ Sent via SNS
```

**Humidity Anomaly:**
```
Input: Humidity consistently 85%+ for 2 hours (normal: 40-60%)
AI Analysis: "Sustained high humidity indicates possible water leak 
or ventilation system failure. Risk of mold growth and equipment damage."
Confidence: 87%
Alert: ✅ Sent via SNS
```

**False Positive Handling:**
```
Input: Temperature drops 15°F over 30 minutes at 8am daily
AI Analysis: "Pattern matches scheduled HVAC adjustment. 
Normal behavior based on 7-day historical data."
Confidence: 95%
Alert: ❌ Suppressed (expected pattern)
```

### Prompt Engineering

The Lambda function uses structured prompts:

```python
prompt = f"""Analyze this environmental sensor data for anomalies:

Current reading:
- Temperature: {temp}°F
- Humidity: {humidity}%
- Air Quality: {air_quality} PPM
- Timestamp: {timestamp}

Historical context (last 24 hours):
- Avg temp: {avg_temp}°F (±{std_temp})
- Avg humidity: {avg_humidity}% (±{std_humidity})
- Known patterns: {patterns}

Determine:
1. Is this an anomaly? (yes/no)
2. Severity: (low/medium/high/critical)
3. Confidence: (0-100%)
4. Explanation: (brief cause analysis)
5. Recommendation: (action to take)

Respond in JSON format.
"""
```

---

## 📊 Dashboard Screenshots

### Real-Time Monitoring
![Dashboard - Real-time view](screenshots/dashboard-realtime.png)
*Live sensor data with 60-second refresh*

### AI Analysis Results
![AI Detection](screenshots/ai-detection.png)
*Claude's anomaly detection with confidence scores*

### Historical Trends
![Trends](screenshots/historical-trends.png)
*30-day temperature and humidity trends*

---

## 🛠️ Tech Stack

### Hardware
- **Raspberry Pi Zero 2 W** - Edge computing device
- **DHT22** - Digital temperature & humidity sensor (±0.5°C accuracy)
- **MQ135** - Air quality sensor (detects NH3, NOx, alcohol, benzene, smoke, CO2)
- **Custom 3D-printed enclosure** - PLA, designed in Fusion 360

### Edge Software
- **Python 3.9** - Sensor reading and IoT communication
- **AWSIoTPythonSDK** - MQTT client for AWS IoT Core
- **Adafruit CircuitPython** - Sensor libraries

### AWS Services
- **IoT Core** - Device management, MQTT broker, rule engine
- **Lambda** - Serverless data processing (Python 3.9 runtime)
- **DynamoDB** - Time-series sensor data storage (on-demand billing)
- **SNS** - Email/SMS alerting
- **S3** - Historical data backup with lifecycle policies
- **CloudWatch** - Logging, metrics, dashboards
- **Amplify** - Dashboard hosting and CI/CD
- **Bedrock** - Claude API for anomaly detection

### Dashboard
- **React 18** - Frontend framework
- **Next.js 14** - React framework with SSR
- **Recharts** - Data visualization
- **TailwindCSS** - Styling
- **AWS Amplify SDK** - API integration

### DevOps
- **Terraform** - Infrastructure as Code
- **GitHub Actions** - CI/CD pipeline
- **Docker** - Containerized Lambda functions

---

## 📈 Performance & Costs

### System Metrics
- **Data ingestion rate**: 1 reading/minute per sensor (60/hour)
- **Lambda executions**: ~4,000/month (data processing + AI checks)
- **DynamoDB reads**: ~10,000/month (dashboard queries)
- **AI API calls**: ~1,500/month (one check per 5 minutes)
- **Dashboard load time**: <2s (cached data)

### Monthly AWS Costs (Estimated)
| Service | Usage | Cost |
|---------|-------|------|
| IoT Core | 43,200 messages/month | $0.22 |
| Lambda | 4,000 invocations (1GB, 30s avg) | $0.83 |
| DynamoDB | 43K writes, 10K reads (on-demand) | $3.47 |
| S3 | 1GB storage + lifecycle | $0.23 |
| SNS | 50 alerts/month | $0.05 |
| Bedrock | 1,500 Claude API calls | $4.50 |
| CloudWatch | Logs + metrics | $2.00 |
| Amplify | Dashboard hosting | $1.00 |
| **Total** | | **~$12.30/month** |

**Scaling:** At 10 devices (10x load): ~$45/month. At 100 devices: ~$280/month.

---

## 🔧 Configuration

### Environment Variables

**Lambda (Data Processor):**
```bash
DYNAMODB_TABLE=sensor-data
SNS_TOPIC_ARN=arn:aws:sns:us-east-1:123456:alerts
LOG_LEVEL=INFO
```

**Lambda (AI Anomaly Detection):**
```bash
ANTHROPIC_API_KEY=sk-ant-xxxxx  # Or use AWS Bedrock
MODEL_NAME=claude-3-5-sonnet-20241022
CONFIDENCE_THRESHOLD=0.70
ANALYSIS_WINDOW_MINUTES=30
```

**Raspberry Pi (sensor_client.py):**
```json
{
  "iot_endpoint": "xxxxx-ats.iot.us-east-1.amazonaws.com",
  "client_id": "pi-zero-2w-001",
  "topic": "sensor/environmental/data",
  "cert_path": "./certs/certificate.pem.crt",
  "key_path": "./certs/private.pem.key",
  "root_ca_path": "./certs/AmazonRootCA1.pem",
  "read_interval_seconds": 60,
  "sensors": {
    "dht22": {"enabled": true, "gpio_pin": 4},
    "mq135": {"enabled": true, "gpio_pin": 17}
  }
}
```

---

## 🧪 Testing

### Unit Tests
```bash
# Lambda functions
cd lambda/data-processor
pytest tests/ -v

cd ../ai-anomaly
pytest tests/ -v
```

### Integration Tests
```bash
# Test full data flow
python3 scripts/integration_test.py

# Expected output:
# ✅ Sensor client publishes data
# ✅ IoT Core receives message
# ✅ Lambda processes and stores to DynamoDB
# ✅ AI anomaly check runs (no alert for normal data)
# ✅ Dashboard displays latest reading
```

### Load Testing
```bash
# Simulate 100 devices publishing every 60 seconds
python3 scripts/load_test.py --devices 100 --duration 600

# Monitor CloudWatch for Lambda throttling
```

---

## 🚀 Deployment

### CI/CD Pipeline (GitHub Actions)

On push to `main`:
1. Run unit tests (Lambda functions)
2. Build Docker images for Lambda
3. Run Terraform plan
4. Manual approval required for `terraform apply`
5. Deploy dashboard to Amplify

```yaml
# .github/workflows/deploy.yml
name: Deploy Smart Monitor
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: pytest
      - name: Terraform Apply
        run: |
          cd terraform
          terraform apply -auto-approve
```

---

## 🔐 Security

- ✅ **IoT certificates**: X.509 certificates for device authentication
- ✅ **Least privilege IAM**: Separate roles for Lambda, IoT, DynamoDB
- ✅ **Encrypted transit**: TLS 1.2+ for all data transmission
- ✅ **Encrypted at rest**: DynamoDB encryption enabled
- ✅ **API key security**: Anthropic API key stored in AWS Secrets Manager
- ✅ **VPC isolation**: Lambda functions in private subnet (optional, disabled by default for cost)

---

## 🐛 Troubleshooting

### Raspberry Pi not sending data
```bash
# Check IoT connection
python3 -c "import test_connection; test_connection.verify_iot()"

# Verify certificates
ls -la certs/
# Ensure: certificate.pem.crt, private.pem.key, AmazonRootCA1.pem exist

# Check systemd logs
sudo journalctl -u smart-monitor -f
```

### Lambda errors
```bash
# Check CloudWatch logs
aws logs tail /aws/lambda/sensor-processor --follow

# Common issues:
# - IAM permissions: Verify Lambda execution role has DynamoDB PutItem
# - Environment variables: Check DYNAMODB_TABLE is set correctly
```

### Dashboard not loading data
```bash
# Verify API Gateway endpoint
curl https://xxxxx.execute-api.us-east-1.amazonaws.com/prod/sensors

# Check CORS configuration in API Gateway
# Ensure dashboard URL is in allowed origins
```

### AI anomaly detection not working
```bash
# Test Claude API directly
python3 scripts/test_ai_api.py

# Check Secrets Manager for API key
aws secretsmanager get-secret-value --secret-id anthropic-api-key

# Verify Lambda has internet access (requires NAT Gateway if in VPC)
```

---

## 🗺️ Roadmap

### Phase 2 (Planned)
- [ ] **Additional sensors**: CO2, light level, motion detection
- [ ] **Multi-location support**: Track multiple devices on single dashboard
- [ ] **ML model training**: Build custom TensorFlow model for faster inference
- [ ] **Mobile app**: React Native dashboard for iOS/Android
- [ ] **Grafana integration**: Advanced visualization and alerting

### Phase 3 (Future)
- [ ] **Edge ML**: Run TensorFlow Lite models directly on Raspberry Pi
- [ ] **Predictive maintenance**: Forecast HVAC failures before they occur
- [ ] **Energy optimization**: AI-driven recommendations for HVAC efficiency
- [ ] **Multi-tenant SaaS**: Deploy as commercial monitoring service

---

## 📝 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 🤝 Contributing

Contributions welcome! Please:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 👨‍💻 Author

**Tre Galloway**
- 🌐 Website: [tregalloway.com](https://tregalloway.com)
- 💼 LinkedIn: [linkedin.com/in/tregalloway](https://linkedin.com/in/tregalloway)
- 📧 Email: tre@tregalloway.com
- 📍 Location: Baton Rouge, LA

*Built as a portfolio project to demonstrate full-stack IoT architecture, cloud infrastructure, and AI integration. Interested in cloud engineering and AI/ML engineering roles.*

---

## 🙏 Acknowledgments

- AWS IoT Core documentation and examples
- Anthropic Claude API for anomaly detection
- Adafruit for excellent sensor libraries
- The open-source community


---

**⭐ If this project helped you, please consider starring it on GitHub!**
