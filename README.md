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


 

 
