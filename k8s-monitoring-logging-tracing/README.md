 ## 🏢 Enterprise Real-World Scenario: Observability Platform for a FinTech Client

### Scenario

As a DevOps engineer at a consultancy, I was tasked by a FinTech enterprise to design and implement a **centralized observability platform** for their AWS EKS-based microservices. The client faced:

- **Limited visibility** into application health and performance
- **Slow incident response** due to scattered logs and metrics
- **No distributed tracing** for debugging complex transactions
- **Compliance requirements** for audit-ready monitoring and alerting

---

### Problem

- **Siloed monitoring:** Teams used different tools for metrics, logs, and traces, making correlation difficult.
- **Manual troubleshooting:** Root cause analysis was slow, impacting SLAs.
- **No proactive alerting:** Outages were often reported by customers before engineers.
- **Scaling issues:** Existing solutions couldn’t handle growing traffic and data volume.

---

### Solution

We architected and deployed a **Kubernetes-native observability stack** using:

- **Prometheus & Alertmanager:** Metrics collection and alerting, exposed via NodePort for secure internal access.
- **Grafana:** Unified dashboards for business and technical KPIs.
- **Elasticsearch & Kibana:** Centralized log storage and analytics, with secure access via AWS NLB and custom domain.
- **Fluent Bit:** Lightweight log shipping from all pods to Elasticsearch.
- **Jaeger:** Distributed tracing, sharing the Elasticsearch backend and exposed via LoadBalancer.

**Key features:**
- Automated Helm-based deployments for repeatability
- RBAC and TLS for security and compliance
- Custom dashboards and alert rules for proactive monitoring

---

### Challenges

- **Token synchronization:** Kibana failed to connect to Elasticsearch due to mismatched service account tokens. We automated token retrieval and Helm value updates.
- **LoadBalancer targeting:** AWS NLB initially targeted nodes, not pods, causing health check failures. We fixed this with proper Kubernetes service annotations.
- **Resource tuning:** Initial resource requests were too low, causing pod scheduling issues. We iteratively tuned requests/limits.
- **TLS management:** Coordinating certificates across services required careful secret and configmap management.

---

### Outcome

- **Unified observability:** All logs, metrics, and traces are now centralized and correlated, enabling rapid root cause analysis.
- **Proactive alerting:** Engineers receive actionable alerts before customers notice issues.
- **Audit-ready:** The platform meets compliance requirements for monitoring and access control.
- **Scalable & automated:** The stack scales with business growth and is fully automated for future clusters.

---

### What I Learned

- **Integration details matter:** Small misconfigurations (like tokens) can block the whole stack.
- **Automation is key:** Scripting repetitive tasks (token sync, resource tuning) saves hours.
- **Documentation & knowledge transfer** are critical for client success.
- **Open-source tools** can deliver enterprise-grade observability with the right architecture.

---

*This project demonstrates how a consultancy can deliver a robust, scalable, and secure observability platform for demanding enterprise environments.*
 



 
 
##  the implementation part of the project
 
 
 
 🛠️  Installation & Configurations
## 📦 Step 1: Create EKS Cluster

### Prerequisites
- Download and Install AWS Cli - Please Refer [this]("https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html") link.
- Setup and configure AWS CLI using the `aws configure` command.
- Install and configure eksctl using the steps mentioned [here]("https://eksctl.io/installation/").
- Install and configure kubectl as mentioned [here]("https://kubernetes.io/docs/tasks/tools/").


```bash
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup
```
```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
```
```bash
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability
```

### 🧰 Step 2: Install kube-prometheus-stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### 🚀 Step 3: Deploy the chart into a new namespace "monitoring"
```bash
kubectl create ns monitoring
```
```bash
cd day-2

helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml
```

### ✅ Step 4: Verify the Installation
```bash
kubectl get all -n monitoring
```
- **Prometheus UI**:
```bash
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090
```

**NOTE:** If you are using an EC2 Instance or Cloud VM, you need to pass `--address 0.0.0.0` to the above command. Then you can access the UI on <instance-ip:port>

- **Grafana UI**: password is `prom-operator`
```bash
kubectl port-forward service/monitoring-grafana -n monitoring 8080:80
```
- **Alertmanager UI**:
```bash
kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093
```





# 🎯 Project Objectives
- 🛠️ **Implement Custom Metrics in Node.js Application**: Use the prom-client library to write and expose custom metrics in the Node.js application.
- 🚨 **Set Up Alerts in Alertmanager**: Configure Alertmanager to send email notifications if a container crashes more than two times.
- 📝 **Set Up Logging**: Implement logging on both application and cluster (node) logs for better observability using EFK stack(Elasticsearch, FluentBit, Kibana).
- 📸 **Implement Distributed Tracing for Node.js Application**: Enhance observability by instrumenting the Node.js application for distributed tracing using Jaeger. enabling better performance monitoring and troubleshooting of complex, multi-service architectures.

# 🏠 Architecture
![Project Architecture](images/architecture.gif)

## 1) Write Custom Metrics
- Please take a look at `application/service-a/index.js` file to learn more about custom metrics. below is the brief overview
- **Express Setup**: Initializes an Express application and sets up logging with Morgan.
- **Logging with Pino**: Defines a custom logging function using Pino for structured logging.
- **Prometheus Metrics with prom-client**: Integrates Prometheus for monitoring HTTP requests using the prom-client library:
    - `http_requests_total`: counter
    - `http_request_duration_seconds`: histogram
    - `http_request_duration_summary_seconds`: summary
    - `node_gauge_example`: gauge for tracking async task duration
### Basic Routes:
- `/` : Returns a "Running" status.
- `/healthy`: Returns the health status of the server.
- `/serverError`: Simulates a 500 Internal Server Error.
- `/notFound`: Simulates a 404 Not Found error.
- `/logs`: Generates logs using the custom logging function.
- `/crash`: Simulates a server crash by exiting the process.
- `/example`: Tracks async task duration with a gauge.
- `/metrics`: Exposes Prometheus metrics endpoint.
- `/call-service-b`: To call service b & receive data from service b

## 2) dockerize & push it to the registry
- To containerize the applications and push it to your Docker registry, run the following commands:
```bash
cd day-4

# Dockerize microservice - a
docker build -t <<NAME_OF_YOUR_REPO>>:<<TAG>> application/service-a/ 
# or use abdihakimdevops/microservice-a:v1

# Dockerize microservice - b
docker build -t <<NAME_OF_YOUR_REPO>>:<<TAG>> application/service-b/ 

or use the pre-built images
- abdihakimdevops/microservice-a:v1
- abdihakimdevops/microservice-b:latest

```

## 3) Kubernetes manifest
- Review the Kubernetes manifest files located in `day-4/kubernetes-manifest`.
- Apply the Kubernetes manifest files to your cluster by running:
```bash
kubectl create ns dev

kubectl apply -k kubernetes-manifest/
```

## 4) Test all the endpoints
- Open a browser and get the LoadBalancer DNS name & hit the DNS name with following routes to test the application:
    - `/`
    - `/healthy`
    - `/serverError`
    - `/notFound`
    - `/logs`
    - `/example`
    - `/metrics`
    - `/call-service-b`
- Alternatively, you can run the automated script `test.sh`, which will automatically send random requests to the LoadBalancer and generate metrics:
```bash
./test.sh <<LOAD_BALANCER_DNS_NAME>>
```

## 5) Configure Alertmanager
- Review the Alertmanager configuration files located in `Monitoring-manifests/alerts-alertmanager-servicemonitor-manifest` but below is the brief overview
    - Before configuring Alertmanager, we need credentials to send emails. For this project, we are using Gmail, but any SMTP provider like AWS SES can be used. so please grab the credentials for that.
    - Open your Google account settings and search App password & create a new password & put the password in `Monitoring-manifest/alerts-alertmanager-servicemonitor-manifest/email-secret.yml`
    - One last thing, please add your email id in the `Monitoring-manifests/alerts-alertmanager-servicemonitor-manifest/alertmanagerconfig.yml`
- **HighCpuUsage**: Triggers a warning alert if the average CPU usage across instances exceeds 50% for more than 5 minutes.
- **PodRestart**: Triggers a critical alert immediately if any pod restarts more than 2 times.
- Apply the manifest files to your cluster by running:
```bash
kubectl apply -k alerts-alertmanager-servicemonitor-manifest/
```
- Wait for 4-5 minutes and then check the Prometheus UI to confirm that the custom metrics implemented in the Node.js application are available:
    - `http_requests_total`: counter
    - `http_request_duration_seconds`: histogram
    - `http_request_duration_summary_seconds`: summary
    - `node_gauge_example`: gauge for tracking async task duration

## 6) Testing Alerts
- To test the alerting system, manually crash the container more than 2 times to trigger an alert (email notification).
- To crash the application container, hit the following endpoint
- `<<LOAD_BALANCER_DNS_NAME>>/crash`
- You should receive an email once the application container has restarted at least 3 times.

# 🔍 Logging overview
- Logging is crucial in any distributed system, especially in Kubernetes, to monitor application behavior, detect issues, and ensure the smooth functioning of microservices.

## 📝 Step-by-Step Setup

### 1) Create IAM Role for Service Account
```bash
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster observability \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
- This command creates an IAM role for the EBS CSI controller.
- IAM role allows EBS CSI controller to interact with AWS resources, specifically for managing EBS volumes in the Kubernetes cluster.
- We will attach the Role with service account

### 2) Retrieve IAM Role ARN
```bash
ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --query 'Role.Arn' --output text)
```
- Command retrieves the ARN of the IAM role created for the EBS CSI controller service account.

### 3) Deploy EBS CSI Driver
```bash
eksctl create addon --cluster observability --name aws-ebs-csi-driver --version latest \
    --service-account-role-arn $ARN --force
```
- Above command deploys the AWS EBS CSI driver as an addon to your Kubernetes cluster.
- It uses the previously created IAM service account role to allow the driver to manage EBS volumes securely.

### 4) Create Namespace for Logging
```bash
kubectl create namespace logging
```

### 5) Install Elasticsearch on K8s

```bash
helm repo add elastic https://helm.elastic.co

helm install elasticsearch \
 --set replicas=1 \
 --set volumeClaimTemplate.storageClassName=gp2 \
 --set persistence.labels.enabled=true elastic/elasticsearch -n logging
```
- Installs Elasticsearch in the `logging` namespace.
- It sets the number of replicas, specifies the storage class, and enables persistence labels to ensure
data is stored on persistent volumes.

### 6) Retrieve Elasticsearch Username & Password
```bash
# for username
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
# for password
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```
- Retrieves the password for the Elasticsearch cluster's master credentials from the Kubernetes secret.
- The password is base64 encoded, so it needs to be decoded before use.
- 👉 **Note**: Please write down the password for future reference

### 7) Install Kibana
```bash
helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging
```
- Kibana provides a user-friendly interface for exploring and visualizing data stored in Elasticsearch.
- It is exposed as a LoadBalancer service, making it accessible from outside the cluster.

### 8) Install Fluentbit with Custom Values/Configurations
- 👉 **Note**: Please update the `HTTP_Passwd` field in the `fluentbit-values.yml` file with the password retrieved earlier in step 6: (i.e NJyO47UqeYBsoaEU)"
```bash
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging
```

## ✅ Conclusion
- We have successfully installed the EFK stack in our Kubernetes cluster, which includes Elasticsearch for storing logs, Fluentbit for collecting and forwarding logs, and Kibana for visualizing logs.
- To verify the setup, access the Kibana dashboard by entering the `LoadBalancer DNS name followed by :5601 in your browser.
    - `http://LOAD_BALANCER_DNS_NAME:5601`
- Use the username and password retrieved in step 6 to log in.
- Once logged in, create a new data view in Kibana and explore the logs collected from your Kubernetes cluster.



## 🧼 Clean Up
```bash

helm uninstall monitoring -n monitoring

helm uninstall fluent-bit -n logging

helm uninstall elasticsearch -n logging

helm uninstall kibana -n logging

cd day-4

kubectl delete -k kubernetes-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/


eksctl delete cluster --name observability

```
# 🏠 Architecture
![Project Architecture](images/architecture.gif)



## ⚙️ Setting Up Jaeger

### Step 1: Instrumenting Your Code
- To start tracing, you need to instrument your services. This means adding tracing capabilities to your code. Most popular programming languages and frameworks have libraries or middleware that make this easy.
- We have already instrumented our code using OpenTelemetry libraries/packages. For more details, refer to `day-4/application/service-a/tracing.js` or `day-4/application/service-b/tracing.js`.


### Step 2: Components of Jaeger
- Jaeger consists of several components:
- Agent: Collects traces from your application.
- Collector: Receives traces from the agent and processes them.
- Query: Provides a UI to view traces.
- Storage: Stores traces for later retrieval (often a database like *Elasticsearch*).


### Step 3: Export Elasticsearch CA Certificate
- This command retrieves the CA certificate from the Elasticsearch master certificate secret and decodes it, saving it to a ca-cert.pem file.
```bash
kubectl get secret elasticsearch-master-certs -n logging -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca-cert.pem
```

### Step 4: Create Tracing Namespace
- Creates a new Kubernetes namespace called tracing if it doesn't already exist, where Jaeger components will be installed.
```bash
kubectl create ns tracing
```

### Step 5: Create ConfigMap for Jaeger's TLS Certificate
- Creates a ConfigMap in the tracing namespace, containing the CA certificate to be used by Jaeger for TLS.
```bash
kubectl create configmap jaeger-tls --from-file=ca-cert.pem -n tracing
```
### Step 6: Create Secret for Elasticsearch TLS
- Creates a Kubernetes Secret in the tracing namespace, containing the CA certificate for Elasticsearch TLS communication.
```bash
kubectl create secret generic es-tls-secret --from-file=ca-cert.pem -n tracing
```
### Step 7: Add Jaeger Helm Repository
- adds the official Jaeger Helm chart repository to your Helm setup, making it available for installations.
```bash
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts

helm repo update
```

### Step 8: Install Jaeger with Custom Values
- 👉 **Note**: Please update the `password` field and other related field in the `jaeger-values.yaml` file with the password retrieved earlier in day-4 at step 6: (i.e NJyO47UqeYBsoaEU)"
-  Command installs Jaeger into the tracing namespace using a custom jaeger-values.yaml configuration file. Ensure the password is updated in the file before installation.
```bash
helm install jaeger jaegertracing/jaeger -n tracing --values jaeger-values.yaml
```
### Step 9: Port Forward Jaeger Query Service
- Command forwards port 8080 on your local machine to the Jaeger Query service, allowing you to access the Jaeger UI locally.
```bash
kubectl port-forward svc/jaeger-query 8080:80 -n tracing

```

## 🧼 Clean Up
```bash

helm uninstall jaeger -n tracing

helm uninstall elasticsearch -n logging

# Also delete PVC created for elasticsearch

helm uninstall monitoring -n monitoring

cd day-4

kubectl delete -k app-manifest/

kubectl delete -k alerts-alertmanager-servicemonitor-manifest/

# Delete cluster
eksctl delete cluster --name observability

```


### 🧼 Step 5: Clean UP
- **Uninstall helm chart**:
```bash
helm uninstall monitoring --namespace monitoring
```
- **Delete namespace**:
```bash
kubectl delete ns monitoring
```
- **Delete Cluster & everything else**:
```bash
eksctl delete cluster --name observability
```



