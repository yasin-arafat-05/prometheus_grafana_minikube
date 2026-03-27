# prometheus_grafana_minikube
**Description:** We will crate a simple flask app and deploy this app with minikube cluster locally and then track the telemetry data with prometheus and then with grafana we will visulize them. 

# ProjectFlow:
- 1. Crate a simple flask file.
- 2. Create a simple docker file. Then built it.
    ```bash
    docker build -t flask-metrics-app:latest .
    ```
- 3. Create yml file for kubernetes.
- 4. Start the cluter.
    ```bash
    minikube start --driver=docker
    ```
- 5. load the docker image into our cluster
    ```bash
        # all the image list 
        minikube image list
        # load images
        minikube image load flask-metrics-app:latest  
    ```
- 6. deployment file commands
    ```bash 
    # create:
    kubectl apply -f deployment.yaml

    # delete:
    kubectl delete deployment myapp
    kubectl delete services myapp
    kubectl delete pod myapp-947f4dbd9-xtmqrf

    # troubleshooting:
    kubectl get pods
    ```
- 7. test the app
    ```bash
    # all info:
    minikube service myapp
    # url:
    minikube service myapp --url 
    ```
- 8. Get Promethus by using package manager helm.
    ```bash
    # 1. Install helm in arch
    sudo pacman -S helm

    # 2. Add the prometheus-community repository to your Helm setup and update it to get the  latest chart information.
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update

    # 3. Now install prometheus into a seperate namespace.
    helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace 

    # 4. see all the namespaces
    kubectl get namespaces

    # 5. forwarding the port to see the prometheus within our browser.
    # - all the services inside monitoring cluster:
    kubectl get svc -n monitoring
    # - port forwarding
    kubectl port-forward -n monitoring svc/prometheus-server 9090:80

    # 6. Delete and Installtion-again:
    helm uninstall prometheus -n monitoring
    # - check pvc(persistance volumne)
    kubectl get pvc -n monitoring
    # - delete all the things we we 
    kubectl delete pvc --all -n monitoring
    # update helm:
    helm repo update
    helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace 
    ```
- 9. configure prometheus to scrape flask app:
    ```bash
    # 1. get the clusser-id and port:
    # 10.109.202.205 5050:31132/TCP
    kubeclt get svc 

    # 2. Before editing take a backup anything is happen:
    kubectl get configmap prometheus-server -n monitoring -o yaml > prometheus-server-backup.yaml

    # 3. edit the configuration file of prometheus:
    # find the scrape_configuration: job_name: promethus after that add the below
    # then add 
    #- job_name: flask-app
    #    static_configs:
    #     - targets: ['10.109.202.205:5050']
    export EDITOR=nano
    kubectl edit configmap prometheus-server -n monitoring 

    # 4. see the file content:
    kubectl get configmap prometheus-server -n monitoring -o yaml
    ```

- 10. Install Grafana:
    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo update
    helm install grafana grafana/grafana -n monitoring --create-namespace

    #check:
    kubectl get svc -n monitoring

    #port forwarding:
    kubectl port-forward svc/grafana -n monitoring 3000:80

    #get the password: (username always:-> admin)
    kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

    #while connecting the server url:
    #url must be like: http://prometheus-server.monitoring.svc.cluster.local:80 
    #running on port:80, we can also see 
    kubectl get svc -n monitoring
    ```

