# MeshCore Station

Lokalna stacja MeshCore na Lenovo z Debianem: docelowo wiadomości, kontakty,
diagnostyka, trwała historia i panel web dostępny przez Tailscale.

Projekt jest niezależny od wcześniejszego projektu Meshtastic.

## Stan projektu

Zakończono rozpoznanie środowiska i przygotowano
[plan pierwszego uruchomienia](docs/pierwsze-uruchomienie.md).
Odbiornik ani panel nie są jeszcze zaimplementowane lub uruchomione.

Użytkownik potwierdził wgranie **Companion BLE**, bez późniejszego restartu.
Lenovo nie ma Bluetooth. Planowany transport stacji to USB, wymagający
**Heltec V3 Companion USB**. Zmiana firmware wymaga osobnego polecenia użytkownika.
Wersja działającego firmware nie została odczytana z urządzenia.

## Dane lokalne

Konfiguracja, klucze, wiadomości, współrzędne, kopie flash i bazy danych nie
trafiają do Git. Przykłady konfiguracji powinny zawierać wyłącznie fikcyjne dane.
