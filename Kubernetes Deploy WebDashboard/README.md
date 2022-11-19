# Kubernetes Deploy WebDashboard
Create directory for dashboard and Download configuration file

```
export K8SYAMLPATH=/root/kubernetes # You can change this value
mkdir $K8SYAMLPATH/Dashboard
cd $K8SYAMLPATH/Dashboard 
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
mv recommended.yaml dashboard.yaml
nano dashboard.yaml # To access the Dashboard from external, you need to change value of yaml file
```

### dashboard.yaml
ADD: type: NodePort & nodePort: 30843
※nodePort must be between 30000 and 39999.

```
---
  kind: Service
  apiVersion: v1
  metadata:
    labels:
      k8s-app: kubernetes-dashboard
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard
  spec:
    type: NodePort
    ports:
      - port: 443
        targetPort: 8443
        nodePort: 30843
    selector:
      k8s-app: kubernetes-dashboard

---
```

```
kubectl apply -f dashboard.yaml
kubectl get svc -n kubernetes-dashboard # And remember the ip of node. You can check service this command.
```

To login to the dashboard, you need to create user using yaml file 
```
nano dashboard-users.yaml
```

### dashboard-users.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```
kubectl apply -f dashboard-users.yaml
```

### You can get token using this command

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

You can access to k8s dashborad
https://{master-node-IP}:30843 