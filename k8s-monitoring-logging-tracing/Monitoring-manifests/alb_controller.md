# ALB Controller Installation

Follow the steps mentioned [here]("https://github.com/iam-veeramalla/aws-devops-zero-to-hero/blob/main/day-22/alb-controller-add-on.md")


 step-by-step commands to set up the **AWS Load Balancer Controller** in your EKS cluster:

---

### **1. Associate an IAM OIDC Provider**
The AWS Load Balancer Controller requires an IAM OIDC provider to associate an IAM role with the Kubernetes service account.

Run the following command:
```bash
eksctl utils associate-iam-oidc-provider --region <region> --cluster <cluster-name> --approve
```

Replace:
- `<region>` with your AWS region (e.g., `us-east-1`).
- `<cluster-name>` with the name of your EKS cluster (e.g., `observability`).

---

### **2. Create the IAM Policy**
Download the required IAM policy for the AWS Load Balancer Controller:
```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

Create the IAM policy using the AWS CLI:
```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam-policy.json
```

Copy the **Policy ARN** from the output. It will look something like this:
```
arn:aws:iam::<account-id>:policy/AWSLoadBalancerControllerIAMPolicy
```

---

### **3. Create an IAM Role for the Controller**
Create an IAM role and associate it with the Kubernetes service account used by the AWS Load Balancer Controller.

Run the following command:
```bash
eksctl create iamserviceaccount \
  --cluster=<cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=<policy-arn> \
  --approve
```

Replace:
- `<cluster-name>` with the name of your EKS cluster (e.g., `observability`).
- `<policy-arn>` with the ARN of the IAM policy you created earlier.

This command will:
- Create a Kubernetes service account named `aws-load-balancer-controller` in the `kube-system` namespace.
- Attach the IAM policy `AWSLoadBalancerControllerIAMPolicy` to the service account.

---

### **4. Install the AWS Load Balancer Controller Using Helm**
Add the EKS Helm repository:
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install the AWS Load Balancer Controller:
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region>
```

Replace:
- `<cluster-name>` with the name of your EKS cluster (e.g., `observability`).
- `<region>` with your AWS region (e.g., `us-east-1`).

---

### **5. Verify the Installation**
Check if the AWS Load Balancer Controller pods are running:
```bash
kubectl get pods -n kube-system
```

You should see a pod named `aws-load-balancer-controller` in the `Running` state.

---

### **6. Reapply Your Ingress Resource**
Reapply your Ingress resource to trigger the creation of the ALB and target groups:
```bash
kubectl apply -f /Users/abdihakimsaid/opt/observability-stack/day-3/ingress_kube_prom_stack.yaml
```

---

### **7. Verify Target Groups**
After reapplying the Ingress resource, check if the target groups are created:
1. Open the **AWS Management Console**.
2. Navigate to **EC2 > Target Groups**.
3. Verify that target groups are created for your services (`prometheus.hakimdevops.art`, `grafana.hakimdevops.art`, `alertmanager.hakimdevops.art`).

---

### **8. Final Verification**
After completing the above steps:
1. Check the Ingress status:
   ```bash
   kubectl get ingress kubernetes-prometheus-stack -n monitoring
   ```
   Ensure the `ADDRESS` field is populated with the ALB DNS name.

2. Verify DNS propagation:
   ```bash
   nslookup prometheus.hakimdevops.art
   ```

3. Access your services:
   - Prometheus: `http://prometheus.hakimdevops.art`
   - Grafana: `http://grafana.hakimdevops.art`
   - Alertmanager: `http://alertmanager.hakimdevops.art`

---
