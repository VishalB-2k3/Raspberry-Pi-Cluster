# 🏗️ Pi-hole on K3s Raspberry Pi Cluster 🚀  

This project sets up **Pi-hole** in a **K3s-powered Raspberry Pi cluster**, acting as a **network-wide ad blocker** while utilizing lightweight Kubernetes.  

## 🌟 Features  
✅ Deploy **Pi-hole** using Helm on a K3s Raspberry Pi Cluster  
✅ Manage DNS filtering across all connected devices  
✅ Load balancing and fault tolerance with multiple worker nodes  
✅ Kubernetes-based **scalability** and **easy maintenance**  

---

## 🛠️ Prerequisites  
1️⃣ **4 Raspberry Pis (1 Master + 3 Worker Nodes)** running **Raspberry Pi OS Lite**  
2️⃣ **K3s installed and configured** on all nodes  
3️⃣ **Helm installed** on the master node  
4️⃣ **A static IP setup for your cluster**  

---

## 🚀 Installation Steps  

### **1️⃣ Setting Up K3s Cluster**  
#### 📌 On the Master Node (`192.168.1.43`):  
```bash
curl -sfL https://get.k3s.io | sh - 
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```
Get the K3s token:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

#### 📌 On Each Worker Node:  
```bash
curl -sfL https://get.k3s.io | K3S_URL="https://192.168.1.43:6443" K3S_TOKEN="<TOKEN_HERE>" sh -
```

Verify all nodes are joined:  
```bash
kubectl get nodes
```

### **2️⃣ Installing Helm on the Master Node**  
```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

### **3️⃣ Deploying Pi-hole Using Helm**  
Add the Pi-hole Helm repository:  
```bash
helm repo add mojo2600 https://mojo2600.github.io/pihole-kubernetes/
helm repo update
```

Create a Namespace for Pi-hole:  
```bash
kubectl create namespace pihole
```

Deploy Pi-hole with Helm:  
```bash
helm install pihole mojo2600/pihole --namespace pihole \
  --set service.type=NodePort \
  --set service.nodePort=31771 \
  --set DNS1="8.8.8.8" \
  --set DNS2="8.8.4.4"
```

### **4️⃣ Accessing Pi-hole Web Interface**  
After installation, access Pi-hole at:  
➡️ [http://192.168.1.43:31771/admin/](http://192.168.1.43:31771/admin/)

To retrieve/reset your Pi-hole admin password:  
```bash
kubectl exec -it $(kubectl get pod -n pihole -l app.kubernetes.io/name=pihole -o jsonpath="{.items[0].metadata.name}") -n pihole -- pihole -a -p
```

### **🖥️ Configuring Pi-hole on Your Network**  
To route all DNS traffic through Pi-hole:  
1️⃣ **Device-by-Device Setup:** Manually set the DNS IP to `192.168.1.43` in network settings.  
2️⃣ **Router-Level Setup:** Change your router's DNS to `192.168.1.43` for whole-network ad blocking.  

---

## 📌 Future Enhancements  
📌 Integrate Prometheus + Grafana for monitoring  
📌 Add redundancy with multiple Pi-hole replicas  
📌 Set up auto-scaling based on traffic  
