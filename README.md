# EKS-Start-to-End

EKS Setup Full Flow (End-to-End)
Step 1: Dockerfile Ready karo

Apna HTML app ka Dockerfile likh (already tere pass hai).
Example:

FROM nginx:alpine
COPY . /usr/share/nginx/html


Build kar:

docker build -t myapp:latest .

Step 2: ECR Repository banao

AWS Console → ECR → Create Repository → myapp
ECR URL milega:

xxxxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com/myapp

Step 3: Image Push karo ECR me
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin xxxxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com

docker tag myapp:latest xxxxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest

docker push xxxxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest


👉 Ab image AWS me store ho gayi ✅

Step 4: EKS Cluster Create karo

AWS Console → EKS → Create Cluster →

VPC select kar (jo tu banaya hai, public subnets choose kar agar NodePort/LB test kar raha hai)

IAM Role = AmazonEKSClusterPolicy

Create (15 min lagta hai).

Step 5: Node Group Create karo

Cluster → Compute → Add Node Group

IAM Role = Worker Node policies (EKS Worker Node Policy, CNI, ECR ReadOnly)

Instance = t3.medium (basic ke liye)

Subnet = Public (agar tu direct NodePort use karega)

Auto-assign Public IP = ✅ Enable (testing ke liye)

Create Node Group.

Step 6: Kubectl Connect kar Laptop se
aws eks --region ap-south-1 update-kubeconfig --name my-cluster


Check:

kubectl get nodes


👉 Worker nodes dikh gaye matlab connect ho gaya.

Step 7: Deployment File banao

deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: xxxxxxxxxx.dkr.ecr.ap-south-1.amazonaws.com/myapp:latest
        ports:
        - containerPort: 80


Apply:

kubectl apply -f deployment.yaml

Step 8: Service File banao (LoadBalancer type)

service.yaml

apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 80
  type: LoadBalancer


Apply:

kubectl apply -f service.yaml

Step 9: Service ka IP/DNS lo
kubectl get svc


Output me:

Agar type=LoadBalancer hai → AWS automatically ek ELB banayega aur EXTERNAL-IP / DNS milega.

Browser me khol → http://<EXTERNAL-IP>

👉 Ab tera HTML app AWS EKS pe live hai ✅

🔑 Quick Recap (flow chart jaisa)

Dockerfile → Docker Image

ECR → Store Image

EKS Cluster → Control Plane

Node Group → Worker EC2

Kubectl Config → Connect AWS se

Deployment → Pod create using ECR Image

Service → Expose App (LoadBalancer / NodePort)

kubectl get svc → Access via IP/DNS
