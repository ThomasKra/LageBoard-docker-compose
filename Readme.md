# LageBoard Docker-Compose

Dieses Repository beinhaltet die notwendigen Dateien, um LageBoard mit Docker zu betreiben.

## Voraussetzungen

Stelle sicher, dass die folgenden Voraussetzungen erfüllt sind:

- Docker und docker-compose sind installiert
- Du hast einen Docker-Login und Zugriff auf das Repository https://hub.docker.com/repository/docker/tkrauseiuk/lageboard
  Die benötigten Zugangsdaten bekommst du beim Inhaber dieses Repositories.
- Du hast einen Mapbox-Accesstoken (dieser ist für die Lagekarte notwendig). Du kannst dir einen kostenlosen Account bei [mapbox.com](https://www.mapbox.com) anlegen, dort findest du dann den Token.

## Vorbereitung

1. Klone dieses Repository:
  ```bash
  git clone https://github.com/ThomasKra/lageboard-docker-compose.git
  cd lageboard-docker-compose
  ```

2. Erstellen der notwendigen Dateien im Ordner "configs"
  Erstelle eine `lageboard.env` Datei (z.B. durch Kopieren der `example.env`) und ändere die Variablen.
  Beim ersten Start kann es hilfreich sein, die Benutzer aus einer Datei zu importieren. Verwende hierzu die `user_seed.example.json` und kopiere diese in `user_seed.json`. Passe die Rollen, Profile und Benutzer so an, wie du es brauchst.

3. Ändere die Benutzernamen und Passwörter für die Datenbank
  Vor dem Start der Container solltest du die Passwörter für die Datenbank ändern (und ggf. auch Benuternamen und Datenbanknamen).
  Dies kannst du in der Datei `configs/mariadb.env`:
  ```env
MYSQL_DATABASE=LageBoard
MYSQL_USER=LageBoard
MYSQL_PASSWORD=mypassword
MYSQL_ROOT_PASSWORD=rootpass
  ```

**Wichtig ist, dass die Verbindungsdaten mit denen in der `configs/lageboard.env` übereinstimmen:**
```env
DB_CONNECTION=mysql
DB_HOST=mariadb
DB_PORT=3306
DB_DATABASE=LageBoard
DB_USERNAME=LageBoard
DB_PASSWORD=mypassword
```
- Der Wert `DB_CONNECTION` muss auf `mysql` stehen!
- Der Wert `DB_HOST` muss dem Container-Namen der Datenbank entsprechen (Standardmäßig ist das `mariadb`)
- Der Wert `DB_PORT` muss dem Port entsprechen, auf dem der Datenbankserver läuft (Standardmäßig ist das `3306`)
- der Wert `DB_DATABASE` muss dem Datenbanknamen entsprechen (gleicher Wert wie `MYSQL_DATABASE` in `configs/mariadb.env`)
- der Wert `DB_USERNAME` muss dem Datenbanknamen entsprechen (gleicher Wert wie `MYSQL_USER` in `configs/mariadb.env`)
- der Wert `DB_PASSWORD` muss dem Datenbanknamen entsprechen (gleicher Wert wie `MYSQL_PASSWORD` in `configs/mariadb.env`)

4. Weitere notwendige Änderungen in der `lageboard.env`
  - `APP_URL` muss auf die URL angepasst werden, unter der die LageBoard erreichbar ist. Wenn das Port-Mapping im Docker-Compose verwendet wird, muss der Port des Docker-Hosts hier verwendet werden.
  - `EINSATZVERWALTUNG_SERVERNAME`: Hier kann ein Name für die Instanz eingetragen werden, z.B.: "IuK Rosenheim"
  - `EINSATZVERWALTUNG_CUSTOM_HEADER_COLOR`: Wenn die Kopfzeile der Software eine andere Farbe haben soll, dann kann dies hier definiert werden - dazu muss der Kommentar am Anfang der Zeile entfernt werden. Jede CSS-Farbe ist ein gültiger Wert - als Referenz siehe https://developer.mozilla.org/de/docs/Web/CSS/named-color
  - `EINSATZVERWALTUNG_MAPS_ACCESS_TOKEN`: Hier muss ein gültiger Access-Token für Mapbox eingetragen werden. Diesen bekommst du, indem du dich bei mapbox.com anmeldest.
  - `EINSATZVERWALTUNG_MAPS_OFFLINE`: darf `true` oder `false` sein. Wenn `true`, dann werden die Kartendaten offline gespeichert.
  - `EINSATZVERWALTUNG_MAPS_TILE_MAX_AGE_DAYS`: maximales Alter von offline-Tiles -- wird dieses Alter überschritten, wird automatisch versucht eine neue Version aus dem Internet zu laden.
5. Docker-Login
  Um das Docker-Image herunterladen zu können musst du dich vorher bei Docker anmelden. 
  
  Unter Linux geht das in der Kommandozeile mit 
  ```bash
  docker login --username DeinDockerBenutzername
  ```
  Anschließend muss dann das Passwort eingegeben werden. Weitere Infos dazu auch unter https://docs.docker.com/reference/cli/docker/login/

### Weitere Konfigurationen
Folgende Werte können noch in der `lageboard.env` angepasst werden:
- `APP_NAME`: Dieser Name wird z.B. für das Cookie verwendet - muss nicht angepasst werden

# Starten des Stacks
In der Datei `docker-compose.yml` ist ein Stack aus LageBoard und Datenbank (MariaDB) sowie einem Webtool zur Datenbank-Administration (Adminer) definiert (diese ist standardmäßig auskommentiert, falls du es verwenden willst, müssen die Kommentare entfernt werden).

Über `docker-compose up -d` kann der Stack installiert und gestartet werden.
Beim Start des LageBoard-Containers wird automatisch die URL in den Scripte korrigiert und automatisch ein `php artisan optimize` ausgeführt.

Folgende Befehle müssen bei Bedarf manuell ausgeführt werden:
- `php artisan migrate --force`
- `php artisan app:init`
- `php artisan app:user-import`

Ob eine Migration ausgeführt werden muss kann über `php artisan migrate:status`geprüft werden: sind alle Migrations durchlaufen worden, muss der Befehl nicht ausgeführt werden

Um die obenstehenden Befehle im Container auszuführen kann man dies im Terminal über 
```bash
docker-compose exec lageboard php artisan migrate:status
```
## Nächste Schritte nach frischer Installation:
- APP_KEY automatisch erstellen: `php artisan key:generate`. Damit wird ein neuer Schlüssel erstellt, der dann automatisch in der `config/lageboard.env` als `APP_KEY` eingetragen wird. 

**Mit diesem Schlüssel werden die Daten in der Datenbank verschlüsselt! Eine Änderung des Schlüssels hat zur Folge, dass aktuelle Daten in der Datenbank nicht mehr verwendbar sind! Es kann auch nur auf Backups zugegriffen werden, die mit dem selben Schlüssel erstellt wurden!**

- Der Befehl `php artisan app:init` ist nur nötig, wenn es sich um eine frische Installation handelt, bei der die Datenbank noch nicht initialisiert wurde.

- Mit dem Befehl `php artisan app:user-import` können die Benutzer, Rollen und Profile die in der Datei `config/user_seed.json` definiert sind in die Datenbank geladen werden.

**Es werden dabei alle vorhanden Benutzer, Rollen und Profile überschrieben!**

## Fehlerbehebung
- **Fehlende Dateien**: Vergewissern Sie sich, dass die benötigten Konfigurationsdateien `config/mariadb.env` und `config/lageboard.env` sowie `config/user-seed.json` vorhanden sind.
