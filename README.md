# sending-metrics-to-thanos-from-remote-cluster-

# sending-metrics-to-thanos-from-remote-cluster-

# create private s3 bucket in aws and attach following policy
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:DeleteObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::Mention your bucket name*",
                "arn:aws:s3:::Mention your bucket name"
            ]
        }
    ]
}
```

# create one values.yaml file place below content
```
objstoreConfig: |-
  type: s3
  config:
    bucket: Mention your s3 bucket name
    endpoint: s3.us-east-1.amazonaws.com
    access_key: Mention your aws account key
    secret_key: "Mention your aws account key"
    insecure: true 
minio:
  enabled: false
receive:
  enabled: true
  ingress:
    enabled: true
    hostname: thanos-receive.mention your domain name
```


# Install Thanos on server
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm search repo bitnami
helm install thanos bitnami/thanos -f values.yaml
```

# create ingress file ingress.yaml for thanos query and write ingress  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  name: annotated
spec:
  ingressClassName: nginx
  rules:
  - host: thanos-query.your domain name
    http:
      paths:
      - backend:
          service:
            name: Mention thanos-query service name
            port:
              number: Mention thanos-query service port no
        path: /
        pathType: Prefix
```
# Apply ingress 
```
kubectl apply -f ingress.yaml
```

# Access thanos query using host name in the ingress file
```
thanos-query.your domain name
```
# create ingress file ingress.yaml for thanos receive and write ingress  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  name: annotated
spec:
  ingressClassName: nginx
  rules:
  - host: thanos-receive.your domain name
    http:
      paths:
      - backend:
          service:
            name: Mention thanos-receive service name
            port:
              number: Mention thanos-receive service port no
        path: /
        pathType: Prefix
```
# Apply ingress 
```
kubectl apply -f thanos-receive-ingress.yaml
```

# Access thanos receive using host name in the ingress file
```
thanos-receive.your domain name
```
# Connect to prometheus server 
```
export KUBECONFIG=$(pwd)/kluster.yaml
```
# Install Prometheus on another server
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
# Add remoteWrite config to values.yaml under server section
```
remote_write:
- url: http://Mention domain name of thanos receive created using ingress(hostname in ingress file)/api/v1/receive
```
# Add external_labels in values.yaml file under global section
```
external_labels:
      cluster: bc1
```
# Now install prometheus
```
helm install prometheus prometheus-community/prometheus -f values.yaml
```
# create ingress file ingress.yaml for thanos query and write ingress  
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  name: annotated
spec:
  ingressClassName: nginx
  rules:
  - host: prometheus-server.your domain name
    http:
      paths:
      - backend:
          service:
            name: Mention prometheus-server service name
            port:
              number: Mention prometheus-server service port no
        path: /
        pathType: Prefix
```
# Apply ingress 
```
kubectl apply -f ingress.yaml
```
# Access prometheus using host name in the ingress file
```
prometheus-server.your domain name
```
# Now you will see kube metrics in thanos-query UI
