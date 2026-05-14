# Infra

Домашняя инфраструктура на базе single-node k3s-кластера.

## Архитектура

k3s
├── Traefik
└── Home Assistant

## Сервер

- Hostname: infra-master
- OS: Debian GNU/Linux 13
- Node IP: 192.168.31.160
- Public IP: 95.84.134.162
- Domain: ryandev.ru
- SSH: 2222/tcp, key-only

## Сеть

DNS управляется через Cloudflare.

Поддомены вида:

*.ryandev.ru

должны указывать на:

95.84.134.162

Cloudflare records для SSH должны быть DNS only, без proxy.

## Traefik

Namespace:

kube-system

Адрес dashboard:

https://traefik.ryandev.ru/dashboard/

Dashboard защищён basic-auth middleware.

## TLS

TLS выпускается Traefik через ACME DNS challenge Cloudflare.

Resolver:

cloudflare

Cloudflare API token хранится в Kubernetes Secret:

kube-system/cloudflare-api-token

## Home Assistant

Namespace:

homeassistant

Адрес:

https://home.ryandev.ru/

Ресурсы:

- Namespace
- PVC
- Deployment
- Service
- IngressRoute

Home Assistant публикуется через Traefik без basic-auth, потому что использует собственную авторизацию.

## Структура репозитория

~/infra
├── README.md
├── .backup
├── traefik
│   ├── .env
│   ├── traefik-dashboard.yaml
│   └── middlewares.yaml
└── homeassistant
    ├── namespace.yaml
    ├── pvc.yaml
    ├── deployment.yaml
    ├── service.yaml
    └── ingressroute.yaml

## Добавление нового сервиса

Типовая структура:

<service-name>/
├── namespace.yaml
├── pvc.yaml
├── deployment.yaml
├── service.yaml
└── ingressroute.yaml

Сервис публикуется через Traefik IngressRoute на поддомене:

<service>.ryandev.ru

TLS берётся через Traefik resolver:

cloudflare

## Применение

kubectl apply -f ~/infra/traefik/
kubectl apply -f ~/infra/homeassistant/

## Проверка

kubectl get nodes -o wide
kubectl get ns
kubectl get pods -A -o wide
kubectl get svc -A
kubectl get pvc -A
kubectl get ingressroute -A
helm list -A

## Проверка доменов

curl -kI --resolve home.ryandev.ru:443:192.168.31.160 https://home.ryandev.ru/
curl -kI --resolve traefik.ryandev.ru:443:192.168.31.160 https://traefik.ryandev.ru/dashboard/

## Home Assistant: Apple / Matter / Thread

Для Apple Home используется HomeKit Bridge.

Для добавления HomeKit-устройств в Home Assistant используется HomeKit Device integration.

Для Matter-over-Thread нужен Thread Border Router.

Варианты Thread Border Router:

- Apple TV 4K с Thread
- HomePod mini
- HomePod 2nd gen
- Home Assistant Connect ZBT-1 / ZBT-2
- другой совместимый Thread Border Router

Для Zigbee нужен отдельный Zigbee coordinator:

- Home Assistant Connect ZBT-1 / ZBT-2
- Sonoff Zigbee 3.0 USB Dongle Plus
- SMLIGHT SLZB series

Важно: один USB-радиомодуль обычно не стоит использовать одновременно как основной Zigbee и Thread production-адаптер. Лучше разделить роли.

## Рекомендуемая целевая схема умного дома

Home Assistant
├── Apple Home через HomeKit Bridge
├── Thread Border Router
├── Zigbee coordinator
├── local automations
└── backups

## Основные принципы

- Минимум лишних компонентов
- Всё декларативно в ~/infra
- Traefik отвечает за ingress и TLS
- Cloudflare отвечает за DNS
- Home Assistant остаётся центральным контроллером умного дома
- Apple Home используется как удобный frontend для iPhone/Siri
