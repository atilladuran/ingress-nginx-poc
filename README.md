# INGRESS-NGINX (PoC-1) 🚀

## 1️⃣ Ingress-Nginx Nedir? 🤔
Ingress-Nginx, Kubernetes ortamında çalışan bir **Ingress Controller**'dır. **Nginx** web sunucusunu kullanarak Kubernetes üzerindeki servisler için **HTTP ve HTTPS yönlendirme** işlemlerini yönetir. Kubernetes **Ingress kaynaklarını** okuyarak, belirlenen trafik yönlendirme kurallarına uygun şekilde **HTTP trafiğini yönlendirir**.

---

## 2️⃣ Ingress-Nginx Hangi Sorunları Çözer? 🛠️
Kubernetes ortamında çalışan uygulamalara dış dünyadan erişim sağlamak için birden fazla yol vardır:
- **🔌 NodePort:** Her servisin belirli bir port üzerinden erişilebilir olmasını sağlar.
- **🌐 LoadBalancer:** Bulut sağlayıcılarının sunduğu yük dengeleyicileri kullanarak erişim sağlar.
- **🚪 Ingress:** Tek bir giriş noktası oluşturarak birden fazla servise yönlendirme yapar.

➡️ **Ingress-Nginx**, **Ingress kaynaklarını** kullanarak tek bir IP adresi üzerinden farklı servislerin farklı **domain** ve **path** bazlı yönlendirilmesini sağlar, böylece yönetimi kolaylaştırır.

---

## 3️⃣ Neden Ingress-Nginx Kullanılmalı? 🎯
- ✔️ **Tek Bir Giriş Noktası:** Birden fazla servisi yönetmek yerine, tek bir **Load Balancer veya IP adresi** üzerinden farklı servislerin erişimini sağlar.
- ✔️ **🔐 Güvenlik:** **TLS/SSL** desteği ile güvenli bağlantılar oluşturabilir.
- ✔️ **📊 Yük Dengeleme:** HTTP trafiğini verimli şekilde dağıtarak performansı artırır.
- ✔️ **🔀 URL Bazlı Yönlendirme:** Farklı yolları (**path**) farklı servislere yönlendirme imkanı sunar.
- ✔️ **⚙️ Gelişmiş Konfigürasyon:** **Rate limiting**, **authentication**, **custom headers** gibi gelişmiş özellikleri destekler.

---

## 4️⃣ Ingress-Nginx Mimarisi ve Çalışma Mantığı 🏗️
- 1️⃣ **Kullanıcı İsteği** → Kullanıcı, belirlenen bir **domain** veya **path** üzerinden Kubernetes cluster'a **HTTP(S) isteği** gönderir.
- 2️⃣ **Ingress Kaynakları** → Kubernetes'te tanımlanan **Ingress kaynakları**, hangi domain ve path’in hangi servise yönlendirileceğini belirtir.
- 3️⃣ **Ingress-Nginx Controller** → **Ingress kaynaklarını** okuyarak **Nginx konfigürasyonlarını oluşturur ve günceller**.
- 4️⃣ **Servislere Yönlendirme** → Gelen HTTP isteğini ilgili **Kubernetes servisine yönlendirir**.
- 5️⃣ **Yanıt Dönüşü** → Servis tarafından dönen yanıt, **Ingress-Nginx üzerinden kullanıcının tarayıcısına iletilir**.

---

## 5️⃣ Ingress-Nginx En İyi Uygulamalar 🏆
- ✅ **TLS/SSL Kullanın** 🔒: Tüm trafiği **HTTPS** üzerinden yönlendirerek güvenliği artırın.
- ✅ **Rate Limiting Uygulayın** 🚦: **DDoS** saldırılarına karşı koruma sağlamak için istek oranlarını sınırlayın.
- ✅ **Güncel ve Optimize Edilmiş Konfigürasyonlar Kullanın** 🛠️: **Nginx yapılandırmasını** ve limitlerini gerektiği gibi ayarlayın.
- ✅ **Health Check ve Monitoring Kullanın** 📈: **Prometheus ve Grafana** ile metrikleri toplayarak durumu izleyin.

---

## 6️⃣ Ortam Bilgileri 🖥️
- **Kubernetes Dağıtımı**: k3s (Hafif olduğu için tercih edildi)
- **İşletim Sistemi**: Ubuntu 24 LTS 🐧
- **Gereksinimler**: MetalLB, Prometheus/Grafana 📊
- **Hedef**: Kubernetes üzerinde **Ingress-Nginx** ile dış dünyaya servis yayınlamak 🌍

---

## 7️⃣ Kurulum Adımları ⚡

### 7.1️⃣ Ön Gereksinimler 🔗
Kubernetes ortamında aşağıdaki bileşenlerin yüklü olması gerekiyor:
```bash
✅ Kubernetes cluster'ı (k3s veya rke2)
✅ Helm (Paket yöneticisi)
✅ Cert-Manager (SSL sertifikaları için)
```
**Eğer bunlar yüklü değilse, önce bu bileşenleri yükleyin!**

---

### 7.2️⃣ K3s Kurulumu ☁️
K3s yüklemek için aşağıdaki komutu çalıştırın:
```bash
curl -sfL https://get.k3s.io | sh -
```
K3s’in çalıştığını kontrol edin:
```bash
systemctl status k3s
```
Kubeconfig ayarlarını yapılandırın:
```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```
Oturum kapandığında bu ayar kaybolur, kalıcı yapmak için:
```bash
echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> ~/.bashrc
source ~/.bashrc
```

---

### 7.3️⃣ Helm Kurulumu 🎩
Helm paket yöneticisini yüklemek için aşağıdaki komutu kullanın:
```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Kurulumun başarılı olup olmadığını kontrol etmek için:
```bash
helm version
```

---

### 7.4️⃣ Nginx-Ingress Kurulumu 📦
Öncelikle **Traefik** kaldırılmalı:
```bash
kubectl get helmchart -A
kubectl delete helmchart traefik -n kube-system
kubectl delete helmchart traefik-crd -n kube-system
```

Ardından **Ingress-Nginx Controller** kurulumu yapılır:
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace
```
Kurulumun başarılı olup olmadığını kontrol etmek için:
```bash
kubectl get pods -n ingress-nginx
```
### Default Nginx Servisi ve Deployment'ı Oluştur

Eğer hali hazırda bir nginx deployment yoksa, aşağıdaki gibi bir nginx deployment ve service oluşturmalısın.
```bash
nginx-deployment.yaml
```
```bash
apiVersion: apps/v1  # Deployment API versiyonu
kind: Deployment  # Kaynak türü: Deployment
metadata:
  name: nginx  # Deployment adı
  namespace: monitoring  # Bu deployment'in içinde bulunduğu namespace
  labels:
    app: nginx  # Uygulama etiketi
spec:
  replicas: 1  # Çalışacak pod sayısı
  selector:
    matchLabels:
      app: nginx  # Pod'ları seçmek için etiket eşlemesi
  template:
    metadata:
      labels:
        app: nginx  # Pod etiketi
    spec:
      containers:
      - name: nginx  # Konteyner adı
        image: nginx:latest  # Kullanılacak Nginx imajı
        ports:
        - containerPort: 80  # Konteynerin dinlediği port
```
```bash
nginx-service.yaml
```
```bash
apiVersion: v1  # Servis API versiyonu
kind: Service  # Kaynak türü: Service
metadata:
  name: nginx  # Servis adı
  namespace: monitoring  # Servisin içinde bulunduğu namespace
spec:
  selector:
    app: nginx  # Hangi pod'lara yönlendirileceğini belirten etiket
  ports:
  - protocol: TCP  # Kullanılan protokol
    port: 80  # Servisin dinlediği port
    targetPort: 80  # Yönlendirme yapılacak konteyner portu
  type: ClusterIP  # Servisin türü (ClusterIP: iç ağda erişilebilir)
```
```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```
---

### 7.5️⃣ MetalLB Kurulumu 🌍
MetalLB’yi yükleyin:
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/manifests/metallb-native.yaml
```
 IP Havuzu Tanımla:
```bash
 metallb-config.yaml dosyası oluştur:
 ```
```bash
apiVersion: metallb.io/v1beta1  # MetalLB API versiyonu
kind: IPAddressPool  # IP havuzu tanımlama
metadata:
  name: first-pool  # IP havuzu adı
  namespace: metallb-system  # MetalLB'nin çalıştığı namespace
spec:
  addresses:
  - 192.168.1.155-192.168.1.158  # Kullanılacak IP aralığı
---
apiVersion: metallb.io/v1beta1  # MetalLB API versiyonu
kind: L2Advertisement  # Layer 2 duyurusu
metadata:
  name: first-advertisement  # Duyuru adı
  namespace: metallb-system  # MetalLB'nin çalıştığı namespace
```
YAML Dosyasını Uygula:
```bash
kubectl apply -f metallb-config.yaml
```
---

### 7.6️⃣ Cert-Manager Kurulumu 🔐
- Cert-Manager’ı kurmak için aşağıdaki komutu çalıştır:
```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```
- Pod’ların çalıştığını kontrol edelim:
```bash
kubectl get pods -n cert-manager
```
---
- cert-manager-selfsigned-issuer.yaml
```bash
apiVersion: cert-manager.io/v1  # Cert-Manager API versiyonu
kind: ClusterIssuer  # Kümeye genel sertifika sağlayıcısı tanımı
metadata:
  name: selfsigned-cluster-issuer  # ClusterIssuer adı
spec:
  selfSigned: {}  # Self-signed sertifika kullanımı
```
```bash
kubectl apply -f cert-manager-selfsigned-issuer.yaml
```
---

### 7.7️⃣ Ingress Kaynaklarının Oluşturulması 🌎
Örnek **ingress-ati-sites.yaml** dosyası oluşturulup uygulanır.
```bash
ingress-ati-sites.yaml
```
```bash
apiVersion: networking.k8s.io/v1  # Ingress API versiyonu
kind: Ingress  # Kaynak türü: Ingress
metadata:
  name: ati-sites  # Ingress kaynağının adı
  namespace: monitoring  # Ingress'in bulunduğu namespace
  annotations:
    cert-manager.io/cluster-issuer: selfsigned-cluster-issuer  # Sertifika yöneticisinin kullanacağı ClusterIssuer
spec:
  ingressClassName: nginx  # Kullanılacak ingress sınıfı
  rules:
  - host: ati.local  # İlk alan adı kuralı
    http:
      paths:
      - path: /  # Tüm yolları yönlendir
        pathType: Prefix  # Yol türü
        backend:
          service:
            name: nginx  # Yönlendirilecek servis adı
            port:
              number: 80  # Servis portu
  - host: ati-duran1.local  # İkinci alan adı kuralı
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
  - host: ati-duran2.local  # Üçüncü alan adı kuralı
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
  tls:
  - hosts:
    - ati.local  # TLS için tanımlanan alan adları
    - ati-duran1.local
    - ati-duran2.local
    secretName: ati-sites-tls  # TLS için kullanılacak gizli anahtar adı
```
```bash
kubectl apply -f ingress-ati-sites.yaml
```
---

## 8️⃣ Test ve Doğrulama 🧪
```bash
kubectl get ingress -A
```
Tüm servislerin çalıştığını doğrulamak için **curl** komutlarını çalıştırın:
```bash
curl -k https://ati.local
curl -k https://grafana.ati.local
curl -k https://prometheus.ati.local
```

🎉 **Kurulum tamamlandı! Artık Kubernetes ortamınızda Ingress-Nginx çalışıyor!** 🚀

---
### Prometheus & Grafana Kurulumu
Prometheus ve Grafana’yı Helm ile kuracağız.
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```
- podları kontrol edelim
```bash
watch kubectl --namespace monitoring get pods -l "release=prometheus"
```
---
```bash
ingress-prometheus.yaml
```
```bash
apiVersion: networking.k8s.io/v1  # Ingress API versiyonu
kind: Ingress  # Kaynak türü: Ingress
metadata:
  name: prometheus  # Ingress kaynağının adı
  namespace: monitoring  # Ingress'in bulunduğu namespace
  annotations:
    cert-manager.io/cluster-issuer: selfsigned-cluster-issuer  # Sertifika yöneticisinin kullanacağı ClusterIssuer
spec:
  ingressClassName: nginx  # Kullanılacak ingress sınıfı
  rules:
  - host: prometheus.ati.local  # Prometheus için alan adı kuralı
    http:
      paths:
      - path: /  # Tüm yolları yönlendir
        pathType: Prefix  # Yol türü
        backend:
          service:
            name: prometheus-kube-prometheus-prometheus  # Yönlendirilecek Prometheus servisi
            port:
              number: 9090  # Prometheus servisi portu
  tls:
  - hosts:
    - prometheus.ati.local  # TLS için tanımlanan alan adı
    secretName: prometheus-tls  # TLS için kullanılacak gizli anahtar adı
```
```bash
kubectl apply -f ingress-prometheus.yaml
```

### Grafana

```bash
grafana-deployment.yaml 
```
```bash
apiVersion: apps/v1  # Deployment API versiyonu
kind: Deployment  # Kaynak türü: Deployment
metadata:
  name: grafana  # Deployment adı
  labels:
    app: grafana  # Uygulama etiketi
spec:
  replicas: 1  # Çalışacak pod sayısı
  selector:
    matchLabels:
      app: grafana  # Pod'ları seçmek için etiket eşlemesi
  template:
    metadata:
      labels:
        app: grafana  # Pod etiketi
    spec:
      containers:
      - name: grafana  # Konteyner adı
        image: grafana/grafana:latest  # Kullanılacak Grafana imajı
        ports:
        - containerPort: 80  # Konteynerin dinlediği port
---
apiVersion: v1  # Servis API versiyonu
kind: Service  # Kaynak türü: Service
metadata:
  name: grafana  # Servis adı
spec:
  selector:
    app: grafana  # Hangi pod'lara yönlendirileceğini belirten etiket
  ports:
    - protocol: TCP  # Kullanılan protokol
      port: 80  # Servisin dinlediği port
      targetPort: 80  # Yönlendirme yapılacak konteyner portu
  type: ClusterIP  # Servisin türü (ClusterIP: iç ağda erişilebilir)
```

---
```bash

grafana-service.yaml 
```
```bash
apiVersion: v1  # Servis API versiyonu
kind: Service  # Kaynak türü: Service
metadata:
  name: prometheus-grafana  # Servis adı
  namespace: monitoring  # Servisin bulunduğu namespace
spec:
  selector:
    app.kubernetes.io/name: grafana  # Hangi pod'lara yönlendirileceğini belirten etiket
  ports:
    - protocol: TCP  # Kullanılan protokol
      port: 80  # Servisin dinlediği port
      targetPort: 3000  # Grafana'nın çalıştığı varsayılan port
  type: ClusterIP  # Servisin türü (ClusterIP: iç ağda erişilebilir)
```

---
```bash
ingress-grafana.yaml 
```
```bash
apiVersion: networking.k8s.io/v1  # Ingress API versiyonu
kind: Ingress  # Kaynak türü: Ingress
metadata:
  name: grafana  # Ingress kaynağının adı
  namespace: monitoring  # Ingress'in bulunduğu namespace
  annotations:
    cert-manager.io/cluster-issuer: selfsigned-cluster-issuer  # Sertifika yöneticisinin kullanacağı ClusterIssuer
spec:
  ingressClassName: nginx  # Kullanılacak ingress sınıfı
  rules:
  - host: grafana.ati.local  # Grafana için alan adı kuralı
    http:
      paths:
      - path: /  # Tüm yolları yönlendir
        pathType: Prefix  # Yol türü
        backend:
          service:
            name: prometheus-grafana  # Yönlendirilecek Grafana servisi
            port:
              number: 80  # Servis portu
  tls:
  - hosts:
    - grafana.ati.local  # TLS için tanımlanan alan adı
    secretName: grafana-tls  # TLS için kullanılacak gizli anahtar adı
```
---
```bash
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
kubectl apply -f ingress-grafana.yaml
```
---
Tarayıcıdan Erişim için Adımlar
1. /etc/hosts Dosyasına Kayıt Ekle

Linux veya macOS için:
```bash
sudo nano /etc/hosts
```
Windows için:
Dosyayı Not Defteri (Yönetici olarak çalıştırarak) aç:
```bash
C:\Windows\System32\drivers\etc\hosts
```
Şu satırları ekleyin (Eğer yoksa):
!!! "Örnek " "Sizde yukarıda ingress lerinize baktığınızda gelen ip olacaktır."
```bash
192.168.1.143 ati.local
192.168.1.143 ati-duran1.local
192.168.1.143 ati-duran2.local
192.168.1.143 grafana.ati.local
192.168.1.143 prometheus.ati.local
```
Kaydedip çıkın:
```bash
    Linux/macOS: CTRL + X → Y → Enter
    Windows: Dosya > Kaydet
```
2. Tarayıcıdan HTTPS ile Erişim

Aşağıdaki adreslerden HTTPS kullanarak erişebilirsiniz:
```bash
    NGINX Sayfası: https://ati.local
    NGINX Sayfası: https://ati-duran1.local
    NGINX Sayfası: https://ati-duran2.local
    Grafana: https://grafana.ati.local
    Prometheus: https://prometheus.ati.local
```
Sertifika self-signed olduğu için tarayıcıda Güvenlik Uyarısı çıkabilir.
Gelişmiş → Devam Et seçeneğiyle geçebilirsiniz.
---
### PROMETHEUS - GRAFANA LOGİN

- Grafana Şifresini Kubernetes Secret İçinden Öğrenme
- Prometheus Stack genellikle şifreyi bir Kubernetes Secret içinde saklar. Mevcut şifreyi görmek için:
```bash
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```
```bash
prom-operator
```

Çıktıda prometheus-kube-prometheus-prometheus gibi bir servis adı görmelisin. Örnek:
```bash
kubectl get svc -n monitoring
```
---
- Prometheus’u Veri Kaynağı Olarak Ekle
    - Veri kaynağı olarak "Prometheus" seç.
    - URL kısmına aşağıdaki Kubernetes servis adresini gir:
```bash
http://prometheus-kube-prometheus-prometheus.monitoring.svc:9090
```

  - Scrape Interval: 15s (Varsayılan olarak bırakabilirsin).
  - HTTP Method: GET olarak bırak.
  - "Save & Test" butonuna tıkla.
---
📢 Geri Bildirim ve Katkı

Bu proje, Ingress-Nginx, MetalLB, Cert-Manager, Longhorn, Prometheus ve Grafana gibi bileşenleri kullanarak Kubernetes ortamında çalışmaktadır.

Eğer projeyi denerken bir hata ile karşılaşırsanız veya geliştirilmesi gerektiğini düşündüğünüz bir nokta varsa issue açabilir ya da pull request göndererek katkıda bulunabilirsiniz.

Geri bildirimlerinizi bekliyorum! 🙌

📩 Bana ulaşmak için:
🔹 LinkedIn: linkedln.com/atilladuran
🔹 E-posta: atilladuran.99@gmail.com

Teşekkürler! 🚀
