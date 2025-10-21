# Case 2: Deploying Mosquitto MQTT Broker on Kubernetes

## Overview
This case demonstrates deploying a Mosquitto MQTT broker on a K3s Kubernetes cluster.  
The broker handles communication between apps running on Kubernetes and IoT devices on the edge.

---

## Folder Structure
synapsis-case2/
├── k8s/
│ ├── deployment.yaml # Mosquitto Deployment
│ ├── service-nodeport.yaml # NodePort Service
│ └── pvc.yaml # Persistent Volume Claim for broker data
└── README.md

---

## Deployment Steps

### 1. Create Persistent Volume Claim
```
sudo kubectl apply -f k8s/pvc.yaml```

### 2. Deploy Mosquitto Broker
```sudo kubectl apply -f k8s/deployment.yaml```

### 3. Expose Broker via NodePort
```sudo kubectl apply -f k8s/service-nodeport.yaml```

### 4. Verify Pods and Service
```sudo kubectl get pods
sudo kubectl get svc mosquitto```
 - Pod should be in Running status
 - Service type NodePort exposes port 1883:30083/TCP

### 5. Persistent Storage Test
 - Enter the pod shell:
```sudo kubectl exec -it <pod-name> -- /bin/sh```
 - Create a test file:
```echo "test data" > /mosquitto/data/test.txt```
 - Exit and verify Persistent Volume:
```sudo kubectl exec -it <pod-name> -- ls -l /mosquitto/data```

- The file persists even if pod is restarted.

---

### Resource Requests & Limits
- CPU and Memory limits are defined in deployment.yaml
- Autoscaler can be added if needed (Horizontal Pod Autoscaler)

### Notes
- NodePort exposes the broker externally via <NodeIP>:<NodePort>
- Persistent storage ensures message data durability across pod restarts
