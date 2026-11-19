# Plan pierwszego uruchomienia MeshCore Station

Data rozpoznania: **19 listopada 2026**, strefa Europe/Warsaw.
Zakres tego etapu: sprawdzenie środowiska, identyfikacja połączenia i plan.
Nie wykonano zmiany firmware, kasowania konfiguracji, restartu radia ani
uruchomienia odbiornika. Dokument rozróżnia obserwacje i informacje użytkownika.

## 1. Wynik i decyzja dotycząca połączenia

Użytkownik potwierdził, że **wgrał MeshCore Companion BLE**, narzędzie zgłosiło
sukces, a następnie nie restartował Helteca. Użytkownik potwierdził również,
że **Lenovo nie ma dostępu do Bluetooth** i obecnie nie jest przy urządzeniu.

Nie ustalono numeru wersji ani nazwy pliku firmware. Nie potwierdzono odczytem,
czy radio uruchomiło już nowy program. Pozostawanie w bootloaderze jest możliwym
wyjaśnieniem, a nie stwierdzonym stanem.

**Docelowy wariant dla tej stacji: `Heltec_v3_companion_radio_usb` — Companion USB.**
Oficjalna konfiguracja Heltec V3 rozdziela warianty USB i BLE. Wariant USB ma
`ENABLE_USB_INTERFACE`; wariant BLE nie udostępnia tego samego interfejsu USB
Companion. Sam kabel lub restart nie zamienia BLE w USB [1].

Na obecnym sprzęcie nie można rozpocząć pracy odbiornika stacji z potwierdzonym
przez użytkownika wariantem BLE. **Zmiana BLE na USB wymaga osobnego polecenia
użytkownika.** Nie jest wykonywana w ramach tego etapu.

Companion obsługuje aplikację użytkownika. Repeater służy rozszerzaniu zasięgu,
a Room Server obsłudze pokoju/BBS; nie wybieramy ich jako zamienników Companion
dla komunikatora. Typowy Companion nie przekazuje ruchu innych węzłów [2].

## 2. Stan Lenovo i repozytorium — sprawdzone

| Element | Wynik |
| --- | --- |
| Dostęp | Narzędzia działają bezpośrednio na Lenovo; SSH do innej maszyny nie jest potrzebne. |
| System | Debian 13.7 (trixie), x86_64, kernel 6.12.116+deb13-amd64. |
| Użytkownik | `ernest`, UID 1000, w rzeczywistej sesji należy do `dialout`. |
| Python | 3.13.5; moduły `serial` i `meshcore` nie są dostępne w systemowym Pythonie. |
| Docker | Dostępny kontekst `rootless`; działają Mosquitto i Home Assistant. Nie zmieniono ich konfiguracji ani stanu. |
| Meshtastic | `meshtastic-receiver.service` i `meshtastic-web.service`: `inactive`, `disabled`; brak kontenerów Meshtastic na liście uruchomionych kontenerów. |
| systemd użytkownika | `Linger=yes`. |
| Tailscale | Backend `Running`, urządzenie online. Nie zmieniano ustawień dostępu. |
| Port 8080 | W chwili kontroli brak nasłuchu TCP. Nie uruchomiono na nim panelu. |
| Bluetooth | Brak kontrolera w `/sys/class/bluetooth`, usługa `bluetooth.service` nieaktywna; brak `bluetoothctl` i `rfkill`. Brak Bluetooth potwierdził użytkownik. |
| Repozytorium | `git@github.com:ernestrudzinski/meshcore-station.git`: początkowo puste, gałąź domyślna `main`, widoczność publiczna, dostęp do zapisu potwierdzony. |
| Lokalna kopia | `/home/ernest/ChatGPT/MeshCore/meshcore-station`. |

Pierwszy odczyt w ograniczonym środowisku nie pokazywał portów i poprawnych grup
użytkownika. Powyższe ustalenia dotyczą późniejszego odczytu bezpośrednio na
hoście. Dziennik kernela był niedostępny dla bieżącego użytkownika; brak wpisów
nie stanowi dowodu braku urządzeń.

Nie zmieniano projektu `/home/ernest/ChatGPT/MESHTASTIC/meshtastic-station`,
jego danych, obrazów ani instrukcji powrotu
`/home/ernest/ChatGPT/MESHTASTIC/meshtastic-station/docs/pauza-meshtastic.md`.

## 3. USB i identyfikacja radia

`lsusb` wykazał dwa mostki Silicon Labs CP210x, oba `10c4:ea60`:

| Bieżący port | Identyfikacja udev | Znaczenie |
| --- | --- | --- |
| `/dev/ttyUSB0` | CP2102N, osobny długi numer seryjny, ścieżka USB kończąca się `usb-0:8:1.0` | Drugie urządzenie; nie otwierano jego portu. |
| `/dev/ttyUSB1` | CP2102, numer `0001`, ścieżka USB `pci-0000:00:14.0-usb-0:2:1.0` | Kandydat na Helteca według istniejącej reguły i aliasu. |

Istniejący `/dev/meshtastic-heltec` obecnie wskazuje `/dev/ttyUSB1`.
Reguła `/etc/udev/rules.d/99-meshtastic-heltec.rules` dopasowuje **łącznie**
VID/PID, numer `0001` i fizyczną ścieżkę USB; nadaje również ACL użytkownikowi.
Nie zmieniono tej reguły. Numer `0001` jest generyczny — sam nie identyfikuje
jednoznacznie konkretnego egzemplarza.

Stabilna ścieżka obecnego kandydata:

```text
/dev/serial/by-id/usb-Silicon_Labs_CP2102_USB_to_UART_Bridge_Controller_0001-if00-port0
```

Oba porty miały `root:dialout`, tryb `0660`; bieżący użytkownik miał prawo odczytu
i zapisu. `lsof` nie wykazał właściciela portów w zakresie widoczności użytkownika.
Nie wykonywano odczytu jako root, więc nie jest to pełny audyt wszystkich procesów.
ModemManager był nieaktywny. Diagnostyka korzystała z jednego procesu i
`TIOCEXCL` po otwarciu; ta blokada nie usuwa ewentualnych wcześniejszych właścicieli.

**Podłączenie mostka w dawnym gnieździe Helteca jest potwierdzone. Fizyczna
tożsamość płytki nie została niezależnie potwierdzona odpowiedzią firmware.**
Przy pierwszej obecności przy Lenovo należy zweryfikować etykietę płytki i zmianę
tego konkretnego wpisu po odłączeniu/podłączeniu wyłącznie Helteca.

Po każdej zmianie okablowania ponowić inwentaryzację; nie kodować `ttyUSB1` na
stałe. Przy pozostawieniu tego samego mostka preferować `by-id`, dodatkowo
sprawdzając ścieżkę fizyczną. Nowy alias `/dev/meshcore-heltec` można przygotować
później, po identyfikacji, z dopasowaniem urządzenia i gniazda. Nie jest teraz potrzebny.

## 4. Co ustalono o działającym firmware

Wykonano wyłącznie na kandydacie wskazanym przez regułę:

1. Odczyt pasywny przez 18 sekund przy 115200, 8N1: **0 bajtów**.
2. Dwa zapytania lokalnego protokołu MeshCore: `APP_START`, następnie
   `DEVICE_QUERY` (wersja protokołu klienta 3). Czas oczekiwania odpowiednio
   2 i 4 sekundy: **brak poprawnych odpowiedzi SELF_INFO i DEVICE_INFO**.

Ramki USB dobrano na podstawie oficjalnej implementacji transportu [3]. Zapytania
identyfikacyjne nie zlecają nadawania LoRa ani zapisu preferencji [4]. Nie pobierano
kolejki wiadomości, kanałów, kluczy ani kontaktów; nie wypisywano surowych danych.
Nie wywoływano sterowania DTR/RTS, celowego resetu ani wejścia w bootloader.
Przywrócono ustawienia terminala i zamknięto port. Samo otwarcie portu może zależeć
od zachowania sterownika; nie obserwowano procesu startowego urządzenia.

**Wniosek:** rodzaj wgranego firmware znamy z potwierdzenia użytkownika
(Companion BLE), natomiast wersja i uruchomienie pozostają niezweryfikowane.
Brak odpowiedzi USB jest zgodny z wybranym wariantem BLE; nie dowodzi awarii.
Nie uruchamiano klienta Meshtastic. Dawna nazwa „Oscar Base”, wersja Meshtastic
i dawny identyfikator nie są aktualnymi danymi MeshCore.

## 5. Plan od połączenia do minimalnego odbiornika

### Etap A — przygotowanie przed zmianą firmware

- Użytkownik podaje, jeśli ma dostęp: nazwę pliku i wersję wgranego BLE oraz
  informację, czy podczas instalacji kasowano flash. Tych informacji teraz brak.
- Przy obecności przy urządzeniu: potwierdzić model Heltec V3, przewód danych,
  właściwy wpis `by-id`, podłączoną antenę i komunikat na ekranie. Nie naciskać
  przycisków kasowania. Zwykły restart może uruchomić BLE, ale nie da Companion USB.
- Ustalić potrzebę zachowania tożsamości MeshCore, kanałów i kontaktów. Jeśli nowy
  BLE nie został jeszcze skonfigurowany, nie zakładać istnienia takich danych.
- Kopię pełnego flash można przygotować narzędziem Espressif po ustaleniu stanu
  układu i pojemności flash. Odczyt może wymagać resetu do bootloadera; to osobna
  czynność, nie wykonana teraz. Kopia zawiera potencjalne sekrety, musi pozostać
  poza Git z ograniczonym dostępem. Jej wykonanie i odtworzenie trzeba zweryfikować.
- Jeżeli BLE jest dostępne z telefonu, eksport obsługiwanych ustawień może być
  dodatkowym zabezpieczeniem. Nie gwarantuje pełnej kopii radia ani starego Meshtastic.

### Etap B — wyłącznie po poleceniu zmiany BLE na USB

1. Ponownie sprawdzić aktualny oficjalny flasher [5] i wybrać **Heltec V3 →
   Companion USB**, odpowiadający środowisku `Heltec_v3_companion_radio_usb` [1].
   Wybrać aktualne stabilne wydanie i zapisać jego dokładną wersję, nazwę pliku
   oraz sumę SHA-256; nie korzystać z numerów przykładowych z FAQ jako „najnowszych”.
2. Przed zapisem przedstawić konkretny obraz, sposób wgrywania i zakres kopii.
   Rozróżnić obraz aplikacji i `merged`, zweryfikować właściwe offsety oraz opcje
   kasowania dla tego wydania. Nie wykonywać `erase_flash` jako rutynowego kroku.
   FAQ opisuje różnice obrazów i utrzymania danych [6], ale nie daje gwarancji
   zachowania każdej konfiguracji przy każdej zmianie wersji i wariantu.
3. Zatrzymać ewentualny proces właściciela portu; nie zatrzymywać Mosquitto,
   Home Assistant ani nie uruchamiać Meshtastic. Wgrywać tylko do potwierdzonego Helteca.
4. Po poprawnym zakończeniu przeprowadzić kontrolowany restart do programu,
   sprawdzić ponownie port i potwierdzić odpowiedź Companion USB oraz wersję.

Na tym etapie nie ma jeszcze zatwierdzonego konkretnego obrazu do zapisania.
Nie podajemy komendy zapisu z przypadkowym offsetem lub historyczną wersją.

### Etap C — odczyt identyfikacji i zgodności

1. Utworzyć projektowe środowisko `.venv`, bez instalacji do systemowego Pythona.
2. Użyć oficjalnej biblioteki **`meshcore` z `meshcore-dev/meshcore_py`** [7].
   Wybrać i przypiąć przetestowaną wersję wraz z zależnościami w czasie wdrożenia.
3. Po weryfikacji wolnego portu otworzyć wyłącznie ustalone `by-id` przy
   **115200, 8N1**. To parametr transportu Companion, a nie ustawienie radia LoRa [3,7].
4. Najpierw lokalne `APP_START` i `send_device_query()`: sprawdzić model,
   wersję i możliwości protokołu. Nie logować całego payloadu: odpowiedź urządzenia
   może zawierać PIN BLE [8]. Własny klucz publiczny trzymać w lokalnym stanie;
   nie przenosić starego identyfikatora Meshtastic.
5. Zweryfikować częstotliwość, BW, SF i CR z danymi lokalnej sieci MeshCore.
   Wartości nie są jeszcze ustalone; nie kopiować ustawień z Meshtastic.
   Zmianę konfiguracji oddzielić od diagnostyki.

Przed użyciem fabrycznej metody połączenia przejrzeć przypiętą wersję biblioteki:
aktualny kod potrafi ponawiać połączenie, zmieniając DTR [9]. Nie traktować
takiej próby jako całkowicie pasywnego odczytu. Uzgodnić zachowanie startu
i unikać niekontrolowanych restartów w pętli.

### Etap D — minimalny odbiornik z zapisem

Proponowana pierwsza implementacja: jeden proces Pythona korzystający z
`meshcore_py`, zapisujący dane do lokalnego SQLite. Jest to decyzja projektowa,
do sprawdzenia w testach; nie przenosimy architektury poprzedniej stacji.

- Jeden właściciel portu, blokada procesu w katalogu runtime i sekwencyjna kolejka
  komend. Panel będzie korzystał z API właściciela, bez własnego dostępu do USB.
- Najpierw podłączyć obsługę zdarzeń i zapis, potem pobrać zaległe wiadomości
  do `NO_MORE_MSGS`, następnie obsługiwać `MESSAGES_WAITING`. Biblioteka udostępnia
  automatyczne pobieranie [7,9]. Dla przypiętej wersji sprawdzić, czy opróżnia całą
  początkową kolejkę, i ewentualnie wykonać jawne pobieranie w pętli.
- Pobranie z kolejki firmware usuwa jej element [4]. Transakcję zapisu wykonywać
  natychmiast; nie obiecywać pełnego braku strat przy awarii pomiędzy odbiorem
  a zapisem. Kolejka radia ma ograniczenia i nie jest trwałą historią stacji.
- Zachować czas odbioru UTC, dostępny czas wiadomości, typ, nadawcę/indeks kanału,
  treść i dostępne metadane. Brakujących pól nie uzupełniać fikcyjnymi wartościami.
  Rozróżniać powtórzenie transportowe od dwóch świadomie identycznych wiadomości.
- Baza i konfiguracja tylko lokalnie, z prawami dostępu użytkownika. Do Git trafia
  schemat i fikcyjne przykłady; treści wiadomości nie trafiają do dziennika systemd.
- Status: połączony/rozłączony, ostatni odbiór, ostatni błąd, kontrolowane ponowienia
  z opóźnieniem. Pierwsza wersja aplikacji nie wysyła wiadomości ani reklam na żądanie.
  Nie oznacza to gwarancji całkowitej ciszy radiowej firmware: może generować ACK
  i inne własne pakiety.

### Etap E — testy i nadzór

Kryteria ukończenia minimalnego odbiornika:

1. Potwierdzona wersja i model z urządzenia, poprawne ustawienia sieci, brak drugiego
   procesu otwierającego port; niezgodne lub zajęte urządzenie daje czytelny błąd.
2. Znana wiadomość testowa z drugiego węzła MeshCore trafia do SQLite; brak ruchu
   w eterze nie jest sam w sobie wynikiem negatywnym testu połączenia USB.
3. Restart procesu zachowuje historię. Rozłączenie/powrót radia daje poprawny status
   i kontrolowane połączenie po ponownym ustaleniu tożsamości urządzenia.
4. Testy na fikcyjnych zdarzeniach sprawdzają początkową kolejkę wielu wiadomości,
   ponowne zdarzenia, błąd bazy i niepełne metadane. Osobno sprawdzić realny odbiór.
5. Dopiero po teście ręcznym przygotować `meshcore-receiver.service` w systemd
   użytkownika z `Restart=on-failure` i lokalnym katalogiem danych. Sprawdzić restart
   usługi. Włączenie autostartu odnotować w dokumentacji wdrożenia.

Na początek USB pozostaje po stronie procesu na hoście. Docker rootless można
rozważyć dla późniejszego panelu; dostęp do urządzeń i prawa trzeba wtedy osobno
przetestować. MQTT i Home Assistant nie są zależnościami minimalnego odbiornika.

## 6. Dostępność funkcji docelowych

| Funkcja | Co jest możliwe i jaki jest warunek |
| --- | --- |
| Odbiór/wysyłanie wiadomości | Companion obsługuje wiadomości bezpośrednie i kanałowe. Wysyłanie dojdzie po odbiorniku; odróżnić przyjęcie do wysłania od ACK i braku potwierdzenia dla kanału [4,7]. |
| Kontakty i informacje o węzłach | Dostępne kontakty, reklamy i ich aktualizacje; są to znane radiu kontakty, nie gwarantowany spis całej sieci. Zakres zależy od firmware i uprawnień [7,8]. |
| Status i diagnostyka | Stan transportu lokalnie; wersja, bateria i obsługiwane statystyki z urządzenia. Żądanie statusu zdalnego jest osobną operacją radiową [4,8]. |
| Trwała historia | Zapewni SQLite na Lenovo i kopie lokalne. Ograniczonej kolejki radia nie traktujemy jako archiwum [4]. |
| Mapa | Tylko kontakty udostępniające prawidłową pozycję. Brak pozycji oznacza brak punktu, nie lokalizację domyślną. Lokalizacja może być stara [8]. |
| Trasy | Protokół/biblioteka udostępniają aktualizacje ścieżek, discovery i trace. Nie gwarantują pełnej topologii ani jednoznacznej geograficznej trasy każdego pakietu; krótkie hashe mogą być niejednoznaczne. Aktywne trace generuje ruch [4,7,8]. |
| Panel z telefonu | Nasz późniejszy panel komunikuje się z API Lenovo przez Tailscale. Telefon nie musi otwierać USB/BLE radia. Sprawdzić uwierzytelnienie i zakres nasłuchu przed udostępnieniem. |

## 7. Co wymaga udziału użytkownika

**Blokuje uruchomienie:** polecenie zmiany firmware z BLE na **Companion USB**.
Przy braku Bluetooth restart istniejącego BLE nie wystarczy do podłączenia stacji.
Jeśli wgrywanie zdalne okaże się niemożliwe, potrzebna będzie fizyczna obsługa
przycisków lub przewodu przy Lenovo. Nie zakładamy, że jest teraz dostępna.

Do ustalenia przed wdrożeniem:

- dokładny model i przypisanie płytki do portu; nazwa/wersja wgranego BLE, jeśli znana;
- czy istnieją ustawienia lub tożsamość MeshCore do zachowania i jakie kopie już są;
- parametry lokalnej sieci MeshCore i wybrany kanał; sekret kanału przekazać
  wyłącznie do lokalnej konfiguracji, nie do publicznego repozytorium;
- źródło jednej znanej wiadomości testowej: drugi węzeł albo współpracujący uczestnik;
- później: nazwa stacji i decyzja o publikowaniu pozycji. Nie wymagać współrzędnych
  do uruchomienia odbiornika.

**Najbliższy krok po poleceniu użytkownika:** wybór konkretnego aktualnego obrazu
Companion USB oraz procedury zachowania danych, następnie zmiana wariantu,
kontrolowany restart i ponowny odczyt identyfikacji. Minimalny odbiornik jest
następnym etapem, a nie wynikiem już wykonanego rozpoznania.

## 8. Oficjalne źródła

Sprawdzone 19 listopada 2026. Odsyłacze do `main` są zmienne; przy implementacji
zapisać konkretne wersje firmware, biblioteki oraz używane rewizje źródeł.
Dokument protokołu sam zaznacza, że jest rozwijany; zachowanie sprawdzać również
w kodzie i na urządzeniu.

1. [Oficjalne warianty Heltec V3](https://github.com/meshcore-dev/MeshCore/blob/main/variants/heltec_v3/platformio.ini).
2. [MeshCore: role i przykładowe aplikacje](https://github.com/meshcore-dev/MeshCore#readme).
3. [Transport szeregowy firmware](https://github.com/meshcore-dev/MeshCore/blob/main/src/helpers/ArduinoSerialInterface.cpp)
   i [transport biblioteki](https://github.com/meshcore-dev/meshcore_py/blob/main/src/meshcore/serial_cx.py).
4. [Obsługa komend, odpowiedzi i kolejki w Companion](https://github.com/meshcore-dev/MeshCore/blob/main/examples/companion_radio/MyMesh.cpp).
5. [Oficjalny flasher MeshCore](https://flasher.meshcore.io/).
6. [Oficjalne FAQ — aktualizacje, obrazy i konfiguracja](https://docs.meshcore.io/faq/).
7. [Oficjalna biblioteka Python meshcore_py](https://github.com/meshcore-dev/meshcore_py).
8. [Oficjalny protokół Companion](https://github.com/meshcore-dev/MeshCore/blob/main/docs/companion_protocol.md)
   i [parser odpowiedzi biblioteki](https://github.com/meshcore-dev/meshcore_py/blob/main/src/meshcore/reader.py).
9. [Połączenie i pobieranie wiadomości w meshcore_py](https://github.com/meshcore-dev/meshcore_py/blob/main/src/meshcore/meshcore.py).
