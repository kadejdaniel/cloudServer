# Serwer plików z monitoringiem

## 1. Wprowadzenie

Celem projektu jest stworzenie prywatnego serwera plików umożliwiającego bezpieczne przechowywanie danych oraz monitorowanie zasobów systemowych. System działa w środowisku wirtualnym i oparty jest na konteneryzacji usług przy użyciu Dockera.

Projekt stanowi w pełni działające środowisko serwerowe dostępne w sieci lokalnej.

---

## 2. Architektura systemu

![Schemat Architektury Systemu](diagram.png)

Poniżej przedstawiona jest rzeczywista struktura katalogów projektu na serwerze:

```
private-cloud/
│
├── docker-compose.yml
├── .gitignore
├── README.md
│
├── filebrowser/
│   ├── data/                # pliki użytkowników
│   ├── db/
│   │   └── filebrowser.db   # baza danych FileBrowser
│   └── config/
│       └── settings.json    # konfiguracja FileBrowser
│
├── portainer/
│   └── data/                # dane i konfiguracja Portainera
│
└── monitoring/
    ├── prometheus/
    │   └── prometheus.yml   # konfiguracja Prometheusa
    └── grafana/             # dane Grafany (dashboardy, alerty)
```


### 2.1 Warstwa fizyczna i wirtualizacja

- Komputer fizyczny (PC) pełni rolę hosta.
- Na hoście uruchomiono środowisko wirtualizacji **VirtualBox**.
- W VirtualBox działa maszyna wirtualna z systemem **Ubuntu Server 24.04 LTS**.
- Karta sieciowa maszyny wirtualnej skonfigurowana jest w trybie **mostkowanym (Bridged)**, dzięki czemu serwer jest widoczny w sieci lokalnej jako niezależne urządzenie.

### 2.2 System operacyjny

- Ubuntu Server 24.04 LTS  
- System pełni rolę hosta dla kontenerów Dockera oraz środowiska do zarządzania usługami.

---

## 3. Zdalny dostęp

- Zainstalowano i skonfigurowano **OpenSSH Server**.
- Dostęp do serwera możliwy jest:
  - z komputera (SSH),
  - z telefonu przy użyciu aplikacji **Termius**.
- Umożliwia to pełne zarządzanie serwerem bez fizycznego dostępu do maszyny.

---

## 4. Konteneryzacja

### 4.1 Docker Engine

Docker Engine odpowiada za:

- uruchamianie i zatrzymywanie kontenerów,
- zarządzanie obrazami,
- obsługę sieci i wolumenów.

Kontenery wykorzystują zasoby systemu hosta i nie posiadają własnej niezależnej pamięci ani dysku.

### 4.2 Docker Compose

Docker Compose umożliwia definiowanie i uruchamianie wielu usług jednocześnie przy użyciu pliku `docker-compose.yml`.

Projekt oparty jest o Docker Compose, który uruchamia wszystkie usługi jedną komendą.

---

# Konfiguracja Docker Compose

Poniżej znajduje się pełna konfiguracja usług uruchamianych w projekcie.

````yaml
services:

  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    user: "0:0"
    ports:
      - "8080:80"
    volumes:
      - ./filebrowser/data:/srv
      - ./filebrowser/db/filebrowser.db:/database.db
      - ./filebrowser/config/settings.json:/config/settings.json
    command:
      [
        "--address", "0.0.0.0",
        "--port", "80",
        "--database", "/database.db"
      ]
    restart: unless-stopped


  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    security_opt:
      - no-new-privileges:true
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./portainer/data:/data
    ports:
      - "9443:9443"


  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - "--path.procfs=/host/proc"
      - "--path.sysfs=/host/sys"
      - "--path.rootfs=/rootfs"


  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter


  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - GF_SMTP_ENABLED=true
      - GF_SMTP_HOST=smtp.gmail.com:587
      - GF_SMTP_USER=twoj_email@gmail.com
      - GF_SMTP_PASSWORD=haslo_aplikacji
      - GF_SMTP_FROM_ADDRESS=twoj_email@gmail.com
      - GF_SMTP_FROM_NAME=Cloud Server
    volumes:
      - ./monitoring/grafana:/var/lib/grafana

````

## Jak uruchomić 

W katalogu projektu:

```bash
docker compose up -d
```

Sprawdzenie statusu kontenerów:

```bash
docker ps
```

Zatrzymanie projektu:

```bash
docker compose down
```

---

## Dostęp do usług

| Usługa        | Adres                         |
|--------------|--------------------------------|
| FileBrowser  | http://IP_SERWERA:8080         |
| Portainer    | https://IP_SERWERA:9443        |
| Prometheus   | http://IP_SERWERA:9090         |
| Grafana      | http://IP_SERWERA:3000         |
| Node Exporter| http://IP_SERWERA:9100/metrics |


## 5. Usługi uruchomione w Dockerze

### 5.1 FileBrowser

FileBrowser zapewnia graficzny interfejs webowy do zarządzania plikami na serwerze.

Funkcje:

- przesyłanie plików z komputera do serwera,
- zarządzanie katalogami i plikami,
- dostęp przez przeglądarkę w sieci LAN.

Dane zapisywane są w wolumenach Dockera, co zapewnia ich trwałość po restarcie kontenera lub systemu.

---

### 5.2 Portainer

Portainer umożliwia graficzne zarządzanie kontenerami Dockera.

Umożliwia:

- podgląd uruchomionych kontenerów,
- zarządzanie obrazami i wolumenami,
- start i stop usług bez użycia linii poleceń.

Wybrane parametry konfiguracji:

- `restart: always` – automatyczny restart po restarcie systemu,
- `security_opt: no-new-privileges:true` – blokada eskalacji uprawnień.

---

## 6. Monitoring systemu

### 6.1 Node Exporter

Node Exporter zbiera dane o stanie systemu, m.in.:

- użycie CPU,
- pamięć RAM,
- przestrzeń dyskową.

Metryki dostępne są pod adresem:

```
http://localhost:9100/metrics
```

---

### 6.2 Prometheus

Prometheus pobiera dane z Node Exportera i zapisuje je w swojej bazie danych.

Interfejs webowy:



```
http://localhost:9090
```

---

### 6.3 Grafana

Grafana służy do wizualizacji danych dostarczanych przez Prometheusa.

Umożliwia:

- tworzenie dashboardów,
- analizę zużycia zasobów w czasie rzeczywistym.

Interfejs webowy:



```
http://localhost:3000
```

---


---

## 7. Stan projektu

### Zrealizowane elementy

- instalacja Ubuntu Server 24.04 LTS,
- konfiguracja VirtualBox i sieci mostkowanej,
- instalacja i konfiguracja OpenSSH,
- dostęp z komputera i telefonu,
- instalacja Dockera i Docker Compose,
- uruchomienie FileBrowsera,
- instalacja Portainera,
- konfiguracja monitoringu: Node Exporter, Prometheus, Grafana.

---

## 8. Podsumowanie

Projekt stanowi kompletną bazę pod prywatny serwer plików działający w sieci lokalnej z funkcją monitoringu zasobów systemowych. Zastosowanie wirtualizacji oraz konteneryzacji pozwala na łatwe zarządzanie środowiskiem oraz jego dalszą rozbudowę.

