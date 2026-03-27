# prometheus_grafana_minikube
**Description:** We will crate a simple flask app and deploy this app with minikube cluster locally and then track the telemetry data with prometheus and then with grafana we will visulize them. 

# ProjectFlow:
- 1. Crate a simple flask file.
- 2. Create a simple docker file. Then built it.
    ```bash
    docker build -t flask_metrics_app:latest .
    ```
- 3. 