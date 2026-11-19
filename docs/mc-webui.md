# mc-webui na Lenovo

19 listopada 2026 uruchomiono mc-webui 2.19.0 w Dockerze rootless. Panel działa
na porcie 8080 pod adresem Tailscale Lenovo. Używa HTTP wewnątrz sieci Tailscale.
Tailscale Serve na tym hoście wymaga osobnych uprawnień administratora.

Panel i `/health` zwróciły HTTP 200 z Lenovo przez adres Tailscale.
`/api/status` zgłosił aktywne połączenie z radiem. Sprawdzono integralność
bazy SQLite na hoście. Test z telefonu i test wiadomości LoRa są osobnymi
krokami. Presetu LoRa nie konfigurujemy w ramach instalacji panelu.

Historia, konfiguracja i kopie aplikacji są w katalogu
`/home/ernest/ChatGPT/MeshCore/mc-webui/data` (prawa `0700`), mapowanym jako
`/data`. Katalog jest poza repozytorium. Kontener ma dostęp wyłącznie do
wskazanego portu USB Helteca. Przy odłączeniu i ponownym podłączeniu radia
sprawdź status i w razie potrzeby odtwórz kontener, aby ponownie zamontować
urządzenie wskazywane przez `by-id`.

Polecenie odtworzenia po zatrzymaniu i usunięciu dotychczasowego kontenera
`mc-webui`. Zachowaj katalog danych. Uruchom z katalogu `MeshCore`, gdy działa
Tailscale, a port radia jest dostępny i wolny.

```sh
umask 077
mkdir -p mc-webui/data
port=/dev/serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0
tail_ip=$(tailscale ip -4)
docker --context rootless run -d --name mc-webui --restart unless-stopped \
  --publish "$tail_ip:8080:5000" \
  --mount "type=bind,src=$PWD/mc-webui/data,dst=/data" \
  --mount "type=bind,src=$port,dst=/dev/ttyUSB0" \
  -e MC_CONFIG_DIR=/data -e MC_SERIAL_PORT=/dev/ttyUSB0 \
  -e MC_DEVICE_NAME=auto -e TZ=Europe/Warsaw \
  --log-opt max-size=10m --log-opt max-file=2 \
  --health-cmd 'curl -fsS http://127.0.0.1:5000/health >/dev/null' \
  --health-interval 30s --health-start-period 20s \
  mawoj/mc-webui@sha256:b7bbf57eb0577ff3faf55c33a0de89394d729e6c35127e5da079d997521d97b9
```

Nie otwieraj portu szeregowego w innym kliencie, gdy działa mc-webui.
Panel daje dostęp do wiadomości i ustawień radia. Dostęp sieciowy ogranicza
adres Tailscale oraz reguły dostępu tailnetu.

[Dokumentacja projektu mc-webui](https://github.com/MarekWo/mc-webui)
