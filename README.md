# ขั้นตอนการติดตั้ง kubectl และ minikube

### kubectl
<details>
  <summary>วิธีติดตั้ง kubectl บน Windows</summary>
  
```
curl.exe -LO "https://dl.k8s.io/release/v1.26.0/bin/windows/amd64/kubectl.exe"
```
  
* add path kubectl ที่ edit the system environment
* check client version

```
kubectl version --client
```
</details>

<details>
  <summary>วิธีติดตั้ง kubectl บน Mac os Arm(HomeBrew)</summary>
  
```
brew install kubectl
```

```
kubectl version --client
```

</details>


### Minikube

<details>
  <summary>วิธีติดตั้ง Minikubes บน Windows</summary>
* download และ install

```
New-Item -Path 'c:\' -Name 'minikube' -ItemType Directory -Force
Invoke-WebRequest -OutFile 'c:\minikube\minikube.exe' -Uri 'https://github.com/kubernetes/minikube/releases/latest/download/minikube-windows-amd64.exe' -UseBasicParsing
```
* command ที่ใช้ set path
```
$oldPath = [Environment]::GetEnvironmentVariable('Path', [EnvironmentVariableTarget]::Machine)
if ($oldPath.Split(';') -inotcontains 'C:\minikube'){ `
  [Environment]::SetEnvironmentVariable('Path', $('{0};C:\minikube' -f $oldPath), [EnvironmentVariableTarget]::Machine) `
}
```

* เช็คว่า minikube สามารถใช้งานได้หรือไม่โดนการพิมcommand บน cmd

```
minikube
```

### ผลลัพธ์

![image](https://user-images.githubusercontent.com/87377798/224981785-1ba2d858-edb7-4c18-ac37-27519fcb90c6.png)

</details>

<details>
  <summary>วิธีติดตั้ง Minikubes บน Mac os Arm(HomeBrew)</summary>
  
```
brew install kubectl
```
  
</details>
* Docker Desktop

# ขั้นตอนการdepoly hello-world-rancer
<details>
  <summary>code hello-world.yaml</summary>
  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rancher-deployment
  namespace: spcn21
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rancher
  template:
    metadata:
      labels:
        app: rancher
    spec:
      containers:
      - name: rancher
        image: rancher/hello-world
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: rancher-service
  labels:
    name: rancher-service
  namespace: spcn21
spec:
  selector:
    app: rancher
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
```
  
</details>

