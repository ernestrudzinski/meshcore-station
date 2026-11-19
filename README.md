# MeshCore Station

Lokalna stacja MeshCore na Lenovo z Debianem: docelowo wiadomości, kontakty,
diagnostyka, trwała historia i panel web dostępny przez Tailscale.

Projekt jest niezależny od wcześniejszego projektu Meshtastic.

## Stan projektu

Zakończono rozpoznanie środowiska i przygotowano
[plan pierwszego uruchomienia](docs/pierwsze-uruchomienie.md).
Odbiornik ani panel nie są jeszcze zaimplementowane lub uruchomione.

Po początkowej instalacji BLE użytkownik potwierdził wgranie **Companion USB**.
Na jego polecenie wysłano sygnał restartu przez mostek USB. Radio nadal nie
odpowiedziało na zapytania identyfikacyjne. Wersja firmware i skuteczny start
programu pozostają niepotwierdzone; szczegóły w aktualizacji planu.
Lenovo nie ma Bluetooth; docelowym transportem pozostaje USB.

## Dane lokalne

Konfiguracja, klucze, wiadomości, współrzędne, kopie flash i bazy danych nie
trafiają do Git. Przykłady konfiguracji powinny zawierać wyłącznie fikcyjne dane.
