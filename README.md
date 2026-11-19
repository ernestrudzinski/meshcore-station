# MeshCore Station

Lokalna stacja MeshCore na Lenovo z Debianem: docelowo wiadomości, kontakty,
diagnostyka, trwała historia i panel web dostępny przez Tailscale.

Projekt jest niezależny od wcześniejszego projektu Meshtastic.

## Stan projektu

Zakończono rozpoznanie środowiska i przygotowano
[plan pierwszego uruchomienia](docs/pierwsze-uruchomienie.md).
Odbiornik ani panel nie są jeszcze zaimplementowane lub uruchomione.

Po wgraniu Companion USB i fizycznym restarcie przez użytkownika potwierdzono
komunikację USB: **Heltec V3**, firmware **v1.18.0-81dbae5**, kompilacja
**15-Oct-2026**, wersja protokołu zgłoszona przez radio: **14**.
Odczyt wykonano 19 listopada 2026; szczegóły w aktualizacji planu.
Następny etap to minimalny odbiornik. Lenovo nie ma Bluetooth; transportem jest USB.

## Dane lokalne

Konfiguracja, klucze, wiadomości, współrzędne, kopie flash i bazy danych nie
trafiają do Git. Przykłady konfiguracji powinny zawierać wyłącznie fikcyjne dane.
