# Kubernetes (k8s) Notlarım

Kubernetes, dağıtık çalışan konteyner orkestrasyon çerçevesidir. Bunu kullanmak için pod, depolama gibi kaynaklara çeşitli tanımlar yazabiliriz. Sonra bunları bir düğüm kümesine (cluster) uyguluyoruz; bu noktada Kubernetes nerede çalıştırılacağına ve hangi iş yükü için hangi depolama alanı alacağına karar veriyor.

K8s notlarımı aldığım repo


- [Sorunlar ve Çözümleri](sorun-giderme-troubleshooting.md)
- [Kubernetes, etrafındaki terimler ve ilişkileri, kurulumlar](master-worker_kurulumlari.md#kubernetes-etrafındaki-terimler-ve-ilişkileri-kurulumlar)
  - [Kubernetes ve Docker İlişkisi](master-worker_kurulumlari.md#kubernetes-ve-docker-i̇lişkisi)
      - [Docker Nedir?](master-worker_kurulumlari.md#docker-nedir)
      - [Kubernetes Nedir?](master-worker_kurulumlari.md#kubernetes-nedir)
      - [Container Runtime Interface (CRI) Nedir?](master-worker_kurulumlari.md#container-runtime-interface-cri-nedir)
      - [Peki OCI (Open Container Initiative) Nedir?](master-worker_kurulumlari.md#peki-oci-open-container-initiative-nedir)
      - [Konteyner Çalışma Zamanı (container runtime)](master-worker_kurulumlari.md#konteyner-çalışma-zamanı-container-runtime)
        - [Kubernetes'te konteyner çalışma zamanınızı nasıl kontrol edebilirsiniz?](master-worker_kurulumlari.md#kuberneteste-konteyner-çalışma-zamanınızı-nasıl-kontrol-edebilirsiniz)
- [Kubernetes Kuralım](master-worker_kurulumlari.md#kubernetes-kuralım)
  - [Kubernetes 1.24 Sürümünü kubeadm ile Ubuntu Üstünde Kuralım](master-worker_kurulumlari.md#kubernetes-124-sürümünü-kubeadm-ile-ubuntu-üstünde-kuralım)
    - [containerd Kurulumu ve Ayarlarının Yapılması](master-worker_kurulumlari.md#containerd-kurulumu-ve-ayarlarının-yapılması)
    - [Kubernetes Paketlerini Kurmak](master-worker_kurulumlari.md#kubernetes-paketlerini-kurmak)
- [Kubernetes Kümesini Oluşturmak](master-worker_kurulumlari.md#kubernetes-kümesini-oluşturmak)
  - [Kubernetes Bileşenleri](master-worker_kurulumlari.md#kubernetes-bileşenleri)
    - [Kubernetes içindeki parçalar](master-worker_kurulumlari.md#kubernetes-içindeki-parçalar)
      - [Master Node Elemanları:](master-worker_kurulumlari.md#master-node-elemanları)
      - [Worker Node Elemanları:](master-worker_kurulumlari.md#worker-node-elemanları)
    - [Kullanıcı & Servis Hesapları](master-worker_kurulumlari.md#kullanıcı--servis-hesapları)
  - [CNI (Container Network Interface)](master-worker_kurulumlari.md#cni-container-network-interface)
      - [Flannel](master-worker_kurulumlari.md#flannel)
      - [Calico](master-worker_kurulumlari.md#calico)
      - [Canal](master-worker_kurulumlari.md#canal)
      - [Weave Net](master-worker_kurulumlari.md#weave-net)
    - [Flannel Ağ Eklentisini Yüklemek](master-worker_kurulumlari.md#flannel-ağ-eklentisini-yüklemek)
    - [CALICO Ağ Eklentisini Yüklemek](master-worker_kurulumlari.md#calico-ağ-eklentisini-yüklemek)
    - [Weave Net Ağ Eklentisini Yüklemek](master-worker_kurulumlari.md#weave-net-ağ-eklentisini-yüklemek)
  - [Worker Düğümleri Kubernetes Kümesine Eklemek](master-worker_kurulumlari.md#worker-düğümleri-kubernetes-kümesine-eklemek)
  - [Tüm komutların listesi](master-worker_kurulumlari.md#tüm-komutların-listesi)
- [Giriş (ingress)](ingress.md)
- [StatefulSet](statefulset.md)
  - [Basit bir statefulset örneği](statefulset.md#basit-bir-statefulset-örneği)
- [Kalıcı Depolama Birimi ve Talebi (PV & PVC)](pv_pvc.md#persistent-volume-kalıcı-depolama-birimi-ve-persistent-volume-claim-kalıcı-depolama-talebi)
- [Network Policy](network_policy.md#network-policy)
- [HPA (Horizontal Pod Autoscaling)](hpa_autoscaling.md#kuberneteste-otomatik-ölçekleme-atölyesi)

## Örnekler
### Servis oluşturma
- [Servisler Neden Gereklidir?](service.md#neden-servisleri-kullanmalıyız)
- [Yüksek Erişilebilirlik için Servis Oluşturmak](ornek_1_servis_olusturmak.md#yüksek-erişilebilirlik-için-servis-oluşturmak)
  - [Göreviniz](ornek_1_servis_olusturmak.md#göreviniz)
  - [ÇÖZÜM](ornek_1_servis_olusturmak.md#çözüm)
    - [Deployment ile 4 kopya olacak şekilde POD'ları oluşturalım](ornek_1_servis_olusturmak.md#deployment-ile-4-kopya-olacak-şekilde-podları-oluşturalım)
    - [Servis yaratıp POD'ları altına alalım](ornek_1_servis_olusturmak.md#servis-yaratıp-podları-altına-alalım)
    - [Sayfanın geldiğini görmek için busybox POD'unu yaratalım](ornek_1_servis_olusturmak.md#sayfanın-geldiğini-görmek-için-busybox-podunu-yaratalım)
    - [Kaynaklar](ornek_1_servis_olusturmak.md#kaynaklar)

### Node Etiketlerine Göre Dağıtım
- [Düğümlerin Etiketlerine Uygun POD Çalıştırma](ornek_2_node_etiketlerine_gore_deployment.md#düğümlerin-etiketlerine-uygun-pod-çalıştırma)
  - [Göreviniz](ornek_2_node_etiketlerine_gore_deployment.md#göreviniz)
  - [ÇÖZÜM](ornek_2_node_etiketlerine_gore_deployment.md#çözüm)
    - [Tüm düğümleri listeleyin](ornek_2_node_etiketlerine_gore_deployment.md#tüm-düğümleri-listeleyin)
    - [Tüm düğümleri etiketleriyle getirin](ornek_2_node_etiketlerine_gore_deployment.md#tüm-düğümleri-etiketleriyle-getirin)
    - [Pod YAML dosyasını disk=ssd etiketli düğümde çalışacak şekilde oluşturun](ornek_2_node_etiketlerine_gore_deployment.md#pod-yaml-dosyasını-diskssd-etiketli-düğümde-çalışacak-şekilde-oluşturun)
    - [POD'u ayaklandırdığınızda doğru düğümde çalıştığını doğrulayın](ornek_2_node_etiketlerine_gore_deployment.md#podu-ayaklandırdığınızda-doğru-düğümde-çalıştığını-doğrulayın)


# Bir Göç Nasıl Planlanır

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/b2144539-7279-4b5d-8b58-64fed996c290)

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/d160eb2d-0243-4716-aa4e-c0d2b8952207)

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/487c2fef-e0d0-43d5-95ee-4af85e6b5c87)

![image](https://github.com/cemtopkaya/kubernetes-notlarim/assets/261946/e4525256-c547-4320-a232-c0334daa3d13)


