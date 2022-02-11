1. Create AKS
2. choco install kubernetes-helm
3. helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
6. helm repo update

9. deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp1-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp1
  template:
    metadata:
      name: myapp1-pod
      labels: # Dictionary 
        app: myapp1       
    spec:
      containers: # List
        - name: myapp1-container
          image: stacksimplify/kubenginx:1.0.0
          ports:
            - containerPort: 80
		
10. service.yml

apiVersion: v1
kind: Service
metadata:
  name: myapp1-loadbalancer
  labels: 
    app: myapp1
spec:  
  selector:
    app: myapp1
  ports: 
    - port: 80
      targetPort: 80
	  
11. ingress.yml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress-demo
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-staging
    cert-manager.io/acme-challenge-type: http01
spec:  
  rules:  
  - host: "test.selftaughtprogrammers.com"
    http:
      paths:
      - path: /   
        pathType: Prefix     
        backend:
          service:
            name: myapp1-loadbalancer
            port:
              number: 80

14.
Look for pubic ip, get the front end ip, and replace it under step 15

15. note: with/out namespace 

# Use Helm to deploy an NGINX ingress controller
helm install ingress-nginx ingress-nginx/ingress-nginx
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.service.externalTrafficPolicy=Local \
    --set controller.service.loadBalancerIP="REPLACE IP IN STEP 14" 
	

	
16. Check ip for load balancer
17. Go to amazon route 53, add ip record to the hosted zones
			  
			  
5. helm repo add jetstack https://charts.jetstack.io
7. kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.crds.yaml
6. helm repo update
8. helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.1 \
  # --set installCRDs=true

12. clusterissuer.yml

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging  
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: taolaobidaomail@gmail.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
		  
13. certificate.yml

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: ssl-cert-staging
  namespace: default
spec:
  secretName: ssl-cert-staging
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer  
  dnsNames:
  - test.selftaughtprogrammers.com

