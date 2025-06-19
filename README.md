# provision_kube_cluster

Ce projet permet de provisionner un cluster Kubernetes de manière automatisée en utilisant `containerd` comme runtime et Calico.

## Prérequis

- Node 'manager'
    - CPU : 2 coeurs minimum
    - Mémoire : 2 GB de RAM minimum
    - Stockage : 20 GB de disque

- Node 'worker'
    - CPU : 1 coeur minimum
    - Mémoire : 2 GB de RAM minimum
    - Stockage : 20 GB de disque

- Adresse MAC et UUID unique pour chaque node
```bash
# Pour vérifier l'adresse MAC
ip link

# Pour vérifier l'UUID
sudo cat /sys/class/dmi/id/product_uuid
```

- Les nodes doivent pouvoir communiquer. Si `ufw` ou `iptables` sont utilisés, assurez-vous d'autoriser les ports suivants :

### Node 'manager'
| **Protocol** | **Direction** | **Port Range** | **Purpose** | **Used By** |
|---|---|---|---|---|
| TCP | Inbound | 6443 | Kubernetes API server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10259 | kube-scheduler | Self |
| TCP | Inbound | 10257 | kube-controller-manager | Self |

### Node 'worker'
| **Protocol** | **Direction** | **Port Range** | **Purpose** | **Used By** |
|---|---|---|---|---|
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10256 | kube-proxy | Self, Load balancers |
| TCP | Inbound | 30000-32767 | NodePort Services† | All |
| UDP | Inbound | 30000-32767 | NodePort Services† | All |

## Installation

1. Clonez ce dépôt :
    ```bash
    git clone https://github.com/antoinezalewski/provision_kube_cluster.git
    cd provision_kube_cluster
    ```

2. Créez votre inventaire statique en créant les groupes :
    - `kube_manager`
    - `kube_node`

3. Exécutez le playbook :
    ```bash
    ansible-playbook provision_kube_cluster.yaml
    ```

## Utilisation

Après le déploiement, initialisez `kubeadm` :
```bash
# Depuis le manager :
sudo kubeadm init --control-plane-endpoint <IP de votre manager>:6443 --pod-network-cidr=10.244.0.0/16
```

Après plusieurs minutes, le cluster est déployé. Configurez l'accès aux commandes `kubectl`
```bash
mkdir -p $HOME/.kube

```

## Structure du projet

- `provision_kube_cluster.yaml` : Fichier de configuration Ansible pour provisionner le cluster Kubernetes.
- `config.toml` : Fichier de configuration containerd
- `crid.yaml` : Fichier de configuration CRI pour Kubernetes.

## Contribution

Les contributions sont les bienvenues ! Ouvrez une issue ou une pull request.

## Licence

Ce projet est sous licence MIT.
