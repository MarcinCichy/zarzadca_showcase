# Zarzadca

Desktopowa aplikacja do zarzadzania kamienica czynszowa: lokale, najemcy,
rachunki, przeglady, remonty, waloryzacja czynszu, rozliczenia ze
wspolwlascicielami oraz kopie zapasowe bazy.

![Zrzut ekranu - ustawienia](screenshots/about.png)

![Zrzut ekranu - glowny widok](screenshots/main.png)

## Funkcje

### Pulpit

Strona startowa z kartami podsumowujacymi liczbe lokali, aktywnych najemcow,
rachunki z biezacego miesiaca, zaleglosci oraz zblizajace sie przeglady.

### Lokale i Najemcy

- Ewidencja lokali: numer, pietro, powierzchnia, liczba pokoi i opis.
- Domyslne stawki lokalu: czynsz, prad, MPGK i koszty sprzatania.
- Najemcy jako osoby fizyczne albo firmy z NIP.
- Historia najemcow przypisana do lokalu.

### Rachunki

- Wystawianie miesiecznych rachunkow.
- Pozycje: czynsz, prad, woda, MPGK, sprzatanie i inne.
- Wiele kont bankowych do wyboru na rachunku.
- Filtrowanie po miesiacu, roku, lokalu i statusie.
- Oznaczanie rachunkow jako oplacone.
- Podglad rachunku i eksport do PDF.

### Import rachunkow

- Import z Excela (`.xlsx`, `.xls`) z recznym mapowaniem kolumn.
- Import z PDF z tabelami danych.
- Dedykowany parser PDF dla rachunkow w formacie `RACHUNEK NR X/RRRR`.

### Waloryzacja

- Pobieranie wskaznika CPI z API BDL GUS.
- Podglad nowych stawek przed zastosowaniem.
- Zapis historii waloryzacji.
- Generowanie wyrownania lutowego.

### Przeglady i Remonty

- Ewidencja przegladow technicznych.
- Alerty o zblizajacych sie terminach.
- Historia remontow, kosztow i wykonawcow.

### Rozliczenia

- Rozliczenia ze wspolwlascicielami wedlug udzialow procentowych.
- Reczne pozycje kosztowe i przychodowe budynku.
- Eksport rozliczenia do PDF.

### Ustawienia

- Lokalna albo sieciowa sciezka do bazy SQLite.
- Obsluga wielu budynkow.
- Konta bankowe.
- Kopie zapasowe i przywracanie bazy.

## Technologia

| Element | Technologia |
|---|---|
| Jezyk | Python 3.11+ |
| GUI | PySide6 / Qt |
| Baza danych | SQLite, WAL mode |
| Import Excel | openpyxl |
| Import PDF | pdfplumber |
| Eksport PDF | reportlab |
| HTTP / GUS | requests |
| Licencje | cryptography, podpis RSA |
| Testy | pytest |

## Instalacja developerska

```powershell
git clone https://gitlab.com/MarcinCichy/zarzadca.git
cd zarzadca
python -m venv .venv
.\.venv\Scripts\activate
python -m pip install -r requirements.txt
```

For an exact copy of the dependency versions used during development, install
from the lock file instead:

```powershell
python -m pip install -r requirements-lock.txt
```

## Uruchomienie

```powershell
python main.py
```

Przy pierwszym uruchomieniu aplikacja tworzy lub otwiera baze wskazana w
`config.json`. Domyslnie jest to `zarzadca.db` w katalogu projektu. Sciezke mozna
zmienic w aplikacji w panelu `Ustawienia -> Baza danych`.

## Testy

```powershell
python -m pytest
```

Testy uzywaja osobnych baz SQLite tworzonych w katalogu `test_runtime_dbs/`.
Nie korzystaja z lokalnego pliku `zarzadca.db`.

## Diagnostyka srodowiska

Przy roznicach wygladu albo zachowania miedzy komputerami uruchom:

```powershell
python tools\diagnose_env.py
```

Skrypt wypisuje wersje Pythona, PySide6, Qt, aktywny styl Qt, dostepne style Qt
oraz sciezke aktualnie skonfigurowanej bazy.

## Konfiguracja sieci lokalnej

Aby korzystac z jednej bazy na wielu komputerach:

1. Umiesc plik `zarzadca.db` na dysku sieciowym, np. `\\SERWER\Wspolny\zarzadca.db`.
2. Na kazdym komputerze zainstaluj aplikacje.
3. W `Ustawienia -> Baza danych` wpisz sciezke sieciowa.
4. Kliknij `Zapisz i polacz`.

Baza dziala w trybie WAL, ktory umozliwia jednoczesny odczyt przez wielu
uzytkownikow. Uprawnienia do pliku bazy i katalogu z backupami powinny byc
kontrolowane na poziomie Windows lub udzialu sieciowego.

## Licencje

Aplikacja szuka pliku licencji w:

```text
%APPDATA%\Zarzadca\license.lic
```

Do generowania licencji sluzy:

```powershell
python tools\license_generator.py
```

Generator wymaga prywatnego klucza `tools/private.pem`. Tego pliku nie wolno
commitowac ani wysylac klientom.

## Build instalatora

Pipeline budowania jest opisany w:

```powershell
tools\build.ps1
tools\installer.iss
```

Ten obszar wymaga osobnego uporzadkowania dokumentacji, bo historycznie projekt
korzystal z PyQt6/PyInstaller, a aktualny kod uzywa PySide6 i build skryptu
opartego o Nuitka.

## Autor

Marcin Cichy
