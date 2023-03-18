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
* การติดตั้ง traefik [https://github.com/iamapinan/kubeplay-traefik](https://github.com/iamapinan/kubeplay-traefik)

* windows

![Screenshot 2023-03-19 000751](https://user-images.githubusercontent.com/87377798/226122046-72c3a492-241c-4862-8667-147ecb17ca29.png)

* MAC OS

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
</details>

<details>
  <summary>อธิบาย service.yaml</summary>
  
### YAML ที่อธิบายการสร้าง IngressRoute ใน Kubernetes โดยใช้ Traefik Ingress Controller ในการติดต่อระหว่าง Service กับ Ingress Controller และเชื่อมต่อกับ EntryPoints ต่างๆที่ถูกกำหนดไว้

IngressRoute ชื่อ "service-ingress" ถูกกำหนดไว้ใน namespace "spcn21" โดยมี entryPoints สองตัวคือ "web" และ "websecure" เพื่อให้ Ingress Controller ของ Traefik รับรู้ว่าจะส่ง HTTP request มาที่ EntryPoints ตัวไหนของ Ingress Controller นี้โดยจะต้องมีการกำหนด Certificate สำหรับ EntryPoints "websecure" ด้วย

มี route เดียวที่ตรงกับ Host(web.spcn21.local) โดยใช้ kind: Rule และให้ forward ไปยัง Service ที่ชื่อว่า "rancher-service" โดยเปิด port 80 ภายใน container ซึ่งถูกกำหนดไว้ใน namespace "spcn21" โดย Service จะส่ง traffic ไปยัง Pod ที่ตรงกับ selector ของ Service นี้ใน namespace เดียวกันกับ Service นั้นๆ
  
</details>

# ขั้นตอนการ deploy rancher/hello-world

* deploy โดยใช้คำสั่ง kubectl apply -f แก้ไขตามชื่อไฟล์ yaml

* ขั้นตอนที่ 1 deploy hello-world.yaml

```
kubectl apply -f hello-world.yaml
```

* ขั้นตอนที่ 2 deploy service.yaml

```
kubectl apply -f service.yaml
```

* ขั้นตอนที่ 3 หลังจาก deploy ให้รันคำสั่ง minikube tunnel

```
minikube tunnel
```

# ผลลัพธ์
* หน้า dashboard ของ kubernetes (ใช้คำสั่งด้านล่างในการรัน)

```
 minikube dashboard
```

![image](https://user-images.githubusercontent.com/87377798/226121006-93e390b0-5218-45ee-a54c-56df1bfd6431.png)

* หน้า service ของ kubernetes

![image](https://user-images.githubusercontent.com/87377798/226121630-fcb4fbe1-afe4-4e28-bf91-5b845f14a488.png)


* หน้า dashboard ของ traefik [traefik.spcn21.local](traefik.scpcn21.local)

![image](https://user-images.githubusercontent.com/87377798/226121043-31aee760-34c0-4b09-ae0a-148b0a1591ba.png)

* หน้าแสดงผล http router ของ traefik

![image](https://user-images.githubusercontent.com/87377798/226121071-a571cccc-8fee-4f6e-b080-e4359f441668.png)

* หน้าแสดงผลของ rancher [web.spcn21.local](web.spcn21.local)

![image](https://user-images.githubusercontent.com/87377798/226121126-507233c4-dd09-44ed-ae33-9f6d62c54f29.png)

* หน้าแสดงผลของ rancher(mac)

<img width="1440" alt="Screenshot 2566-03-18 at 05 13 23" src="https://user-images.githubusercontent.com/87377798/226121902-096636bd-4324-40f5-84e9-77d87073bebe.png">


* wakatime kube project

[![wakatime](https://wakatime.com/badge/github/SiwakornEDZ/Kube.svg)](https://wakatime.com/badge/github/SiwakornEDZ/Kube)

# ปัญหาที่พบเจอ

* หลังจากที่ทำการรันผ่าน mac os arm ผ่าน docker พบปัญหาว่าบาง service ยังไม่รองรับเช่น ingress ของ traefik การแก้ปัญหาเบื้องต้นอาจจะต้องไปรันใน vmware (ปัจจุบันยังไม่รองรับ)

# REF
* [https://github.com/iamapinan/kubeplay-traefik](https://github.com/iamapinan/kubeplay-traefik)
* [https://minikube.sigs.k8s.io/docs/handbook/accessing/#access-to-ports-1024-on-windows-require](https://minikube.sigs.k8s.io/docs/handbook/accessing/#access-to-ports-1024-on-windows-require)
* [https://kubernetes.io/docs/setup/](https://kubernetes.io/docs/setup/)
* [https://minikube.sigs.k8s.io/docs/start/](https://minikube.sigs.k8s.io/docs/start/)
* [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)

