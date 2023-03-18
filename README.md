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

# การเตรียมการ 

* Windows add host ใน C:\Windows\System32\drivers\etc/hosts
* Mac add hosts etc/hosts โดยใช้ namo หรือ vim editor

# ขั้นตอนการ depoly hello-world-rancer
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
</details>

<details>
  <summary>อธิบาย hello-world.yaml</summary>
  
### YAML ของ Kubernetes คือการรทำ Deployment และ Service ใน Kubernetes cluster

Deployment ชื่อ "rancher-deployment" ถูกกำหนดไว้ใน namespace "spcn21" และระบุว่าจะมี pod ที่มี label "app: rancher" อยู่ 1 ตัว ภายใน template ของ pod จะถูกกำหนดเพิ่มเติม โดยมี container ชื่อ "rancher" ใช้ image "rancher/hello-world" และ กำหนดport 80

Service ชื่อ "rancher-service" ถูกกำหนดไว้ใน namespace เดียวกัน และใช้ label selector "app: rancher" เพื่อ match กับ pod ที่ถูกสร้างโดย Deployment นั้น ๆ Service นี้จะ expose port 80 โดยใช้ protocol TCP และ map ไปที่ targetPort 80 ของ container ภายใน pod ที่มี label "app: rancher" นี้ Service จะใช้เพื่อให้เป็นตัวชื่อมต่อเครือข่ายของ pod ที่สร้างดเวย Deployment นี้ โดยใช้ได้ทั้งภายใน Kubernetes cluster และภายนอก cluster ตามที่กำหนดไว้ในเงื่อนไขของ Service นี้
  
</details>

<details>
  <summary>code service rancer</summary>
  
```
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: service-ingress
  namespace: spcn21
spec:
  entryPoints:
    - web
    - websecure
  routes:
  - match: Host(`web.spcn21.local`)
    kind: Rule
    services:
    - name: rancher-service
      port: 80

```
<details>
  <summary>อธิบาย service.yaml</summary>
  
### YAML ที่อธิบายการสร้าง IngressRoute ใน Kubernetes โดยใช้ Traefik Ingress Controller ในการติดต่อระหว่าง Service กับ Ingress Controller และเชื่อมต่อกับ EntryPoints ต่างๆที่ถูกกำหนดไว้

IngressRoute ชื่อ "service-ingress" ถูกกำหนดไว้ใน namespace "spcn21" โดยมี entryPoints สองตัวคือ "web" และ "websecure" เพื่อให้ Ingress Controller ของ Traefik รับรู้ว่าจะส่ง HTTP request มาที่ EntryPoints ตัวไหนของ Ingress Controller นี้โดยจะต้องมีการกำหนด Certificate สำหรับ EntryPoints "websecure" ด้วย

มี route เดียวที่ตรงกับ Host(web.spcn21.local) โดยใช้ kind: Rule และให้ forward ไปยัง Service ที่ชื่อว่า "rancher-service" โดยเปิด port 80 ภายใน container ซึ่งถูกกำหนดไว้ใน namespace "spcn21" โดย Service จะส่ง traffic ไปยัง Pod ที่ตรงกับ selector ของ Service นี้ใน namespace เดียวกันกับ Service นั้นๆ
  
</details>

# ขั้นตอนการ deploy
# ผลลัพธ์
# ปัญหาที่พบเจอ
