K6:
---

Modern load-testing tool for performance and testing industries.  <br>
**Features:**
1. Configurable load generation 
2. Tests as a code 
3. A Full-featured API 
4. an embedded Javascript engine 
5. Multiple protocol support 
6. Large extention ecosystem 
7. Flexible metrics storage and visualization 
8. Native integration with grafana cloud 

### Installation of K6: 
```BASH
curl -s https://dl.k6.io/key.gpg | sudo apt-key add - 

echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list 

sudo apt update && sudo apt install k6 
```
 
### Example script: 
```
https://github.com/grafana/k6 
```

1. Create test.js script. 
```js
import http from "k6/http";
import { check, sleep } from "k6";

// Test configuration
export const options = {
  thresholds: {
    // Assert that 99% of requests finish within 3000ms.
    http_req_duration: ["p(99) < 3000"],
  },
  // Ramp the number of virtual users up and down
  stages: [
    { duration: "30s", target: 15 },
    { duration: "1m", target: 15 },
    { duration: "20s", target: 0 },
  ],
};

// Simulated user behavior
export default function () {
  let res = http.get("https://domain.com");
  // Validate response status
  check(res, { "status was 200": (r) => r.status == 200 });
  sleep(1);
}
```
Explaination: <br>
- This simulates gradual load increase, instead of sending all traffic at once.<br>
- `target` means how many virtual users will be active.

So this script designed for **stress test** and **overload** the server. <br>
`http_req_duration` 95% requests should finish within 2 seconds.<br>
`http_req_failed` no more than 5% requests failure allowed. <br>

2. Run it with: 
```
k6 run test.js 
```

 How to prevent server from DDOS?
 ------------------------------
1. Rate limiting if behind proxy:

```
http {
  limit_req_zone $binary_remote_addr zone=req_limit_per_ip:10m rate=20r/s;

  server {
    location / {
      limit_req zone=req_limit_per_ip burst=40 nodelay;
    }
  }
}
```
**This means:** <br>

Normal: 20 requests/sec per IP <br>
Bursts up to 40, then block / delay <br>
 
If you're behind **Cloudflare**, enable: <br>
Rate Limiting Rules <br>
Bot Fight Mode <br>
WAF rules <br>

Locust: 
-------
Python based **load testing tool** where you design user behaviour using **python code**.<br>
it simulates **real user workflows**, **not just raw http traffic**, like **k6**. <br>

**open-source load testing tool** that lets you simulate **concurrent users** (called locusts) **sending requests** to your system — **web APIs, websites, or services** — to measure performance and find bottlenecks.

### Install locust library for python:
```
sudo apt install python3.12-venv -y

# Create a virtual environment
python3 -m venv locust-env

# Activate it
source locust-env/bin/activate

# Now install Locust safely
pip install locust
```

### Method 1: Simple test
locustfile.py
```
from locust import HttpUser, task, between

class MyWebsiteUser(HttpUser):
    wait_time = between(1, 3)  # simulated user think time

    @task
    def homepage(self):
        self.client.get("/")
```

Execute:
```
locust -f locustfile.py <br>
```

Now, open web ui in browser. <br>
http://localhost:8089 <br>

#### Configure load test in web UI:
**Number of users** – total concurrent simulated users <br>
**Spawn rate** – how many users start per second <br>
**Host** – base URL of your target (e.g. https://example.com) <br>

<img width="1920" height="1088" alt="image" src="https://github.com/user-attachments/assets/1725bd22-1caa-4de7-9eef-0f70c397e9ff" />

### Method 2: Login test
```
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 3)   # wait between requests (seconds)

    def on_start(self):
        """This runs when a simulated user starts."""
        self.login()

    def login(self):
        payload = {
            "username": "your_username",
            "password": "your_password"
        }
        with self.client.post("/login", json=payload, catch_response=True) as response:
            if response.status_code == 200:
                response.success()
                print("✅ Login successful")
            else:
                response.failure(f"❌ Login failed: {response.text}")

    @task
    def dashboard(self):
        """This is the protected page after login."""
        self.client.get("/dashboard")   # example endpoint

```
Execute:
```
locust -f locustfile.py
```

### Method 3: Headless (CLI) Mode (Without UI )
useful for automation.
```
locust -f locustfile.py --headless -u 100 -r 10 -t 2m --host https://example.com
```

-u 100: simulate 100 users <br>
-r 10: spawn 10 users/sec <br>
-t 2m: test runs for 2 minutes <br>
--host: base target URL <br>
