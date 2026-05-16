# Infra

Домашняя инфраструктура на single-node Kubernetes-кластере.

## Текущее состояние

- Node: infra-master
- OS: Debian GNU/Linux 13
- Kubernetes: v1.36.1
- Runtime: containerd
- Node IP: 192.168.1.10
- Public IP: 95.84.134.162
- Domain: ryandev.ru
- Git repo: git@github.com:ryantrue/infra.git

## Архитектура

- GitOps через Argo CD
- Один ingress controller: Traefik в kube-system
- TLS через cert-manager и Cloudflare DNS-01
- Home Assistant в namespace homeassistant
- Persistent storage через local-path
- Home Assistant PV переведён в Retain
- Ручной Helm Traefik из namespace traefik удалён

## Компоненты

- Argo CD: argocd
- cert-manager: cert-manager
- Traefik: kube-system
- Home Assistant: homeassistant
- local-path-provisioner: local-path-storage
- kube-flannel: kube-flannel

## Argo CD

URL:

https://argocd.ryandev.ru

Applications:

- infra
- argocd-platform
- cert-manager-infra
- homeassistant
- traefik-infra
- traefik-routes

Ожидаемое состояние:

Synced / Healthy

## Traefik

Namespace:

kube-system

Service:

kube-system/traefik

Type:

NodePort

Ports:

- HTTP: 30080
- HTTPS: 30443

Published service:

kube-system/traefik

Dashboard:

https://traefik.ryandev.ru/dashboard/

Dashboard защищён Basic Auth middleware.

## cert-manager

Namespace:

cert-manager

ClusterIssuer:

letsencrypt-prod

ACME challenge:

Cloudflare DNS-01

Cloudflare token secret:

cert-manager/cloudflare-api-token

Certificates:

- argocd/argocd-ryandev-ru
- homeassistant/home-ryandev-ru
- kube-system/ryandev-ru-wildcard

## DNS

DNS управляется через Cloudflare.

Wildcard record:

*.ryandev.ru -> 95.84.134.162

SSH-related records должны быть DNS only.

## Home Assistant

Namespace:

homeassistant

URL:

https://home.ryandev.ru

Service:

homeassistant/homeassistant:8123

PVC:

homeassistant/homeassistant-config

PV:

pvc-9aff2ea0-e474-4975-aa78-2d99f4bb9824

StorageClass:

local-path

ReclaimPolicy:

Retain

Local path:

/opt/local-path-provisioner/pvc-9aff2ea0-e474-4975-aa78-2d99f4bb9824_homeassistant_homeassistant-config

## Backup

Backup path:

/home/ryan/migration-backup-20260516/homeassistant/

Contains:

- config/
- config-stopped/

Consistent backup procedure:

    kubectl scale deploy homeassistant -n homeassistant --replicas=0
    kubectl rollout status deploy/homeassistant -n homeassistant
    sudo rsync -aHAX --numeric-ids /opt/local-path-provisioner/pvc-9aff2ea0-e474-4975-aa78-2d99f4bb9824_homeassistant_homeassistant-config/ ~/migration-backup-$(date +%Y%m%d)/homeassistant/config-stopped/
    kubectl scale deploy homeassistant -n homeassistant --replicas=1
    kubectl rollout status deploy/homeassistant -n homeassistant

## Repository layout

    ~/infra
    ├── argocd
    ├── cert-manager
    ├── homeassistant
    ├── traefik
    └── README.md

## Проверка

    kubectl get applications -n argocd
    kubectl get pods -A
    kubectl get svc -A
    kubectl get ingressroutes.traefik.io -A
    kubectl get certificates -A
    kubectl get clusterissuer
    kubectl get pvc -A
    kubectl get pv

## Git workflow

    cd ~/infra
    git status
    git add README.md
    git commit -m "Update infrastructure README"
    git push

## Принципы

- Всё прикладное состояние хранится в Git
- Один ingress controller
- Секреты не коммитятся
- TLS выпускает cert-manager
- DNS управляется Cloudflare
- Home Assistant хранит состояние в local-path PVC
- Перед рискованными изменениями PV переводится в Retain и бэкапится
