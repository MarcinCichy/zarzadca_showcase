# Zarządca

Desktopowa aplikacja do zarządzania kamienicą czynszową: lokale, najemcy,
rachunki, przeglądy, remonty, waloryzacja czynszu, rozliczenia ze
współwłaścicielami oraz kopie zapasowe bazy.

![Zrzut ekranu - ustawienia](screenshots/about.png)

![Zrzut ekranu - główny widok](screenshots/main.png)

## Funkcje

### Pulpit

Strona startowa z kartami podsumowującymi liczbę lokali, aktywnych najemców,
rachunki z bieżącego miesiąca, zaległości oraz zbliżające się przeglądy.

### Lokale i Najemcy

- Ewidencja lokali: numer, piętro, powierzchnia, liczba pokoi i opis.
- Domyślne stawki lokalu: czynsz, prąd, MPGK i koszty sprzątania.
- Najemcy jako osoby fizyczne albo firmy z NIP.
- Historia najemców przypisana do lokalu.

### Rachunki

- Wystawianie miesięcznych rachunków.
- Pozycje: czynsz, prąd, woda, MPGK, sprzątanie i inne.
- Wiele kont bankowych do wyboru na rachunku.
- Filtrowanie po miesiącu, roku, lokalu i statusie.
- Oznaczanie rachunków jako opłacone.
- Podgląd rachunku i eksport do PDF.

### Import rachunków

- Import z Excela (`.xlsx`, `.xls`) z ręcznym mapowaniem kolumn.
- Import z PDF z tabelami danych.
- Dedykowany parser PDF dla rachunków w formacie `RACHUNEK NR X/RRRR`.

### Waloryzacja

- Pobieranie wskaźnika CPI z API BDL GUS.
- Podgląd nowych stawek przed zastosowaniem.
- Zapis historii waloryzacji.
- Generowanie wyrównania lutowego.

### Przeglądy i Remonty

- Ewidencja przeglądów technicznych.
- Alerty o zbliżających się terminach.
- Historia remontów, kosztów i wykonawców.

### Rozliczenia

- Rozliczenia ze współwłaścicielami według udziałów procentowych.
- Ręczne pozycje kosztowe i przychodowe budynku.
- Eksport rozliczenia do PDF.

### Ustawienia

- Lokalna albo sieciowa ścieżka do bazy SQLite.
- Obsługa wielu budynków.
- Konta bankowe.
- Kopie zapasowe i przywracanie bazy.

## Technologia

| Element | Technologia |
|---|---|
| Język | Python 3.11+ |
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

Aby odtworzyć dokładne wersje zależności używane podczas prac, zainstaluj pakiety
z pliku lock:

```powershell
python -m pip install -r requirements-lock.txt
```

## Uruchomienie

```powershell
python main.py
```

Przy pierwszym uruchomieniu aplikacja tworzy lub otwiera bazę wskazaną w
`config.json`. Domyślnie jest to `zarzadca.db` w katalogu projektu. Ścieżkę można
zmienić w aplikacji w panelu `Ustawienia -> Baza danych`.

## Testy

```powershell
python -m pytest
```

Testy używają osobnych baz SQLite tworzonych w katalogu `test_runtime_dbs/`.
Nie korzystają z lokalnego pliku `zarzadca.db`.

## Diagnostyka środowiska

Przy różnicach wyglądu albo zachowania między komputerami uruchom:

```powershell
python tools\diagnose_env.py
```

Skrypt wypisuje wersję Pythona, PySide6, Qt, aktywny styl Qt, dostępne style Qt
oraz ścieżkę aktualnie skonfigurowanej bazy.

## Konfiguracja sieci lokalnej

Aby korzystać z jednej bazy na wielu komputerach:

1. Umieść plik `zarzadca.db` na dysku sieciowym, np. `\\SERWER\Wspolny\zarzadca.db`.
2. Na każdym komputerze zainstaluj aplikację.
3. W `Ustawienia -> Baza danych` wpisz ścieżkę sieciową.
4. Kliknij `Zapisz i połącz`.

Baza działa w trybie WAL, który umożliwia jednoczesny odczyt przez wielu
użytkowników. Uprawnienia do pliku bazy i katalogu z backupami powinny być
kontrolowane na poziomie Windows lub udziału sieciowego.

## Licencje

Aplikacja szuka pliku licencji w:

```text
%APPDATA%\Zarzadca\license.lic
```

Do generowania licencji służy:

```powershell
python tools\license_generator.py
```

Generator wymaga prywatnego klucza `tools/private.pem`. Tego pliku nie wolno
commitować ani wysyłać klientom.

## Build instalatora

Pipeline budowania jest opisany w:

```powershell
tools\build.ps1
tools\installer.iss
```

Ten obszar wymaga osobnego uporządkowania dokumentacji, bo historycznie projekt
korzystał z PyQt6/PyInstaller, a aktualny kod używa PySide6 i build skryptu
opartego o Nuitka.

## Autor

Marcin Cichy
