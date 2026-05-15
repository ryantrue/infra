# Infra

Домашняя инфраструктура на single-node k3s-кластере.

## Компоненты

- k3s
- Traefik
- cert-manager
- Home Assistant
- Cloudflare DNS

## Сервер

- Hostname: infra-master
- OS: Debian GNU/Linux 13
- Node IP: 192.168.31.160
- Public IP: 95.84.134.162
- Domain: ryandev.ru
- SSH: 2222/tcp, key-only

## DNS

DNS управляется через Cloudflare.

Wildcard DNS:

*.ryandev.ru

указывает на:

95.84.134.162

SSH-related records должны быть DNS only.

## TLS

TLS выпускает cert-manager через ACME DNS-01 Cloudflare challenge.

ClusterIssuer:

letsencrypt-prod

Cloudflare API token Secret:

cert-manager/cloudflare-api-token

Сертификаты:

- kube-system/ryandev-ru-wildcard-tls
- homeassistant/home-ryandev-ru-tls

## Traefik

Namespace:

kube-system

Dashboard:

https://traefik.ryandev.ru/dashboard/

Dashboard защищён Basic Auth middleware.

Ожидаемый ответ без авторизации:

HTTP/2 401

## Home Assistant

Namespace:

homeassistant

URL:

https://home.ryandev.ru/

Home Assistant опубликован через Traefik IngressRoute.

Ожидаемый ответ на HEAD-запрос:

HTTP/2 405

Это нормально, потому что Home Assistant ожидает GET.

## Структура репозитория

~/infra
├── README.md
├── cert-manager
│   ├── namespace.yaml
│   ├── clusterissuer-letsencrypt-prod.yaml
│   └── wildcard-ryandev-ru.yaml
├── homeassistant
│   ├── namespace.yaml
│   ├── pvc.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── certificate.yaml
│   └── ingressroute.yaml
└── traefik
    ├── middlewares.yaml
    └── traefik-dashboard.yaml

## Применение

kubectl apply -f cert-manager/
kubectl apply -f homeassistant/
kubectl apply -f traefik/

## Проверка

kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get certificate -A
kubectl get ingressroute -A

curl -fsS https://home.ryandev.ru/ | head
curl -I https://traefik.ryandev.ru/dashboard/

## Home Assistant / Apple Home / Matter / Thread

Apple Home подключается через HomeKit Bridge.

Для Matter-over-Thread нужен Thread Border Router.

Подходящие варианты:

- Apple TV 4K
- HomePod mini
- HomePod 2nd gen
- Home Assistant Connect ZBT-1/ZBT-2

Для Zigbee рекомендуется отдельный coordinator:

- Sonoff Zigbee 3.0 USB Dongle Plus
- SMLIGHT SLZB
- Home Assistant Connect ZBT-1/ZBT-2

## Принципы

- Минимум лишних компонентов
- Всё декларативно хранится в ~/infra
- Traefik отвечает за ingress
- cert-manager отвечает за TLS
- Cloudflare отвечает за DNS
- Home Assistant — центральный контроллер
