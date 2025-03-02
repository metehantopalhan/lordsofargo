# Lords of Argo - Kubernetes Altyapı Kodu

Bu depo, ArgoCD aracılığıyla yönetilen, kapsamlı bir izleme, günlük kaydı ve yönetim yığını uygulayan Kubernetes altyapı yapılandırmalarını içerir.

## Bileşenler

### Temel Altyapı
- **NGINX Ingress Controller**: Servislere dış erişimi yönetir
- **Cert-Manager**: SSL/TLS sertifika yönetimini gerçekleştirir
- **Rancher**: Kubernetes yönetim platformu

### Günlük Kaydı Yığını (ELK)
- **Elasticsearch**:
  - Master Düğümleri: Küme durumunu ve indeksleri yönetir
  - Veri Düğümleri: Veriyi depolar ve işler
  - Koordinatör Düğümleri: İstemci isteklerini ve arama işlemlerini yönetir
- **Kibana**: Elasticsearch için görselleştirme ve yönetim arayüzü
- **Fluent Bit**: Günlük toplama ve iletme

### İzleme Yığını
- **Prometheus**: Metrik toplama ve depolama
- **Grafana**: Metrik görselleştirme ve gösterge paneli
  - Kubernetes küme ve Node Exporter gösterge panelleri önceden yapılandırılmış

## Mimari

### Elasticsearch Kümesi
- 3 Master düğümü (özel kontrol düzlemi)
- Her biri 100GB depolama alanına sahip 3 Veri düğümü
- Yük dengeleme için 3 Koordinatör düğümü

### Ingress Yapılandırması
- LoadBalancer servisi ile NGINX Ingress Controller
- Yapılandırılmış uç noktalar:
  - Kibana: kibana.example.com
  - Grafana: grafana.9.163.80.51.sslip.io
  - Prometheus: prometheus.9.163.80.51.sslip.io
  - Alertmanager: alertmanager.9.163.80.51.sslip.io
  - Rancher: rancher.9.163.80.51.sslip.io

## Ön Koşullar
- Kubernetes kümesi
- ArgoCD kurulu ve yapılandırılmış
- Helm 3.x
- kubectl küme erişimi yapılandırılmış

## Dağıtım

Tüm uygulamalar ArgoCD aracılığıyla dağıtılır ve yönetilir. Her bileşen, ilgili Kubernetes manifestoları ile kendi dizininde tanımlanmıştır:

```
├── elk-coordinator/     # Elasticsearch koordinatör düğümleri
├── elk-data/           # Elasticsearch veri düğümleri
├── elkmaster/          # Elasticsearch master düğümleri
├── fluentbit/          # Günlük toplama
├── grafana/            # Metrik görselleştirme
├── kibana/             # Elasticsearch arayüzü
├── nginx-ingress/      # Ingress controller
├── prometheus/         # Metrik toplama
└── rancher/            # Küme yönetimi
```

## Yapılandırma

### Kaynak Tahsisi
- **Elasticsearch**:
  - Master: 500m-1000m CPU, 1Gi-2Gi Bellek
  - Veri: 1000m-2000m CPU, 2Gi-4Gi Bellek
  - Koordinatör: 500m-1000m CPU, 1Gi-2Gi Bellek
- **Kibana**: 500m-1000m CPU, 1Gi-2Gi Bellek
- **Grafana**: 10Gi kalıcı depolama
- **Prometheus**: 50Gi sunucu depolama

### Güvenlik
- Grafana için varsayılan yönetici kimlik bilgileri: admin/admin123
- Rancher için varsayılan yönetici kimlik bilgileri: admin/admin123
- Elasticsearch'te X-Pack güvenliği devre dışı (üretim için önerilmez)

## Bakım

### Ölçeklendirme
- Elasticsearch düğümleri ilgili YAML dosyalarındaki `replicas` alanı değiştirilerek ölçeklendirilebilir
- Koordinatör düğümleri yük dengeleme için yatay ölçeklendirme yapacak şekilde ayarlanmıştır

### Kalıcılık
- Veri düğümleri: Düğüm başına 100GB
- Master düğümleri: Düğüm başına 10GB
- Grafana: 10GB
- Prometheus: 50GB

## Notlar

- Bu kurulum, veri ve koordinatör düğümleri için yumuşak anti-affinity, master düğümleri için sert anti-affinity kullanır
- Tüm bileşenler ArgoCD aracılığıyla otomatik senkronizasyon ve kendi kendini onarma ile yapılandırılmıştır
- Tüm bileşenler için namespace oluşturma otomatikleştirilmiştir

## Üretim Ortamı Önerileri

1. Elasticsearch'te X-Pack güvenliğini etkinleştirin
2. Tüm ingress uç noktaları için uygun TLS sertifikalarını yapılandırın
3. Gerçek kullanıma göre kaynak tahsislerini gözden geçirin ve ayarlayın
4. Kalıcı veriler için uygun yedekleme stratejileri uygulayın
5. Prometheus/Alertmanager'da uyarı kurallarını yapılandırın

## Katkıda Bulunma

1. Depoyu fork edin
2. Özellik dalınızı oluşturun
3. Değişikliklerinizi commit edin
4. Dalınıza push yapın
5. Yeni bir Pull Request oluşturun