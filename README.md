# Projekt: Serwer plików z automatycznym backupem i monitoringiem

## 1. Wprowadzenie

Celem projektu jest stworzenie serwera plików umożliwiającego bezpieczne przechowywanie danych, automatyczne wykonywanie kopii zapasowych oraz monitorowanie zasobów systemowych wraz z powiadomieniami o ich stanie. Projekt realizowany jest w środowisku wirtualnym i opiera się na konteneryzacji usług przy użyciu Dockera.

Projekt jest w trakcie realizacji – część funkcjonalności została już wdrożona, a pozostałe są zaplanowane do implementacji w kolejnych etapach.

---

## 2. Architektura systemu


![Schemat Architektury Systemu](diagramProject.drawio%20(1).png)


### 2.1 Warstwa fizyczna i wirtualizacja

* Komputer fizyczny (PC) pełni rolę hosta.
* Na hoście uruchomiono środowisko wirtualizacji **VirtualBox**.
* W VirtualBox działa maszyna wirtualna z systemem **Ubuntu Server 24.04 LTS**.
* Karta sieciowa maszyny wirtualnej skonfigurowana jest w trybie **mostkowanym (Bridged)**, dzięki czemu serwer jest widoczny w sieci lokalnej (LAN) jako niezależne urządzenie.

### 2.2 System operacyjny

* Ubuntu Server 24.04 LTS
* System pełni rolę hosta dla kontenerów Dockera oraz środowiska do uruchamiania skryptów automatyzujących.

---

## 3. Zdalny dostęp

* Zainstalowano i skonfigurowano **OpenSSH Server**.
* Dostęp do serwera możliwy jest:

  * z komputera (SSH),
  * z telefonu przy użyciu aplikacji **Termius**.
* Pozwala to na pełne zarządzanie serwerem bez fizycznego dostępu do maszyny.

---

## 4. Konteneryzacja

### 4.1 Docker Engine

Docker Engine jest podstawowym komponentem odpowiedzialnym za:

* uruchamianie i zatrzymywanie kontenerów,
* zarządzanie obrazami,
* obsługę sieci i wolumenów.

Docker Engine wykorzystuje zasoby systemu hosta – kontenery nie posiadają własnej pamięci ani dysku.

### 4.2 Docker Compose

Docker Compose jest narzędziem wyższego poziomu służącym do definiowania i uruchamiania wielu usług jednocześnie przy użyciu pliku `docker-compose.yml`.

Docker Compose:

* opisuje usługi, sieci i wolumeny,
* automatycznie tworzy wymagane zasoby,
* uruchamia całą konfigurację jedną komendą.

Docker Compose nie zastępuje Docker Engine – działa jako warstwa konfiguracyjna.

---

## 5. Usługi uruchomione w Dockerze

### 5.1 FileBrowser

FileBrowser to usługa zapewniająca graficzny interfejs webowy (Web GUI) do zarządzania plikami na serwerze.

Funkcje:

* przesyłanie plików z komputera do serwera,
* zarządzanie katalogami i plikami,
* dostęp przez przeglądarkę w sieci LAN.

Dane FileBrowsera zapisywane są przy użyciu wolumenów Dockera, które mapują katalog systemowy do wnętrza kontenera. Zapewnia to trwałość danych po restarcie kontenera lub serwera.

### 5.2 Portainer

Portainer to narzędzie umożliwiające graficzne zarządzanie kontenerami Dockera.

Umożliwia:

* podgląd uruchomionych kontenerów,
* zarządzanie obrazami i wolumenami,
* start i stop usług bez użycia linii poleceń.

Wybrane parametry:

* `restart: always` – kontener uruchamia się automatycznie po restarcie Dockera lub systemu.
* `security_opt: no-new-privileges:true` – zwiększa bezpieczeństwo poprzez blokadę eskalacji uprawnień.

---

## 6. Zarządzanie danymi i wolumeny

Dane przechowywane są na hoście systemu Linux, a nie wewnątrz kontenerów.

Przykładowe mapowanie wolumenu:

* katalog hosta: `/opt/cloud/data`
* katalog w kontenerze: `/data`

Takie rozwiązanie:

* zapewnia trwałość danych,
* umożliwia wykonywanie backupów na poziomie systemu,
* ułatwia migrację danych.

---

## 7. Monitoring systemu

### 7.1 Node Exporter

Node Exporter zbiera dane o stanie systemu, m.in.:

* użycie CPU,
* pamięć RAM,
* przestrzeń dyskowa.

Metryki udostępniane są pod adresem:

```
http://localhost:9100/metrics
```

### 7.2 Prometheus

Prometheus pobiera dane z Node Exportera i zapisuje je w swojej bazie danych.

Dostęp do interfejsu Prometheusa:

```
http://localhost:9090
```

### 7.3 Grafana

Grafana służy do wizualizacji danych dostarczanych przez Prometheusa.

Umożliwia:

* tworzenie dashboardów,
* analizę zużycia zasobów w czasie rzeczywistym.

Interfejs webowy:

```
http://localhost:3000
```

---

## 8. Automatyzacja

### 8.1 Lokalizacja skryptów

* Skrypty systemowe: `/usr/local/bin`
* Skrypty i konfiguracje usług dodatkowych: `/opt`

### 8.2 Planowane skrypty

* **Bash**

  * automatyczny backup danych (raz dziennie),
  * archiwizacja plików.

* **Python**

  * monitorowanie ilości wolnego miejsca na dysku,
  * wysyłanie powiadomień na telefon,
  * wysyłanie backupów do chmury.

---

## 9. Backup danych

Backupy danych:

* wykonywane są na poziomie systemu hosta,
* nie są przechowywane wyłącznie w kontenerach,
* w kolejnych etapach projektu będą wysyłane do chmury.

Takie podejście zwiększa bezpieczeństwo i niezależność danych od środowiska kontenerowego.

---

## 10. Stan projektu

### Zrealizowane elementy

* instalacja Ubuntu Server 24.04 LTS,
* konfiguracja VirtualBox i sieci mostkowanej,
* instalacja i konfiguracja OpenSSH,
* dostęp z komputera i telefonu,
* instalacja Dockera i Docker Compose,
* uruchomienie FileBrowsera,
* instalacja Portainera,
* konfiguracja monitoringu: Node Exporter, Prometheus, Grafana.

### Elementy planowane

* skrypty backupowe w Bashu,
* skrypty powiadomień w Pythonie,
* backup do chmury,

---

## 11. Podsumowanie

Projekt stanowi kompletną bazę pod domowy lub laboratoryjny serwer plików z funkcjami monitoringu i automatyzacji. Zastosowanie wirtualizacji i konteneryzacji pozwala na łatwą rozbudowę systemu oraz bezpieczne zarządzanie danymi. Kolejne etapy projektu skupią się na automatyzacji backupów i integracji z chmurą.
