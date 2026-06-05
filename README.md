# Project-CC
Image classification with Kubernetes HPA 



Step 1 — Open Docker Desktop** (wait for it to fully start)

**Step 2 — Start Minikube:**
```bash
minikube start
```

**Step 3 — Start Prometheus:**
```bash
docker run -d \
  -p 9090:9090 \
  -v /Users/bhavyapatel/Documents/Project/tu-cloud-project/dispatcher/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

**Step 4 — Start Redis:**
```bash
brew services start redis
```

**Step 5 — Terminal 1, minikube tunnel:**
```bash
minikube service tu-cloud-project
```

**Step 6 — Terminal 2, port-forward:**
```bash
kubectl port-forward deployment/tu-cloud-project 6001:6001 8001:8001
```

**Step 7 — Terminal 3, dispatcher:**
```bash
cd ~/Documents/Project/tu-cloud-project/dispatcher
source venv_disp/bin/activate
lsof -i :5001 -i :8000 | awk 'NR>1 {print $2}' | xargs kill -9 2>/dev/null
python3 dispatcher_redis.py
```

**Step 8 — Terminal 4, logger:**
```bash
cd ~/Documents/Project/tu-cloud-project/dispatcher
source venv_disp/bin/activate
echo "Timestamp,P99_Latency,Queue_Size,Replica_Count" > autoscaler_log.csv
python3 autoscaler_logger.py
```

**Step 9 — Terminal 5, traffic:**
```bash
cd ~/Documents/Project/tu-cloud-project/dispatcher
source venv_disp/bin/activate
python3 -c "
import requests, time, threading
def send():
    for i in range(100):
        requests.post('http://127.0.0.1:5001/query', json={'image': 'images/fire_truck.jpeg'})
        time.sleep(0.05)
threads = [threading.Thread(target=send) for _ in range(10)]
for t in threads: t.start()
for t in threads: t.join()
"
```

**Step 10 — analyze results:**
```bash
cd ~/Documents/Project/tu-cloud-project/dispatcher
source venv_disp/bin/activate
python3 analyze_autoscaler_log.py
```
