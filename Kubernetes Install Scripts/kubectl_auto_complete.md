### Kubectl Autocomplete

#### Tümü

```sh
sudo apt-get install bash-completion
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc

```

#### Ayrı ayrı

```sh
apt-get install bash-completion
```

veya 

```sh
yum install bash-completion
```

Sistemdeki tüm kullanıcılar için:

```sh
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
```

Tekrar login olunca `kubectl -` yazıp tab tuşuna basınca çıktı şu şekilde gelecek:

![image](https://github.com/user-attachments/assets/95f31ccc-1d29-4d4f-b90d-a2680c737f3b)


### Kısaltma ile kubectl Kullanımı

Kubectl için bir takma adı k olarak ayarlayın.
```sh
echo 'alias k=kubectl' >>~/.bashrc
```

Otomatik tamamlama için takma adı etkinleştirin.
```sh
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

Bash kabuğunuzu yeniden yükleyin. Kullanılabilir seçenekleri görmek ve otomatik tamamlamanın takma adla çalıştığını doğrulamak için `k -` yazın ve ardından sekmeye iki kez basın.
