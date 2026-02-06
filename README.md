# Serverless Uptime & Performance Monitor (GCP)

[![GCP](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)

An automated, cloud-native **Synthetic Monitoring** solution. This project transitions traditional manual QA testing into proactive **Site Reliability Engineering (SRE)** by monitoring application health and latency via a serverless pipeline.

---

## 🏗 Architecture
The system follows a decoupled, event-driven architecture:

1.  **Cloud Scheduler:** Acts as a managed Cron job, triggering the pipeline every 5 minutes.
2.  **Cloud Functions:** A Python-based serverless worker that executes the "Ping" and calculates latency.
3.  **BigQuery:** A data warehouse where execution results are stored for historical auditing.
4.  **Looker Studio:** A business intelligence layer that visualizes system stability.

---

## 🛠 Tech Stack
* **Infrastructure:** Google Cloud Platform (GCP)
* **Compute:** Google Cloud Functions (Python 3.11)
* **Database:** BigQuery (Data Warehousing)
* **Automation:** Cloud Scheduler
* **Visualization:** Looker Studio

---

## 📂 Code Snippets

### `main.py`
The core logic for checking endpoint health:
```python
import requests
import time
from google.cloud import bigquery
from datetime import datetime

def check_site(request):
    client = bigquery.Client()
    url = "[https://your-target-website.com](https://your-target-website.com)"
    
    start_time = time.time()
    try:
        response = requests.get(url, timeout=10)
        status = response.status_code
    except Exception:
        status = 500
    
    latency = time.time() - start_time

    # Data to be inserted
    rows_to_insert = [{
        "url": url,
        "status_code": status,
        "latency": round(latency, 4),
        "timestamp": datetime.utcnow().isoformat()
    }]
    
    # Insert into BigQuery
    client.insert_rows_json("YOUR_PROJECT_ID.monitoring_data.uptime_logs", rows_to_insert)
    return f"Logged {status} for {url}", 200
