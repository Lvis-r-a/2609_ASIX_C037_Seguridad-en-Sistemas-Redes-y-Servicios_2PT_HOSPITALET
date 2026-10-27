````markdown
# Seguridad en el DNS: análisis con Wireshark y DNS sobre HTTPS

| Datos de la práctica | Información |
|---|---|
| Módulo | 037 — Seguridad en servicios |
| Ciclo formativo | Administración de Sistemas Informáticos en Red — 2.º de ASIX |
| Actividad | 03_M037T1A3 — DNS + Wireshark |
| Entorno | Ubuntu Server y Debian |
| Servicios utilizados | BIND 9, Kea DHCP, encaminamiento IPv4 y NAT |
| Herramientas de análisis | Wireshark, nslookup, dig y Mozilla Firefox |

> Documento elaborado a partir del enunciado y de la configuración recogida en la memoria de la práctica. Los comandos de comprobación y las mejoras propuestas se diferencian de los resultados documentados. No se atribuyen a las capturas recuentos de paquetes, direcciones de respuesta ni resultados que no puedan verificarse.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

## Índice

1. [Introducción](#1-introducción)
2. [Infraestructura y configuración previa](#2-infraestructura-y-configuración-previa)
3. [Pregunta 1. Análisis del DNS tradicional](#3-pregunta-1-análisis-del-dns-tradicional)
4. [Pregunta 2. DNS sobre HTTPS en Firefox](#4-pregunta-2-dns-sobre-https-en-firefox)
5. [Comparación de resultados](#5-comparación-de-resultados)
6. [Correcciones técnicas de la memoria inicial](#6-correcciones-técnicas-de-la-memoria-inicial)
7. [Conclusiones](#7-conclusiones)

## 1. Introducción

El objetivo de esta práctica es analizar el funcionamiento del servicio DNS en una infraestructura cliente-servidor y comparar las consultas tradicionales con las realizadas mediante DNS sobre HTTPS.

Para ello se utiliza una máquina Debian conectada a una red interna y un servidor Ubuntu con dos interfaces de red. El servidor proporciona los servicios DHCP y DNS, además de actuar como puerta de enlace hacia Internet.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

La actividad se divide en dos bloques:

1. Realizar consultas DNS a tres dominios de Internet y analizar los paquetes mediante Wireshark.
2. Activar DNS sobre HTTPS en el navegador del cliente, repetir las consultas y estudiar los cambios en el tráfico.

La diferencia principal que se pretende comprobar es la **confidencialidad** de las consultas: con el DNS tradicional se puede inspeccionar el nombre consultado directamente en los paquetes; con DoH, el contenido de la consulta viaja dentro de una conexión HTTPS cifrada.  [firefox-admin-docs.mozilla](https://firefox-admin-docs.mozilla.org/reference/policies/dnsoverhttps/)

### 1.1. Alcance de las evidencias

La memoria aportada documenta la siguiente configuración:

- Servidor Ubuntu con interfaces `ens33` y `ens37`.
- Red interna `192.168.1.0/24`.
- Dirección interna del servidor `192.168.1.1`.
- Cliente Debian identificado en la memoria con la dirección `192.168.1.10`.
- BIND 9 con reenviadores `8.8.8.8` y `8.8.4.4`.
- Kea DHCP con un rango de concesiones entre `192.168.1.10` y `192.168.1.100`.
- Firefox y Cloudflare para la prueba de DNS sobre HTTPS.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

> Las imágenes enlazadas en este documento deben guardarse dentro de `image capture/`, junto al directorio que contiene este `README.md`. Se utiliza la sintaxis `(<./image capture/archivo.png>)` para que los espacios de la ruta no interrumpan el enlace Markdown.

## 2. Infraestructura y configuración previa

### 2.1. Topología del laboratorio

```text
                     RED INTERNA: 192.168.1.0/24

┌──────────────────────────┐
│ Cliente Debian           │
│ IP: 192.168.1.10         │
│ Gateway: 192.168.1.1     │
│ DNS: 192.168.1.1         │
│ Firefox y Wireshark      │
└────────────┬─────────────┘
             │
             │ Red interna del hipervisor
             │
┌────────────▼─────────────────────────┐
│ Ubuntu Server                        │
│                                      │
│ ens37: 192.168.1.1/24                 │
│ - Kea DHCP                           │
│ - BIND 9                             │
│ - Encaminamiento IPv4                │
│ - NAT/MASQUERADE                     │
│                                      │
│ ens33: dirección obtenida por DHCP   │
└────────────┬─────────────────────────┘
             │
             │ Adaptador NAT o puente
             ▼
       Red exterior / Internet
             │
             ├── Reenviadores DNS de Google
             │   8.8.8.8 y 8.8.4.4
             │
             └── Proveedor DoH de Cloudflare
```

La dirección externa `192.168.103.129` aparece en el esquema de la memoria como dirección de `ens33`. Es una dirección privada, no una dirección pública de Internet. Si el adaptador utiliza NAT del hipervisor, existe otra traducción de direcciones antes de alcanzar Internet.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

![Topología y adaptadores de las máquinas virtuales](<./image capture/topologia_adaptadores_red.png>)

### 2.2. Configuración de red del servidor

La configuración Netplan documentada utiliza DHCP en la interfaz externa y una dirección fija en la interfaz interna:

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: true
      dhcp6: true
    ens37:
      addresses:
        - 192.168.1.1/24
```

La interfaz interna necesita una dirección estable porque los clientes la utilizan como servidor DNS y puerta de enlace.

Antes de aplicar cambios de red, se recomienda comprobar los nombres reales de las interfaces:

```bash
ip -br address
ip route
```

Para aplicar la configuración de forma controlada:

```bash
sudo netplan try
```

Después de verificar la conectividad:

```bash
sudo netplan apply
```

> La red interna y la red de la interfaz externa no deben solaparse. Si la red física o la red NAT del hipervisor también utiliza `192.168.1.0/24`, es necesario cambiar el direccionamiento de uno de los segmentos.

![Configuración Netplan del servidor Ubuntu](<./image capture/servidor_netplan.png>)

### 2.3. Servicio DNS: BIND 9

#### Instalación

```bash
sudo apt update
sudo apt install bind9 bind9utils dnsutils -y
```

La memoria configura los reenviadores de Google en `/etc/bind/named.conf.options`. BIND recibe las consultas del cliente y, cuando no puede responder desde sus datos locales o su caché, consulta a los reenviadores.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

#### Configuración revisada

La siguiente configuración amplía la original para restringir explícitamente el servicio a la red interna y al propio servidor:

```conf
acl clientes_laboratorio {
    localhost;
    192.168.1.0/24;
};

options {
    directory "/var/cache/bind";

    recursion yes;

    allow-query {
        clientes_laboratorio;
    };

    allow-recursion {
        clientes_laboratorio;
    };

    allow-query-cache {
        clientes_laboratorio;
    };

    listen-on {
        127.0.0.1;
        192.168.1.1;
    };

    listen-on-v6 {
        ::1;
    };

    forwarders {
        8.8.8.8;
        8.8.4.4;
    };

    forward only;

    dnssec-validation auto;

    allow-transfer {
        none;
    };
};
```

Esta es una propuesta de configuración revisada; no debe confundirse con una transcripción literal de la captura original.

Las directivas más importantes son:

| Directiva | Función |
|---|---|
| `recursion yes` | Permite resolver consultas recursivas para los clientes autorizados. |
| `allow-query` | Restringe quién puede realizar consultas. |
| `allow-recursion` | Restringe quién puede utilizar la resolución recursiva. |
| `allow-query-cache` | Controla el acceso a los datos almacenados en caché. |
| `listen-on` | Limita las direcciones IPv4 en las que escucha BIND. |
| `forwarders` | Define los resolutores a los que se reenvían consultas. |
| `forward only` | Evita recurrir a resolución iterativa directa si los reenviadores no responden. |
| `dnssec-validation auto` | Habilita la validación DNSSEC con gestión automática del ancla de confianza correspondiente. |

El uso explícito de ACL y de `forward only` hace que el comportamiento del servidor sea coherente con el papel de reenviador previsto en el laboratorio.  [bind9.readthedocs](https://bind9.readthedocs.io/en/v9.20.23/chapter6.html)

> DNSSEC y cifrado no son equivalentes. Activar la validación DNSSEC no convierte el tráfico DNS del puerto 53 en tráfico cifrado.

#### Comprobación del servicio

```bash
sudo named-checkconf
sudo systemctl restart bind9
sudo systemctl status bind9 --no-pager
sudo ss -lntup | grep ':53'
```

Comprobación funcional:

```bash
dig @192.168.1.1 www.u-tokyo.ac.jp A
```

El campo `SERVER` de la salida permite comprobar qué servidor ha recibido la consulta.

![Configuración y comprobación de BIND 9](<./image capture/bind9_configuracion_estado.png>)

### 2.4. Servicio DHCP: Kea

El servicio DHCP entrega automáticamente la configuración IPv4 al cliente.

La configuración recogida en la memoria utiliza los siguientes parámetros:  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

| Parámetro | Valor |
|---|---|
| Interfaz de servicio | `ens37` |
| Subred | `192.168.1.0/24` |
| Rango de concesiones | `192.168.1.10`–`192.168.1.100` |
| Puerta de enlace | `192.168.1.1` |
| Servidor DNS | `192.168.1.1` |
| Duración de la concesión | 4000 segundos |
| Temporizador de renovación | 1000 segundos |
| Temporizador de revinculación | 2000 segundos |

Instalación:

```bash
sudo apt install kea-dhcp4-server -y
```

Archivo `/etc/kea/kea-dhcp4.conf`:

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": [
        "ens37"
      ]
    },
    "lease-database": {
      "type": "memfile",
      "persist": true,
      "name": "/var/lib/kea/kea-leases4.csv"
    },
    "valid-lifetime": 4000,
    "renew-timer": 1000,
    "rebind-timer": 2000,
    "option-data": [
      {
        "name": "domain-name-servers",
        "data": "192.168.1.1"
      },
      {
        "name": "routers",
        "data": "192.168.1.1"
      }
    ],
    "subnet4": [
      {
        "id": 1,
        "subnet": "192.168.1.0/24",
        "pools": [
          {
            "pool": "192.168.1.10-192.168.1.100"
          }
        ]
      }
    ]
  }
}
```

Validación y puesta en marcha:

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
sudo systemctl restart kea-dhcp4-server
sudo systemctl status kea-dhcp4-server --no-pager
```

Si el servicio no arranca:

```bash
sudo journalctl -u kea-dhcp4-server -b --no-pager
```

> No se deben modificar indiscriminadamente los permisos de `/etc/kea` o `/run/kea`. Primero se identifica el error en los registros y después se corrige el propietario o permiso concreto. Además, `/run` contiene datos de ejecución que pueden recrearse al arrancar.

![Configuración del servicio Kea DHCP](<./image capture/kea_dhcp_configuracion.png>)

### 2.5. Encaminamiento y NAT

El cliente necesita que Ubuntu reenvíe los paquetes entre la interfaz interna y la externa.

#### Activación persistente del reenvío IPv4

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-laboratorio-router.conf
sudo sysctl --system
```

Comprobación:

```bash
sysctl net.ipv4.ip_forward
```

Resultado esperado:

```text
net.ipv4.ip_forward = 1
```

#### Traducción de direcciones

La siguiente regla limita el enmascaramiento a la subred del laboratorio:

```bash
sudo iptables -t nat -A POSTROUTING \
  -s 192.168.1.0/24 \
  -o ens33 \
  -j MASQUERADE
```

Si el cortafuegos restringe el reenvío, se necesitan reglas que permitan el tráfico de salida y las respuestas:

```bash
sudo iptables -A FORWARD \
  -i ens37 -o ens33 \
  -s 192.168.1.0/24 \
  -m conntrack --ctstate NEW,ESTABLISHED,RELATED \
  -j ACCEPT

sudo iptables -A FORWARD \
  -i ens33 -o ens37 \
  -d 192.168.1.0/24 \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT
```

Estos comandos son una referencia para un laboratorio controlado. Antes de ejecutarlos se debe revisar el conjunto de reglas existente para evitar duplicidades o conflictos con otras herramientas de cortafuegos.

Comprobación:

```bash
sudo iptables -S FORWARD
sudo iptables -t nat -L POSTROUTING -n -v
```

#### Persistencia

```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

Con `iptables-persistent`, las reglas IPv4 se guardan normalmente en:

```text
/etc/iptables/rules.v4
```

> Guardar un archivo como `/etc/iptables.rules.v4` no hace que se restaure automáticamente. Debe coincidir con la ruta utilizada por el mecanismo de carga o existir una configuración adicional que lo restaure.

La memoria original documenta el uso de IP forwarding y MASQUERADE; esta versión separa el encaminamiento, la traducción de direcciones y el filtrado para evitar tratarlos como una única función.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

![Reenvío IPv4 y reglas NAT](<./image capture/servidor_forwarding_nat.png>)

### 2.6. Comprobación de la configuración del cliente

En Debian se comprueba:

```bash
ip -br address
ip route
nmcli device show
cat /etc/resolv.conf
```

La configuración esperada es:

```text
Dirección IPv4:   192.168.1.10/24
Puerta de enlace: 192.168.1.1
Servidor DNS:    192.168.1.1
```

La dirección del cliente puede variar dentro del rango DHCP si no existe una reserva.

> Para la primera parte de la práctica, el cliente debe utilizar `192.168.1.1` como DNS. Configurar directamente `8.8.8.8` o `1.1.1.1` en el cliente evitaría el servicio BIND local y alteraría el escenario que se quiere analizar.

Si `/etc/resolv.conf` está gestionado por NetworkManager u otro servicio, la configuración se debe modificar mediante ese gestor, no sobrescribiendo manualmente el archivo.

![Configuración DHCP, puerta de enlace y DNS del cliente](<./image capture/cliente_configuracion_dhcp_dns.png>)

## 3. Pregunta 1. Análisis del DNS tradicional

### 3.1. Enunciado

Realizar consultas DNS a tres dominios de Internet poco habituales, capturar el tráfico con Wireshark y explicar:

- Los paquetes generados.
- Los protocolos y puertos utilizados.
- Si la información está cifrada.
- El recorrido de la consulta y de la respuesta.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/4b9d06d1-847e-4f9d-a1ea-1c63ed1392f4/ACT2-DNS-Wireshark-Tagged.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=2876e8ccf85e7dfe8ca05cdbbe8cd41afd30e7688af19284299c6ef60403e2c7)

### 3.2. Preparación de Wireshark

En Debian, la captura puede habilitarse para usuarios del grupo `wireshark` mediante la configuración del paquete:

```bash
sudo dpkg-reconfigure wireshark-common
sudo usermod -aG wireshark "$USER"
```

Después se cierra y se vuelve a iniciar la sesión del usuario para actualizar los grupos.

Comprobaciones:

```bash
id
getcap /usr/bin/dumpcap
```

Wireshark se ejecuta como usuario normal:

```bash
wireshark
```

La memoria describe este enfoque de separación entre la aplicación gráfica y el motor de captura `dumpcap`. No se debe presentar como una garantía de seguridad absoluta: el acceso a capturas de red sigue siendo un privilegio sensible.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

Se selecciona la interfaz de Debian conectada a la red interna. Su nombre debe comprobarse en el cliente y no deducirse del nombre de las interfaces del servidor.

Filtro de visualización inicial:

```wireshark
dns
```

Para incluir tráfico del puerto 53 aunque no se haya diseccionado como DNS:

```wireshark
udp.port == 53 || tcp.port == 53
```

![Wireshark capturando en la interfaz interna del cliente](<./image capture/wireshark_interfaz_filtro_dns.png>)

### 3.3. Control de la caché

Consultar un dominio poco habitual reduce la probabilidad de encontrarlo en la caché local, pero no garantiza que el reenviador público no lo tenga almacenado.

Hay que distinguir:

1. Caché de Firefox.
2. Caché del sistema, si existe un servicio de resolución con caché.
3. Caché de BIND.
4. Caché del reenviador público.

Si Debian utiliza `systemd-resolved`, su caché puede vaciarse con:

```bash
sudo resolvectl flush-caches
```

Este comando no es aplicable si dicho servicio no está instalado o no está en uso.

En Ubuntu Server:

```bash
sudo rndc flush
```

Para Firefox se puede utilizar la página:

```text
about:networking#dns
```

y su opción de limpieza de la caché DNS.

> Vaciar BIND no vacía los resolutores de Google. Incluso con una caché local vacía, el reenviador puede contestar desde su propia caché.

### 3.4. Dominios utilizados y trazabilidad

La parte textual de la memoria identifica `www.u-tokyo.ac.jp`, pero no permite recuperar con certeza los otros dos nombres ni los recuentos exactos de cada captura. Por este motivo, los campos pendientes deben completarse con los valores visibles en las evidencias, sin sustituirlos por resultados inventados.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

| Prueba | Nombre consultado | Tipo de registro | Servidor consultado |
|---|---|---|---|
| 1 | `www.u-tokyo.ac.jp` | `A` para una prueba controlada | `192.168.1.1` |
| 2 | `<DOMINIO_2_DE_LA_CAPTURA>` | Según la consulta realizada | `192.168.1.1` |
| 3 | `<DOMINIO_3_DE_LA_CAPTURA>` | Según la consulta realizada | `192.168.1.1` |

### 3.5. Primera consulta

Ejemplo de consulta controlada:

```bash
nslookup -type=A www.u-tokyo.ac.jp 192.168.1.1
```

Alternativa con `dig`:

```bash
dig @192.168.1.1 www.u-tokyo.ac.jp A
```

Para localizar los paquetes:

```wireshark
dns.qry.name == "www.u-tokyo.ac.jp"
```

En el paquete de consulta se revisan:

- Dirección IP de origen: la del cliente Debian.
- Dirección IP de destino: `192.168.1.1`.
- Puerto de origen: un puerto efímero del cliente.
- Puerto de destino: `53`.
- Nombre consultado.
- Tipo de registro.
- Identificador de transacción DNS.

En la respuesta se comprueba:

- Dirección de origen: el servidor DNS local.
- Puerto de origen: `53`.
- Puerto de destino: el puerto utilizado por el cliente.
- Coincidencia del identificador con la consulta.
- Código de respuesta.
- Registros devueltos y sus TTL.
- Posibles alias `CNAME`.

![Consulta DNS tradicional al primer dominio](<./image capture/dns_dominio_1_consulta_respuesta.png>)

#### Interpretación

Para una única transacción DNS mediante UDP, sin reintentos ni fragmentación, el intercambio básico en el enlace cliente-servidor consta de:

1. Un datagrama con la consulta.
2. Un datagrama con la respuesta.

Esto representa un caso básico de dos paquetes DNS observados en ese enlace, no un recuento demostrado de la captura ni una regla universal para cualquier comando o navegación.

La consulta y la respuesta no se cifran por utilizar el DNS tradicional: Wireshark puede mostrar directamente el nombre consultado y los registros devueltos.

### 3.6. Segunda consulta

Sustituir el marcador por el segundo nombre real de la práctica:

```bash
nslookup -type=A <DOMINIO_2_DE_LA_CAPTURA> 192.168.1.1
```

Filtro de visualización:

```wireshark
dns.qry.name == "<DOMINIO_2_DE_LA_CAPTURA>"
```

En esta captura se debe comprobar si:

- Existe una única consulta `A` o aparecen otras consultas.
- La respuesta contiene una o varias direcciones.
- Se devuelve algún alias `CNAME`.
- Se observa algún reintento.
- El código de respuesta indica éxito o un error.
- El tráfico utiliza UDP o se establece una conexión TCP.

![Consulta DNS tradicional al segundo dominio](<./image capture/dns_dominio_2_consulta_respuesta.png>)

> Una respuesta con varias direcciones IP no implica necesariamente varios paquetes: un mismo mensaje DNS puede contener varios registros de respuesta.

### 3.7. Tercera consulta

Sustituir el marcador por el tercer nombre real de la práctica:

```bash
nslookup -type=A <DOMINIO_3_DE_LA_CAPTURA> 192.168.1.1
```

Filtro de visualización:

```wireshark
dns.qry.name == "<DOMINIO_3_DE_LA_CAPTURA>"
```

Se repite el análisis de direcciones, puertos, identificador, tipo de consulta, código de respuesta y registros devueltos.

![Consulta DNS tradicional al tercer dominio](<./image capture/dns_dominio_3_consulta_respuesta.png>)

### 3.8. Recuento de paquetes

Para contar correctamente se deben definir el punto de captura y el alcance del recuento.

| Punto de captura | Tráfico que permite observar |
|---|---|
| Interfaz interna del cliente | Consultas del cliente y respuestas del servidor local. |
| `ens37` del servidor | Intercambio entre el cliente y BIND, además de otro tráfico de la red interna. |
| `ens33` del servidor | Consultas de BIND a los reenviadores y tráfico encaminado hacia Internet. |
| Ambas interfaces del servidor | Los dos tramos, que deben analizarse por separado. |

En un escenario simple sin respuesta en la caché de BIND:

```text
Cliente → BIND: consulta
BIND → reenviador: consulta
Reenviador → BIND: respuesta
BIND → cliente: respuesta
```

Este esquema representa cuatro mensajes distribuidos entre dos tramos. No significa que el cliente deba capturar cuatro paquetes DNS.

Si BIND responde desde su caché, no necesita generar una nueva consulta al reenviador para esa respuesta.

Tabla de resultados que debe completarse con las capturas:

| Dominio | Consultas DNS capturadas | Respuestas DNS capturadas | Transporte | Observaciones |
|---|---:|---:|---|---|
| `www.u-tokyo.ac.jp` | Pendiente de verificar | Pendiente de verificar | Según captura | Indicar `A`, `AAAA`, `CNAME`, reintentos y errores si aparecen. |
| `<DOMINIO_2_DE_LA_CAPTURA>` | Pendiente de verificar | Pendiente de verificar | Según captura | Completar con la evidencia. |
| `<DOMINIO_3_DE_LA_CAPTURA>` | Pendiente de verificar | Pendiente de verificar | Según captura | Completar con la evidencia. |

#### Por qué pueden aparecer más de dos paquetes

- Consultas de diferentes tipos, como `A` y `AAAA`.
- Consultas adicionales realizadas por el navegador.
- Reintentos por pérdida de paquetes o falta de respuesta.
- Uso de TCP.
- Fragmentación IP.
- Consultas de fondo ajenas a la prueba.

También hay que separar el tráfico DNS del resto:

- ARP resuelve direcciones de enlace en la red local.
- ICMP puede corresponder a `ping`.
- TCP y TLS pueden corresponder a la conexión web posterior.

> Un `ping` puede provocar una resolución DNS antes de enviar mensajes ICMP, pero los paquetes ICMP no forman parte de la consulta DNS.

### 3.9. Recorrido de una consulta DNS tradicional

```mermaid
sequenceDiagram
    participant C as Cliente Debian
    participant B as BIND 9 · 192.168.1.1
    participant R as Reenviador público

    C->>B: Consulta DNS · puerto destino 53
    alt Respuesta válida en caché
        B-->>C: Respuesta DNS desde caché
    else No hay respuesta en caché
        B->>R: Nueva consulta DNS · puerto destino 53
        R-->>B: Respuesta DNS
        B-->>C: Respuesta DNS
    end
```

BIND no se limita a encaminar sin cambios el paquete del cliente: procesa la consulta y genera su propio intercambio con el reenviador. Por ello, los puertos de origen y los identificadores de transacción no tienen por qué coincidir entre ambos tramos.

### 3.10. Valoración de seguridad

En el DNS tradicional, la captura permite inspeccionar los nombres consultados y las respuestas.

Esto facilita la administración y el diagnóstico, pero también expone información de resolución a quien pueda observar ese tráfico.

El nombre consultado no equivale a la URL completa: una consulta DNS no contiene por sí sola la ruta de una página, sus parámetros ni el contenido de la sesión web.

## 4. Pregunta 2. DNS sobre HTTPS en Firefox

### 4.1. Enunciado

Activar DNS sobre HTTPS en el navegador de la máquina cliente, mostrar dónde se configura, repetir las peticiones y explicar cómo se observa el tráfico con Wireshark.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/4b9d06d1-847e-4f9d-a1ea-1c63ed1392f4/ACT2-DNS-Wireshark-Tagged.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=2876e8ccf85e7dfe8ca05cdbbe8cd41afd30e7688af19284299c6ef60403e2c7)

### 4.2. Funcionamiento de DoH

DNS sobre HTTPS transporta las consultas y respuestas DNS dentro de conexiones HTTPS hacia un resolutor.

En este laboratorio, la memoria describe el uso de Firefox con Cloudflare. El navegador realiza la resolución mediante el proveedor DoH en lugar de enviar esas consultas al DNS local del sistema.  [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/160530867/8fb80786-40e8-4373-a70f-ad7d28026c3c/DNS-DHCP.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Checksum-Mode=ENABLED&X-Amz-Credential=ASIA2F3EMEYE7MMQOMAL%2F20261027%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20261027T125307Z&X-Amz-Expires=3311&X-Amz-Security-Token=IQoJb3JpZ2luX2VjECUaCXVzLWVhc3QtMSJHMEUCIF6U1R7XHteKxYVZKIUS6eJZzwiwgBmJsaLQadg2%2F%2FIfAiEAs9sg1XezLsZp6wq8CWi5sOs7uX8m3wGUeTnMlWNdVcMqiQUI7f%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARABGgw2OTk3NTMzMDk3MDUiDOLtOxFmNKBUpeRe0CrdBFVfy4s2AwZLyFxILOrYfWqUV6JzmzGXmI94s5E1%2Fvo%2FA394WlOAza7DF2e8WjJ9M%2B4T1SYhPyW4K%2BmtX1O8KJp5VO0hbi3%2Bi3IpZFDRu92yFwIBtorlusAKOhLZPJQ01EH2Fy55Fjwpyk4z6qaTy%2BEglEJAxbeCvFQrOnT3cXCaslT1E5DayQpCyYWegGWR2IHLec4FvkNLPBjiaAI5ShqlFZfM094Db14p55KJTisbMByenjmmzVNF4HRt9o8OQm7BEqpiqj9ORs5ob8%2B0ZQsbo8mK6Ucwa%2FlhgIIn1U1JcaRkJ2UvsPQpR%2B3z8a3pnMObqf9m2flbFhmrskd263wy8tvFNdebiaDGxlD%2Bk1ZQanEcgCUId6uJYEIIfZ6wn5qLvaB4nM6WLVKhHBzAYNfEQfyldCobePyhPUcQP%2F7oAsCsyhG3alYakBTXZEw%2FxNIireJlcVBZ%2FMq%2FJpCsAKLVs%2F3BcFnJCLMR59WY1eroiRSrVz0eH%2FMxllhA%2BDmUiY%2FPM67JUYWontJbneQKN46Bk59OLDF0GNSpjlNVWWmttpAclZmIkU0iM7BKHQcKm1KO9Mm1XLE4qp7CLjOQ4IrqhIuJyhj5amDPtt%2FjpYkC9e%2Fv0At%2F%2B7W2WiDao7wKVpW0%2Fs3iFHxoRzGr1hbL6gU6zvKSvdxl7CyB91iIiRPVHGiG9QrLNIlp%2B3nGKdHFHc%2FJ8Af2pjmEcmYJKCpoLZtnD3FcBgDMXFHz0dhokYPKrV0IhuzUBmspPJznkrnC7uBNU1na4%2B45TbmLR8JQaI%2BL8pWkv%2FOUAcRIvSwiMM%2B7gtcGOpgBC8Pa1ZPOup9jNlH4u2t0hqVamqCzV3sqDQgKAvyYeYAQAnrqhNHkUi4tPnKajrXEfkMDXHwns2QbQVdwvUjzOD51trNjJBS2gy2vuz5F%2BYM65%2FkS0FVLWUO3Aw%2F91JoGt4K0TQo7WAZQIdbCaUtLFH8S0ESpj7sAacV79JFBBfQPjECpjWVmE2K0QP4V%2F0sGviqdDW%2F8yC8%3D&X-Amz-SignedHeaders=host&x-id=GetObject&X-Amz-Signature=cce26cc31d60d660dd79695cdc0d00fc804c46dd3d85de6f972acd48ec4e3a6a)

Esto modifica la función del servidor Ubuntu:

- Continúa actuando como puerta de enlace.
- Continúa aplicando NAT al tráfico del cliente.
- BIND deja de intervenir en las consultas que Firefox envía efectivamente por DoH.
- Otras aplicaciones pueden seguir utilizando BIND y el puerto 53.

> Activar DoH en Firefox no cambia automáticamente el funcionamiento de `nslookup`, `dig` o el resto de las aplicaciones del sistema.

### 4.3. Activación en Firefox

Ruta de configuración:

```text
Menú de Firefox
  → Ajustes
  → Privacidad y seguridad
  → DNS sobre HTTPS
```

La ubicación y los nombres concretos pueden variar entre versiones. La documentación de Mozilla sitúa la selección del nivel de protección en esta sección.  [support.mozilla](https://support.mozilla.org/en-US/kb/dns-over-https)

Para una prueba controlada:

1. Seleccionar el nivel de protección máxima.
2. Elegir Cloudflare como proveedor.
3. Comprobar que no se han añadido excepciones para los dominios de la práctica.

La protección máxima utiliza DNS seguro y muestra un aviso si no puede acceder al resolutor seguro. Otros niveles pueden permitir recurrir al DNS del sistema, por lo que no basta con activar una opción sin revisar su comportamiento.  [support.mozilla](https://support.mozilla.org/en-US/kb/dns-over-https)

![Activación de DNS sobre HTTPS en Mozilla Firefox](<./image capture/firefox_activacion_doh.png>)

### 4.4. Repetición de las consultas

Las pruebas deben realizarse desde Firefox, no únicamente repitiendo `nslookup`.

Procedimiento:

1. Vaciar la caché DNS de Firefox.
2. Iniciar una nueva captura en Wireshark.
3. Abrir el primer dominio.
4. Identificar el tráfico hacia el proveedor DoH.
5. Repetir el procedimiento con el segundo y el tercer dominio.
6. Anotar los nombres y tiempos de cada prueba.

El navegador puede generar consultas adicionales para recursos de terceros y servicios propios. Para reducir ruido, conviene cerrar otras pestañas y evitar navegación simultánea.

![Prueba de navegación con DNS sobre HTTPS](<./image capture/firefox_prueba_dominios_doh.png>)

### 4.5. Qué se observa con el filtro DNS

Filtro:

```wireshark
dns
```

Las consultas transportadas dentro de HTTPS no se muestran como DNS legible en una captura cifrada sin las claves de sesión.

Sin embargo, pueden seguir apareciendo paquetes DNS tradicionales por diferentes motivos:

- Otras aplicaciones utilizan el DNS del sistema.
- Se realizan consultas auxiliares.
- Se resuelve inicialmente el nombre del proveedor DoH.
- Hay excepciones o mecanismos de retorno al DNS del sistema.

Por ello, la ausencia de paquetes en el filtro `dns` no demuestra por sí sola que DoH funcione; también podría deberse a una respuesta desde caché o a la captura de una interfaz equivocada.

![Filtro DNS después de activar DoH](<./image capture/wireshark_filtro_dns_con_doh.png>)

### 4.6. Localización del tráfico cifrado

Para observar el tráfico HTTPS:

```wireshark
tcp.port == 443 || udp.port == 443
```

Para localizar protocolos cifrados:

```wireshark
tls || quic
```

Si se ha identificado la dirección IP del proveedor utilizada en la sesión:

```wireshark
ip.addr == <IP_DEL_PROVEEDOR_DOH> && (tcp.port == 443 || udp.port == 443)
```

El marcador debe sustituirse por la dirección realmente observada, no por una dirección supuesta.

Con HTTP/2 sobre TLS se espera tráfico TCP hacia el puerto 443. Si se utiliza HTTP/3, puede observarse QUIC sobre UDP hacia el puerto 443.

En una captura cifrada pueden analizarse:

- Direcciones IP.
- Puertos.
- Tamaños y tiempos.
- Establecimiento de la conexión.
- Intercambios TLS o QUIC identificables.
- Datos de aplicación cifrados.

No se puede leer directamente el nombre incluido en la consulta DoH solo por visualizar los paquetes TLS.

> El puerto 443 no identifica por sí mismo una consulta DoH: también se utiliza para la navegación HTTPS ordinaria.

![Tráfico TLS asociado a la prueba de DoH](<./image capture/wireshark_trafico_tls_doh.png>)

### 4.7. Visualizar el tráfico cifrado y descifrar su contenido son tareas distintas

Hay dos niveles de análisis:

| Nivel | Requisitos | Información disponible |
|---|---|---|
| Observación de tráfico cifrado | Captura en la interfaz adecuada | IP, puertos, tiempos, tamaños y protocolos visibles. |
| Inspección del contenido DoH | Captura y secretos de sesión del navegador | Solicitudes HTTP y mensajes DNS, si Wireshark puede reconstruirlos y diseccionarlos. |

Para el segundo nivel, Wireshark admite el uso de un archivo de secretos TLS generado por el navegador mediante `SSLKEYLOGFILE`.  [wiki.wireshark](https://wiki.wireshark.org/TLS)

### 4.8. Procedimiento adicional de descifrado en el laboratorio

> Este procedimiento es una ampliación reproducible. No debe presentarse como una operación ya realizada si no existe una captura que la documente.

#### Paso 1. Cerrar Firefox completamente

Es necesario cerrar todas las instancias para que el proceso que realiza la prueba herede la variable de entorno. Este requisito forma parte del procedimiento documentado por Wireshark.  [wiki.wireshark](https://wiki.wireshark.org/TLS)

#### Paso 2. Iniciar Firefox con registro de secretos TLS

En una terminal del cliente Debian:

```bash
umask 077
export SSLKEYLOGFILE="$HOME/sslkeys-doh.log"
firefox
```

En instalaciones donde el ejecutable se denomine `firefox-esr`, debe utilizarse ese nombre.

Comprobar que se ha creado el archivo y que contiene datos:

```bash
ls -l "$HOME/sslkeys-doh.log"
test -s "$HOME/sslkeys-doh.log" && echo "Archivo de secretos generado"
```

#### Paso 3. Configurar Wireshark

Acceder a:

```text
Editar
  → Preferencias
  → Protocolos
  → TLS
  → (Pre)-Master-Secret log filename
```

Seleccionar el archivo `sslkeys-doh.log` generado en el directorio personal.

Este archivo contiene secretos de las sesiones, no la clave privada del servidor remoto. Wireshark utiliza los secretos correspondientes para descifrar el tráfico capturado.  [wiki.wireshark](https://wiki.wireshark.org/TLS)

![Configuración del archivo de secretos TLS en Wireshark](<./image capture/wireshark_configuracion_sslkeylogfile.png>)

#### Paso 4. Capturar y repetir las pruebas

1. Iniciar la captura antes de generar las conexiones de prueba.
2. Vaciar la caché DNS del navegador.
3. Visitar los tres dominios.
4. Detener la captura.
5. Revisar los paquetes reconstruidos por Wireshark.

Si la sesión utiliza HTTP/2, un filtro útil es:

```wireshark
http2
```

También puede utilizarse:

```wireshark
dns || http2
```

La visualización depende de la versión de Wireshark, del transporte empleado y de que la captura y los secretos correspondan a la misma sesión.

#### Paso 5. Identificar el intercambio DoH

Después del descifrado se buscan elementos como:

- Solicitudes al servicio de resolución del proveedor.
- Una ruta como `/dns-query`.
- Métodos HTTP `GET` o `POST`.
- El tipo de contenido `application/dns-message`, cuando esté presente.
- El mensaje DNS encapsulado, si el analizador lo interpreta.

No se deben atribuir valores concretos de cabeceras ni nombres consult
