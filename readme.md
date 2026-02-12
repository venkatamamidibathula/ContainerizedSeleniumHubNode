
#Architecture


┌─────────────────┐    ┌──────────────────┐    ┌──────────────────┐
│   Test Client   │───▶│   selenium-hub-0 │◄──▶│selenium-node-chrome-X│
│ (App Teams)     │    │ (StatefulSet 1)  │    │   (StatefulSet N)  │
└─────────────────┘    └──────────┬───────┘    └──────────────────┘
                                 │
                    ┌────────────▼─────────────┐
                    │ selenium-hub (headless)  │
                    │  DNS: hub-0.hub.ns.svc   │
                    └──────────┬───────────────┘
                               │
                    ┌──────────▼──────────┐
                    │selenium-nodes       │
                    │(headless service)   │
                    └─────────────────────┘




Headless Service (clusterIP: None): No load balancing, returns direct Pod IPs/DNS

StatefulSet: Guarantees stable Pod identity (selenium-hub-0, selenium-node-chrome-1)

Stable DNS: selenium-hub-0.selenium-hub.default.svc.cluster.local


Pod Crash → StatefulSet recreates → SAME Pod name + DNS → Tests continue uninterrupted



| Problem      | Deployment Solution         | StatefulSet + Headless Solution           |
| ------------ | --------------------------- | ----------------------------------------- |
| Node crashes | New random name, tests fail | node-chrome-1 reborn at same DNS          |
| Discovery    | Unpredictable IPs           | Stable node-chrome-1.nodes.svc            |
| Scaling      | LoadBalancer hides nodes    | Direct access + Console shows exact nodes |
| Debugging    | Generic pod names           | hub-0, chrome-2, firefox-1                |



```python


from selenium import webdriver
from selenium.webdriver.remote.webdriver import WebDriver as RemoteWebDriver
from selenium.webdriver.chrome.options import Options

# Point to your Grid endpoint
hub_url = "http://selenium-grid-lb:4444"  # Service DNS

chrome_options = Options()
chrome_options.set_capability("browserName", "chrome")
chrome_options.set_capability("browserVersion", "latest")

driver = RemoteWebDriver(
    command_executor=f"{hub_url}/wd/hub",
    options=chrome_options
)


```



