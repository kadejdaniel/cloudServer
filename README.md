# Serwer plików z monitoringiem

## 1. Wprowadzenie

Celem projektu jest stworzenie prywatnego serwera plików umożliwiającego bezpieczne przechowywanie danych oraz monitorowanie zasobów systemowych. System działa w środowisku wirtualnym i oparty jest na konteneryzacji usług przy użyciu Dockera.

Projekt stanowi w pełni działające środowisko serwerowe dostępne w sieci lokalnej.

---

## 2. Architektura systemu

![Schemat Architektury Systemu](diagram.png)

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

Docker Compose:

- opisuje usługi, sieci i wolumeny,
- automatycznie tworzy wymagane zasoby,
- uruchamia całą konfigurację jedną komendą.

---

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

