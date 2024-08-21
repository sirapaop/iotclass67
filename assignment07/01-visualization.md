# Data Visualization.

### Install flowcharting

when we run docker compose up
 ```bash
 docker exec -it grafana /bin/bash
 cd /var/lib/grafana/plugins
 grafana cli plugins install agenty-flowcharting-panel

 ```
 next, we will edit the code in /etc/grafana

 ```bash
 cd /etc/grafana

 ```
 - vim grafana.ini 
 - find angular (/angular)
 - change ;angular_support_enabled = false    to    angular_support_enabled = true
 - restart docker

 ```bash
 docker compose restart grafana
```
