# cloudServer

Opis dzialania docker-compose.yml

## docker-compose.yml

Plik `docker-compose.yml` opisuje konfigurację usług uruchamianych w Dockerze oraz sposób ich działania.

### services
Sekcja `services` zawiera listę usług uruchamianych w kontenerach. W projekcie wykorzystano:
- **filebrowser** – aplikację umożliwiającą graficzne zarządzanie plikami na serwerze,
- **portainer** – narzędzie webowe do zarządzania kontenerami Docker.

### image
Parametr `image` określa obraz Dockera, z którego tworzony jest kontener. Obrazy pobierane są z publicznego repozytorium Docker Hub.

### container_name
`container_name` definiuje nazwę kontenera w systemie, co ułatwia jego identyfikację i zarządzanie.

### ports
Sekcja `ports` mapuje porty hosta na porty kontenera, dzięki czemu usługi są dostępne z sieci lokalnej.

### volumes
Sekcja `volumes` odpowiada za trwałość danych i mapowanie katalogów z systemu hosta do kontenerów.

#### Filebrowser – wolumeny
- `./data:/srv`  
  Katalog `data` na hoście przechowuje pliki użytkowników. W kontenerze są one dostępne w katalogu `/srv`, z którego korzysta aplikacja Filebrowser. Dzięki temu pliki pozostają na serwerze nawet po restarcie lub usunięciu kontenera.

- `./db/filebrowser.db:/database.db`  
  Plik bazy danych aplikacji Filebrowser jest przechowywany na hoście. Zapewnia to zachowanie użytkowników, haseł i konfiguracji aplikacji po ponownym uruchomieniu kontenera.

- `./config/settings.json:/config/settings.json`  
  Plik konfiguracyjny aplikacji jest przechowywany na hoście, co umożliwia jego łatwą edycję bez konieczności przebudowy obrazu kontenera.

#### Portainer – wolumeny
- `./portainer_data:/data`  
  Wolumen przechowuje dane konfiguracyjne Portainera, takie jak konta użytkowników i ustawienia aplikacji. Dzięki temu konfiguracja Portainera jest trwała i nie zostaje utracona po restarcie kontenera.

### restart
Opcja `restart` określa politykę ponownego uruchamiania kontenerów. Zapewnia automatyczne wznawianie usług po restarcie serwera lub usługi Docker.
