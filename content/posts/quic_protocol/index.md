+++
title = "QUIC: Транспортный протокол, который переосмыслил интернет"
draft = false
date = "2026-03-31"
[taxonomies]
categories = ["net"]
tags = ["quic"]
+++

# QUIC: Транспортный протокол, который переосмыслил интернет

> Эта статья — о протоколе QUIC: откуда он появился, как работает и почему он стал основой HTTP/3.
> В конце — пример возможной архитектуры клиент-серверного приложения поверх QUIC.

---

## Предыстория: что было не так с TCP

Интернет десятилетиями стоял на двух столпах: IP для адресации и **TCP** для надёжной доставки данных.
TCP — рабочая лошадка. Он гарантирует порядок доставки пакетов, следит за потерями и переотправляет
их при необходимости. Но у него есть три системных проблемы, которые становятся всё заметнее
по мере роста мобильных сетей и требований к задержкам.

### 1. Head-of-line blocking

TCP доставляет данные строго по порядку. Если один пакет потерялся — всё, что пришло после него,
ждёт в очереди, даже если это совершенно независимые данные. При загрузке страницы это означает,
что потеря одного пакета CSS блокирует отрисовку изображений, которые уже пришли.

HTTP/2 добавил мультиплексирование потоков поверх одного TCP-соединения — но
проблему не решил: TCP всё равно держит единую очередь.

### 2. Долгое установление соединения

Стандартное соединение TCP + TLS 1.2 требует **3 round-trip'а** до первого байта данных:
TCP-handshake (1-RTT), TLS-handshake (2 RTT). Для мобильных сетей с задержкой 50–150 мс
это 300–450 мс только на установление соединения.

### 3. Привязанность к IP-адресу

TCP идентифицирует соединение по четырём параметрам: IP-адрес клиента, порт клиента,
IP-адрес сервера, порт сервера. Стоит телефону переключиться с Wi-Fi на мобильную сеть —
и IP-адрес меняется. TCP разрывает соединение, и всё начинается заново.

---

## Как появился QUIC

В **2012 году** инженер Google **Джим Роскинд** (Jim Roskind) предложил и реализовал
новый транспортный протокол, который должен был решить все три проблемы сразу.
Его назвали QUIC — изначально как аббревиатуру "Quick UDP Internet Connections".

Google развернул QUIC в браузере Chrome и на своих серверах для внутренних нужд.
К 2013 году протокол был публично анонсирован, а в 2015-м Google подал Internet Draft
в **IETF** (Internet Engineering Task Force) для стандартизации.

Стандартизация шла почти шесть лет: 26 очных заседаний рабочей группы, 1749 открытых задач,
тысячи писем. В **мае 2021 года** IETF опубликовал финальную спецификацию — **RFC 9000**.

При этом IETF QUIC заметно отличается от оригинального Google QUIC (gQUIC):
рабочая группа переработала его с нуля, сохранив идеи, но изменив детали реализации.
Версия gQUIC была признана устаревшей.

---

## Как работает QUIC

### Поверх UDP, а не TCP

QUIC работает поверх **UDP** — протокола без гарантий, без порядка, без рукопожатия.
Это кажется шагом назад, но на самом деле это ключевое решение:
QUIC сам управляет надёжностью, порядком и потоком данных —
но делает это умнее, чем TCP, потому что работает на уровне приложения,
а не в ядре операционной системы.

{% mermaid() %}
graph TD
    A["Приложение"]
    B["HTTP/3 (или другой protocol)"]
    C["QUIC (streams + crypto)"]
    D["UDP"]
    E["IP"]

    A --> B --> C --> D --> E
{% end %}


### TLS 1.3 как часть протокола

QUIC не добавляет TLS поверх транспорта — он **встроил TLS 1.3 внутрь себя**.
Криптографический и транспортный handshake объединены в один.

Результат:
- первое соединение — **1-RTT** до первого байта приложения;
- повторное соединение — **0-RTT**: клиент отправляет данные уже в первом пакете.

Для сравнения: TCP + TLS 1.2 требует 2–3 RTT.

### Независимые потоки (streams)

QUIC поддерживает несколько независимых потоков внутри одного соединения.
Если пакет одного потока потерялся — остальные потоки продолжают работать.
Это решает head-of-line blocking в корне: нет единой очереди, нет блокировки.

### Connection Migration

QUIC идентифицирует соединения не по IP-адресу, а по **Connection ID** —
уникальному идентификатору, который сохраняется при смене сети.

Когда смартфон переключается с Wi-Fi на мобильную сеть:
- TCP: соединение разрывается, приложение начинает заново;
- QUIC: соединение **мигрирует** на новый путь прозрачно для приложения.

---

## Где применяется QUIC

### HTTP/3

Главное применение QUIC — это **HTTP/3**.
В 2018 году IETF официально решил, что HTTP поверх QUIC будет называться HTTP/3.
По данным w3techs, на март 2026 года HTTP/3 поддерживают **38.7%** всех веб-сайтов.

Все основные браузеры (Chrome, Firefox, Edge, Safari) поддерживают HTTP/3 по умолчанию.
В Google Chrome QUIC используется **более чем в половине** всех соединений с серверами Google.

### Streaming и gaming

Видеоплатформы, онлайн-игры и VoIP-приложения выигрывают от независимых потоков и
0-RTT reconnect. Потеря пакета видео-потока не блокирует аудио-поток.

### IoT и мобильные устройства

Для устройств с нестабильным соединением Connection Migration — критически важная функция.
Датчики и смартфоны могут переключаться между сетями без потери сессии.

### CDN и облачные платформы

Cloudflare, Fastly, AWS (ноябрь 2025: поддержка QUIC в NLB) и другие платформы
уже поддерживают HTTP/3 и QUIC для своих клиентов.
QUIC снижает задержку на 25–30% по сравнению с TCP на мобильных сетях.

### DNS-over-QUIC

RFC 9250 описывает передачу DNS-запросов поверх QUIC вместо UDP/TCP.
Это даёт шифрование и быстрое восстановление соединения без накладных расходов.

---

## QUIC vs TCP: ключевые отличия

| Параметр | TCP | QUIC |
|---|---|---|
| Транспортный слой | Отдельно от TLS | UDP + встроенный TLS 1.3 |
| Установление соединения | 2–3 RTT | 1-RTT / 0-RTT |
| Head-of-line blocking | Есть | Нет |
| Connection Migration | Не поддерживается | Поддерживается |
| Мультиплексирование | Одна очередь | Независимые потоки |
| Управление перегрузкой | В ядре ОС | В user space |
| Обновляемость | Сложно (в ядре) | Легко (в приложении) |

---

## Пример архитектуры приложения поверх QUIC

Ниже — пример архитектуры защищённого клиент-серверного приложения,
где QUIC используется как транспортный слой.

Система состоит из нескольких компонентов:
- **Management** — центральный сервис для управления пользователями, серверами и ключами доступа;
- **Bot** — интерфейс управления через Telegram;
- **Node / Exit** — выходной узел на зарубежном сервере, реализующий QUIC ↔ TCP прокси;
- **Node / Relay** — промежуточный узел, форвардящий QUIC-потоки на exit без обработки;
- **Android Client** — мобильный клиент, импортирующий конфигурационный файл и поднимающий туннель через Android VpnService.

### Deployment diagram

{% mermaid() %}
graph TD
    Admin["👤 Admin"]
    TgUser["📱 User"]
    Bot["Bot\nTelegram Long Polling"]
    Mgmt["Management\nSpring Boot + SQLite"]
    DB["SQLite DB"]
    RuVPS["RU VPS\nnode · RELAY\nQUIC :443/udp"]
    GeVPS1["GE VPS #1\nnode · EXIT\nQUIC :443/udp"]
    GeVPS2["GE VPS #2\nnode · EXIT\nQUIC :443/udp"]
    AndroidClient["Android Client\nVpnService"]
    WinClient["Windows Client"]

    Admin -->|Telegram commands| Bot
    TgUser -->|receives .quic file| Bot
    Bot -->|REST API| Mgmt
    Mgmt --> DB
    Mgmt -->|node-agent HTTP| RuVPS
    Mgmt -->|node-agent HTTP| GeVPS1
    Mgmt -->|node-agent HTTP| GeVPS2
    RuVPS -->|QUIC relay stream| GeVPS1
    RuVPS -->|QUIC relay stream| GeVPS2
    AndroidClient -->|QUIC UDP 443| RuVPS
    WinClient -->|QUIC UDP 443| GeVPS1
{% end %}

### Logical components

{% mermaid() %}
graph LR
    subgraph Management
        UC[User Controller]
        SC[Server Controller]
        KC[Key Controller]
        KS[Key Service]
        NAC[Node Agent Client]
        DB2[(SQLite + Flyway)]
    end

    subgraph Bot
        AB[McsAdminBot]
        MC[Management Client]
    end

    subgraph Node
        NA[Node Agent Controller]
        QS[QUIC Server]
        EC[Exit Connection]
        RC[Relay Connection]
        TV[Token Validator]
    end

    subgraph AndroidClient["Android Client"]
        PR[ProfileRepository]
        CP[ConfigParser]
        TS[TunnelService]
        QT[QuicTransport]
        DC[DirectConnector]
        REL[RelayConnector]
    end

    AB --> MC --> UC
    MC --> SC
    MC --> KC
    KC --> KS --> NAC --> NA
    QS --> EC --> TV
    QS --> RC
    TS --> QT
    QT --> DC
    QT --> REL
{% end %}

### Поток установления соединения (direct профиль)

{% mermaid() %}
sequenceDiagram
    participant C as Android Client
    participant E as EXIT Node
    participant I as Internet

    C->>C: import .quic file
    C->>C: VpnService.Builder.establish()
    C->>E: QUIC handshake (TLS 1.3, 1-RTT)
    E-->>C: handshake complete
    C->>E: QUIC stream: AUTH frame (userId + token)
    E->>E: validate HMAC token
    E-->>C: AUTH OK
    C->>E: IP packets via QUIC stream
    E->>I: TCP to destination
    I-->>E: TCP response
    E-->>C: IP packets via QUIC stream
{% end %}

### Поток установления соединения (relay профиль)

{% mermaid() %}
sequenceDiagram
    participant C as Android Client
    participant R as RELAY Node
    participant E as EXIT Node
    participant I as Internet

    C->>R: QUIC handshake (1-RTT)
    R-->>C: handshake complete
    C->>R: connect stream to EXIT
    R->>E: QUIC relay stream
    E-->>R: relay accepted
    C->>R: IP packets
    R->>E: forwarded IP packets
    E->>I: TCP to destination
    I-->>E: TCP response
    E-->>R: forwarded response
    R-->>C: IP packets
{% end %}

---

## Почему QUIC важен для разработчиков

QUIC — это не просто "быстрый TCP". Это переосмысление того,
как должна работать надёжная сетевая коммуникация:

- **Независимость от ядра** — алгоритмы управления перегрузкой живут в user space
  и обновляются без изменений ОС;
- **Независимые потоки** — потеря одного пакета не останавливает остальные;
- **Встроенная криптография** — нет отдельного TLS, нет лишних round-trip'ов;
- **Connection Migration** — соединение живёт дольше сессии в одной сети.

Если вы проектируете мобильное приложение, IoT-систему или любой сервис
с нестабильным сетевым окружением — QUIC является разумным выбором
как транспортный слой уже сегодня.

---

*Стандарты: RFC 9000, RFC 8999, RFC 9001, RFC 9002, RFC 9114 (HTTP/3), RFC 9250 (DoQ)*