```
 https://github.com/atulkamble/AzureVM-Grafana-Prometheus-Node-Exporter 
  
 // Set Alerting  
  
 // Tip: Dahsboard - Node Exporter Full - 1860 
  
 1. edit file and update  
  
 sudo nano /etc/prometheus/rules/node-alerts.yml 
  
  
 // CPU High Utilization Alert  
  
 groups: 
 - name: node-alerts 
   rules: 
   - alert: HighCPUUsage 
     expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80 
     for: 1m 
     labels: 
       severity: warning 
     annotations: 
       summary: "High CPU usage detected" 
       description: "CPU usage > 80% for more than 1 minute" 
  
 2. update prometheus config file  
  
 sudo nano /etc/prometheus/prometheus.yml 
  
 rule_files: 
   - "rules/*.yml" 
  
  
 3.  
 sudo apt install stress -y  
 stress --version 
 stress --cpu 2 --timeout 120
```
