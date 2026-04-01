# MongoDB and Mongo Express Kubernetes Deployment

This repository contains Kubernetes deployment files for MongoDB and Mongo Express.

**Note:** All resources are deployed in the `mongo` namespace.

## Files Overview

- `mongodb-secret.yaml` - Contains MongoDB credentials (base64 encoded)
- `mongodb-pvc.yaml` - Persistent Volume Claim for MongoDB data storage
- `mongodb-deployment.yaml` - MongoDB deployment configuration
- `mongodb-service.yaml` - MongoDB service (ClusterIP)
- `mongo-express-configmap.yaml` - Configuration for Mongo Express
- `mongo-express-deployment.yaml` - Mongo Express deployment configuration
- `mongo-express-service.yaml` - Mongo Express service (LoadBalancer with NodePort)

## Default Credentials

### MongoDB Database
- Username: `admin`
- Password: `password123`

### Mongo Express Web UI Login
- Username: `admin`
- Password: `admin123`

**Note:** The Mongo Express UI requires these credentials to access the web interface. Once logged in, it will connect to MongoDB using the MongoDB credentials above.

## Deployment Instructions

### 1. Create the namespace first:

```bash
kubectl create namespace mongo
```

### 2. Apply all configurations in order:

```bash
# Create the secret first
kubectl apply -f mongodb-secret.yaml

# Create the ConfigMap
kubectl apply -f mongo-express-configmap.yaml

# Create the PVC
kubectl apply -f mongodb-pvc.yaml

# Deploy MongoDB
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml

# Deploy Mongo Express
kubectl apply -f mongo-express-deployment.yaml
kubectl apply -f mongo-express-service.yaml
```

### 3. Or apply all at once (after creating namespace):

```bash
kubectl create namespace mongo
kubectl apply -f .
```

## Verify Deployment

Check if all pods are running:

```bash
kubectl get pods -n mongo
```

Check services:

```bash
kubectl get services -n mongo
```

Check PVC status:

```bash
kubectl get pvc -n mongo
```

### Troubleshooting

If Mongo Express cannot connect to MongoDB:

1. Check MongoDB pod logs:
```bash
kubectl logs -n mongo deployment/mongodb-deployment
```

2. Check Mongo Express pod logs:
```bash
kubectl logs -n mongo deployment/mongo-express-deployment
```

3. Verify MongoDB service is accessible:
```bash
kubectl exec -n mongo deployment/mongo-express-deployment -- ping mongodb-service
```

4. Test MongoDB connection from within the cluster:
```bash
kubectl run -it --rm debug --image=mongo:latest --restart=Never -n mongo -- mongosh "mongodb://admin:password123@mongodb-service:27017"
```

## Access Mongo Express

### Using LoadBalancer (if supported):
```bash
kubectl get service mongo-express-service -n mongo
```
Access via the external IP on port 8081.

### Using NodePort:
```bash
# Get the node IP
kubectl get nodes -o wide

# Access via: http://<NODE_IP>:30000
```

### Using Port Forwarding (for local testing):
```bash
kubectl port-forward service/mongo-express-service 8081:8081 -n mongo
```
Then access: http://localhost:8081

## Connect to MongoDB

From within the cluster:
```
mongodb://admin:password123@mongodb-service:27017
```

Using port forwarding for external access:
```bash
kubectl port-forward service/mongodb-service 27017:27017 -n mongo
```
Then connect to: `mongodb://admin:password123@localhost:27017`

## Clean Up

To remove all resources:

```bash
kubectl delete -f .
```

To delete the namespace and all resources within it:

```bash
kubectl delete namespace mongo
```

## Security Notes

⚠️ **Important**: The credentials in `mongodb-secret.yaml` are base64 encoded, NOT encrypted. For production use:
1. Use proper secret management (e.g., Sealed Secrets, External Secrets Operator)
2. Change the default passwords
3. Consider using RBAC and network policies
4. Enable authentication and TLS for MongoDB

## Customization

### Change Storage Size
Edit `mongodb-pvc.yaml` and modify the storage request:
```yaml
resources:
  requests:
    storage: 5Gi  # Change as needed
```

### Change Service Type
Edit `mongo-express-service.yaml` to use different service types:
- `ClusterIP` - Internal only
- `NodePort` - Accessible via node IP and port
- `LoadBalancer` - External load balancer (cloud provider dependent)

### Update Credentials
To change credentials, update the base64 encoded values in `mongodb-secret.yaml`:
```bash
echo -n 'your-username' | base64
echo -n 'your-password' | base64