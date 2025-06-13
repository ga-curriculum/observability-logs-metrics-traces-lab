<h1>
  <span class="headline">Observability: Logs, Metrics, and Traces Lab</span>
  <span class="subhead">Exercise</span>
</h1>

## Observability: Logs, Metrics, and Traces Lab

In software engineering, **observability** refers to how well you can understand the internal state of a system by examining its external outputs — such as logs, metrics, and traces.

A highly observable system makes it easier to:

- Detect and diagnose issues in production
- Understand how software behaves under different conditions
- Respond quickly to failures or unusual behavior

Observability is essential for maintaining **reliability**, especially in complex systems like APIs, microservices, or cloud-native applications. It allows engineers to move from “something’s broken” to “here’s exactly where and why” — faster and with more confidence.

## Prometheus and Grafana

Prometheus and Grafana are considered **core tools in the modern observability stack**, and they are among the most popular open-source tools for monitoring, metrics collection, and visualization.

### What Are These Tools?

| Tool           | Category               | Role in Observability                                                                    |
| -------------- | ---------------------- | ---------------------------------------------------------------------------------------- |
| **Prometheus** | Metrics Collector      | Gathers and stores time-series data from applications (request counts, latencies).       |
| **Grafana**    | Visualization Platform | Visualizes metrics from Prometheus and other sources via dashboards, graphs, and alerts. |

They help answer questions like:

- What’s happening?
- Is something wrong?
- Where should I look to fix it?

### Adoption & Popularity

- **Prometheus** is the de facto standard for open-source metrics. It originated at SoundCloud and is a graduated CNCF project.
- **Grafana** has 10+ million users and is widely used with Prometheus, InfluxDB, and Loki.
- Used by companies like Google, Bloomberg, Red Hat, GitLab, eBay, and Roblox.

### Why They're Widely Adopted

- Fully open-source and free
- Easy to get started locally
- Scales to production environments
- Integrates with modern stacks like Kubernetes and microservices

## What Is Prometheus?

Prometheus is a monitoring system that collects **metrics** — numerical representations of system behavior (request counts, error rates, memory usage).

It works by **scraping** metrics from a `/metrics` endpoint, stores the data, and lets you query it using **PromQL**, or view it with Grafana.

### Why Use Prometheus?

- **Track performance**
- **Catch problems early**
- **Automate alerts**
- **Debug issues in production**

## What Is Grafana?

Grafana turns raw time-series data (like Prometheus metrics) into visual dashboards and alerts.

- Connects to Prometheus
- Builds custom dashboards
- Helps you spot trends, monitor health, and debug visually

## Lab Instructions

### Your Goal

You’ll update a FastAPI app to expose Prometheus metrics and visualize them in Grafana.

### 1. Install `prometheus_client`

```bash
pip install prometheus_client
```

> ⚠️ Note: The prometheus_client library is installed via pip and used inside your Python app.
> Prometheus and Grafana themselves must be installed separately, as they are standalone services. This was done in Setup.

### 2. Add Metrics to FastAPI

Update `app/main.py` to add new imports and a new route called `/metrics`.

```python
from fastapi import FastAPI, Request, Response
from prometheus_client import Counter, generate_latest, CONTENT_TYPE_LATEST  # New Imports
from app.routes import router

REQUEST_COUNT = Counter("app_requests_total", "Total number of requests", [
                        "method", "endpoint"])

app = FastAPI(title="Observability Starter App")
app.include_router(router)


@app.middleware("http")
async def count_requests(request: Request, call_next):
    response = await call_next(request)
    REQUEST_COUNT.labels(method=request.method,
                         endpoint=request.url.path).inc()
    return response

# New Route
@app.get("/metrics")
def metrics():
    return Response(generate_latest(), media_type=CONTENT_TYPE_LATEST)
```

### 3. Run & Test It

```bash
uvicorn app.main:app --reload
```

Visit:

- [http://localhost:8000/](http://localhost:8000/)
- [http://localhost:8000/items/42](http://localhost:8000/items/42)
- [http://localhost:8000/metrics](http://localhost:8000/metrics)

You should now see Prometheus metrics in plain text.

```plaintext
# HELP app_requests_total Total number of requests
# TYPE app_requests_total counter
app_requests_total{endpoint="/",method="GET"} 1.0
app_requests_total{endpoint="/favicon.ico",method="GET"} 1.0
app_requests_total{endpoint="/items/42",method="GET"} 1.0
# HELP app_requests_created Total number of requests
# TYPE app_requests_created gauge
app_requests_created{endpoint="/",method="GET"} 1.749848773836558e+09
app_requests_created{endpoint="/favicon.ico",method="GET"} 1.749848773851161e+09
app_requests_created{endpoint="/items/42",method="GET"} 1.749848798266423e+09
```

**_Keep the server running in your terminal window._**

## Configuring Prometheus to Scrape Metrics

### 1. Open a new terminal window.

### 2. Create a `prometheus.yml` in the root folder.

```bash
touch prometheus.yml
```

### 3. Add This Config

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fastapi-app'
    static_configs:
      - targets: ['localhost:8000']
```

> Prometheus will scrape `http://localhost:8000/metrics` every 15 seconds.

Your file structure should look like this :

```
observability-logs-metrics-traces-lab-starter/
├── app/
│   ├── __init__.py
│   ├── main.py
│   └── routes.py
├── prometheus.yml
├── requirements.txt
├── README.md
└── venv/
```

### 4. Start Prometheus in a second terminal window

For Mac install with Homebrew:

```bash
prometheus --config.file=prometheus.yml
```

**For Windows:**

Make sure `prometheus.yml` is in that same folder, or provide the full path:

```bash
prometheus.exe --config.file=C:\path\to\prometheus.yml
```

**For Linux:**

If they copied the binary somewhere in $PATH (like /usr/local/bin):

```bash
prometheus --config.file=/full/path/to/prometheus.yml
```

### 5. Confirm

Go to [http://localhost:9090](http://localhost:9090) → **Status → Targets**  
You should see `fastapi-app` listed and marked as **UP**.

image

Try this query in the **Graph** tab:

```text
app_requests_total
```

Click **Execute** to see your metric.

image

## Connecting Grafana to Prometheus

### 1. Access Grafana

- [http://localhost:3000](http://localhost:3000)
- Login: `admin / admin`
- Set a new password when prompted

### 2. Add Prometheus as Data Source

- Grafana Logo → **Connections** → **Data Sources**
- `+ Add New Data Source`
- Choose **Prometheus**
- URL: `http://localhost:9090`
- Click **Save & Test**

### 3. Create a Dashboard

- Grafana Logo → **Dashboards** → `+ Create dashboard`
- **`+ Add visialization`**
- Select `Prometheus`
- Select: `app_requests_total` as a query
- See a live graph of your metric

You will see the graph increase as your visits to your routes increase.

### 4. Trigger More Requests

Visit the API endpoints again and watch your dashboard update in real time.

Congrats!

You’ve now successfully built a full observability pipeline:  
**FastAPI app → Prometheus → Grafana**
