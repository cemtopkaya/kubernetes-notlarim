Kubernetes Secrets, hassas verileri (şifreler, tokenlar, SSH anahtarları) pod'ların dışında tutarak güvenli bir şekilde yönetmeni sağlayan nesnelerdir. ConfigMap'e benzer ancak veriyi `base64` encode ederek (sadece obfuscation) tutar; varsayılan olarak etcd üzerinde şifrelenmez (encryption at rest yapılandırılmalıdır).

---

### 1. Temel Yapı ve Kullanım

Secrets üç şekilde pod'lara entegre edilir:

* **Environment Variables:** Secret verisi ortam değişkeni olarak inject edilir.
* **Volumes:** Secret, dosya sistemine bir dosya (mount) olarak yazılır.
* **ImagePullSecrets:** Kubelet'in private registry'den imaj çekmesi için kullanılır.

#### Secret Oluşturma

```bash
# Literal değerle
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=s3cr3t

# Dosyadan (dosya adı key, içeriği value olur)
kubectl create secret generic ssh-key --from-file=id_rsa=./id_rsa

```

---

### 2. Mimari Yaklaşım ve Güvenlik

Secrets'ın güvenli yönetimi için şu katmanlar kritiktir:

* **Encryption at Rest:** Encryption at Rest, Kubernetes özelinde etcd veritabanında tutulan statik verilerin (Secrets, ConfigMaps, vb.) diskte fiziki olarak şifrelenmiş halde saklanmasıdır. Etcd içindeki verilerin AES-256 ile şifrelenmesi için `EncryptionConfiguration` kaynağı oluşturulmalıdır.
* **RBAC Kısıtlamaları:** `secrets` kaynaklarına erişim `get`, `list`, `watch` yetkileri ile `Role/ClusterRole` bazlı sınırlandırılmalıdır.
  Eğer bir kullanıcıya secrets kaynağı üzerinde get yetkisi verirseniz, o kullanıcı o namespace içindeki tüm secret'ları okuyabilir. Eğer bir kullanıcının sadece belirli bir Secret'a erişmesini istiyorsanız, resourceNames filtresini kullanmalısınız.
  ```yaml
  # Örnek: Sadece 'db-password' isimli secret'a erişim sağlayan kısıtlayıcı Role
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: secret-reader
  rules:
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["db-password"] # Kritik: Sadece bu isimli secret
    verbs: ["get"]
  ```
* **Volume Mount vs Env Var:** Ortam değişkenleri loglarda veya `kubectl describe pod` komutunda sızabilir. Hassas veriler için **Volume Mount** kullanımı daha güvenlidir; pod silindiğinde dosya sistemi ile birlikte kaybolur.
* **Kısıtlı Scope:** Secret'lar namespace-scoped'dır.

---

### 3. İleri Seviye Entegrasyonlar

Üretim ortamlarında yerel Kubernetes Secret'ları yerine dış vault çözümleri tercih edilmelidir:

* **External Secrets Operator (ESO):** AWS Secrets Manager, HashiCorp Vault veya Azure Key Vault gibi dış sistemlerdeki verileri otomatik olarak k8s secret'larına senkronize eder.
* **Secrets Store CSI Driver:** Secret'ları doğrudan pod içine mount eder, k8s secret objesi oluşturmadan doğrudan memory'de tutulmasını sağlar.

---

### 4. Dikkat Edilmesi Gerekenler

* **Base64 Sınırı:** Secret'ların içindeki verilerin `base64` olması bir şifreleme değil, sadece kodlamadır. Asla plaintext secret'ları git reposuna commit etme (git-secrets veya sops kullan).
* **Immutable Secrets:** `immutable: true` flag'i ile secret'ı salt-okunur yaparak konfigürasyon sürüklenmelerini (drift) engelleyebilirsin.
* **Namespace İzolasyonu:** Secret'ların yanlış namespace'lere sızmaması için `NetworkPolicy` ve `RBAC` disiplini şarttır.

```bash
# Literal değerle
kubectl create secret generic db-credentials --from-literal=username=admin --from-literal=password=s3cr3t

# Dosyadan (dosya adı key, içeriği value olur)
kubectl create secret generic ssh-key --from-file=id_rsa=./id_rsa
```

```bash

kubectl create secret generic db-credentials --from-literal=api-key=MTIzNDU2
```

<img width="357" height="167" alt="image" src="https://github.com/user-attachments/assets/f02ee450-e9b6-4ef4-b9d0-bde74bc29f5b" />
<img width="222" height="130" alt="image" src="https://github.com/user-attachments/assets/1755e396-aa23-4fd4-978c-8a857e453061" />
