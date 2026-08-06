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
| Język | Python 3.13 (wspierana wersja produkcyjna; testowane na 3.13.14) |
| GUI | PySide6 / Qt |
| Baza danych | SQLite, WAL (lokalnie) / rollback journal (sieciowo); opcjonalnie SQLCipher (szyfrowanie hasłem) |
| Import Excel | openpyxl |
| Import PDF | pdfplumber |
| Eksport PDF | reportlab |
| HTTP / GUS | requests |
| Licencje | cryptography, podpis RSA |
| Testy | pytest |
| Typy statyczne | mypy |

Krótki przegląd warstw kodu (GUI, serwisy, DAO, baza, import, PDF, licencje):
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

Instrukcja obsługi codziennej pracy z klientami (wydawanie licencji,
sprawdzanie licencji, odzyskiwanie hasła do zaszyfrowanej bazy klienta):
[`docs/PROCEDURY_ADMIN.md`](docs/PROCEDURY_ADMIN.md).

## Instalacja developerska

Wymagany Python 3.13. Na Python 3.14 obserwowano ostrzeżenie `Could not find
platform independent libraries <prefix>` przy starcie — PySide6 6.11 nie było
na nim testowane, więc do czasu potwierdzenia pełnej zgodności zalecane jest
pozostanie przy 3.13.

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

## Statyczne sprawdzanie typów

```powershell
python -m mypy database services utils gui importer main.py
```

Konfiguracja w `mypy.ini`. To nie jest krok wymagany do uruchomienia aplikacji —
to osobne narzędzie, które czyta adnotacje typów (`: int`, `-> dict | None`, ...)
i szuka niespójności między nimi *bez* uruchamiania kodu (np. funkcja obiecuje
zwrócić `int`, a w pewnej ścieżce może zwrócić `None`). Warto uruchamiać przed
commitem przy zmianach dotykających sygnatur funkcji — na dziś kod przechodzi
bez błędów (`Success: no issues found`).

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

Na ścieżce sieciowej aplikacja automatycznie przełącza się z trybu WAL na
zwykły rollback journal (`czy_sciezka_sieciowa()` w `database/db.py`) — WAL
wymaga pliku pamięci współdzielonej, który nie zawsze działa poprawnie na
sieciowych systemach plików (SMB/CIFS), więc na sieci wybieramy bezpieczniejszą
opcję. Dodatkowo, przed otwarciem formularza edycji aplikacja ostrzega, jeśli
ten sam rekord edytuje już ktoś inny (semafor z 30-minutowym leasingiem,
`utils/blokady.py`) — to nie zastępuje ochrony przed nadpisaniem przy zapisie
(`updated_at`), ale pozwala uniknąć sytuacji, w której dwie osoby wypełniają
ten sam formularz jednocześnie.

Uprawnienia do pliku bazy i katalogu z backupami powinny być
kontrolowane na poziomie Windows lub udziału sieciowego:

- Ogranicz dostęp do folderu z `zarzadca.db` i katalogu kopii zapasowych (`.zip`)
  tylko do kont Windows osób, które faktycznie zarządzają nieruchomościami —
  baza zawiera dane osobowe najemców (imię, nazwisko, telefon, email, NIP).
- Na udziale sieciowym (`\\SERWER\...`) ustaw uprawnienia NTFS/udziału tak, aby
  zapis mieli tylko zaufani użytkownicy aplikacji, nie "Wszyscy"/"Everyone".
- Kopie zapasowe (`.zip`) zawierają pełną kopię bazy — traktuj je z tą samą
  ostrożnością co plik `zarzadca.db`, nie zostawiaj ich w folderach współdzielonych
  bez kontroli dostępu (np. publiczny folder Dropbox/OneDrive).

## Szyfrowanie bazy danych

Baza może być opcjonalnie zaszyfrowana (SQLCipher) — dane najemców (imię,
nazwisko, telefon, email, adres) nie leżą wtedy w pliku `.db` czystym tekstem.
Włącza się to w `Ustawienia -> Baza danych` przyciskiem **"Zaszyfruj bazę
hasłem…"** (widoczny tylko gdy baza jeszcze nie jest zaszyfrowana). Oryginalny,
niezaszyfrowany plik zostaje zachowany obok jako `*.przed_szyfrowaniem.bak` na
wypadek problemów. Aplikacja pyta o hasło przy każdym starcie, gdy baza jest
zaszyfrowana — zawsze, bez opcji "zapamiętaj to urządzenie".

**Mechanizm (klucz podwójnie zawinięty, jak "recovery key" w BitLockerze):**
losowy klucz główny (MEK) faktycznie szyfruje bazę i jest zapisany dwa razy w
pliku `<baza>.db.keyring.json` obok pliku bazy (musi być dostępny na każdym
stanowisku przy bazie na dysku sieciowym):

- kopia **użytkownika** — hasłem klienta (PBKDF2 + AES-256-GCM),
- kopia **odzyskiwania** — kluczem publicznym oddzielnej pary RSA-2048
  przeznaczonej tylko do tego celu (`tools/db_recovery_genkeys.py`,
  analogicznie do pary kluczy licencyjnych, ale celowo inna).

**To nie jest szyfrowanie zero-knowledge** — kto ma plik keyringu i prywatny
klucz odzyskiwania, ten odzyska dane bez hasła klienta. To świadomy kompromis
na wypadek zapomnianego hasła (bez tego: zapomniane hasło = trwała utrata
wszystkich danych najemców/rachunków klienta, bez możliwości odzyskania).
**Ta konsekwencja musi być jasno opisana w polityce prywatności/EULA** — nie
reklamuj tego jako "nikt oprócz Ciebie nie ma dostępu".

**Odzyskiwanie dostępu klientowi, który zapomniał hasła:**

1. Klient wysyła Ci (mailem) **tylko** swój plik `<nazwa_bazy>.db.keyring.json`
   — nigdy całą bazę.
2. `python tools\db_recover.py sciezka\do\przyslanego.keyring.json`
3. Podajesz nowe hasło — skrypt zapisuje `*.recovered.keyring.json`.
4. Odsyłasz ten plik klientowi; podmienia swój keyring i loguje się nowym hasłem.

Wymaga `tools/db_recovery_private.pem` — wygeneruj raz przez
`python tools\db_recovery_genkeys.py` i przechowuj z tą samą ostrożnością co
`tools/private.pem` (patrz sekcja "Licencje" niżej): nigdy nie commituj, nigdy
nie wysyłaj klientowi, zrób jedną zaszyfrowaną kopię zapasową.

## Licencje

**Ograniczenia obecnej ochrony licencyjnej:**

- Plik `.lic` jest podpisany RSA-2048 — edycja jego zawartości (np. zmiana
  `max_lokali`, `max_budynki` czy daty ważności) unieważnia podpis i licencja
  przestaje działać. Ta część jest solidna.
- Plik `.lic` **nie jest powiązany ze sprzętem** — nie ma sprawdzania numeru
  seryjnego dysku, MAC adresu ani innego identyfikatora komputera. Skopiowanie
  pliku `%APPDATA%\Zarzadca\license.lic` na inny komputer daje temu komputerowi
  pełny dostęp do funkcji z tej licencji, bez ponownego zakupu. To znany,
  świadomie zaakceptowany na razie kompromis — powiązanie z sprzętem to
  osobne zadanie do rozważenia, jeśli skala sprzedaży zacznie czynić to
  realnym problemem.
- Stan triala jest zapisywany offline jako podpisany rekord z datą startu,
  datą ostatniego uruchomienia i identyfikatorem maszyny. Na Windows aplikacja
  zapisuje go w `%APPDATA%\Zarzadca\trial_state.json` oraz dodatkowo w rejestrze
  `HKCU\Software\Zarzadca\TrialState`; na macOS/Linux używany jest podpisany
  plik w katalogu danych aplikacji. To utrudnia prostą edycję/reset triala i
  wykrywa cofnięcie daty systemowej, ale nie jest tak mocne jak aktywacja online.
- Docelowa aktywacja online będzie wymagała osobnego backendu/API, bazy danych
  licencji i hostingu. Bez serwera aplikacja desktopowa zawsze przechowuje część
  stanu lokalnie.

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

**Bezpieczne przechowywanie `tools/private.pem`:**

- To jedyny sekret, który pozwala wygenerować ważną licencję dla dowolnego
  klienta — traktuj go jak klucz do całego modelu sprzedaży aplikacji, nie
  jak zwykły plik konfiguracyjny.
- Nie trzymaj go na dysku, który jest automatycznie synchronizowany do chmury
  publicznej (OneDrive/Dropbox) bez szyfrowania, ani na współdzielonym
  komputerze.
- Zrób jedną zaszyfrowaną kopię zapasową (np. w menedżerze haseł obsługującym
  załączniki, albo w zaszyfrowanym archiwum z osobnym hasłem) na wypadek awarii
  dysku — bez kopii utrata pliku oznacza brak możliwości wystawienia nowych
  licencji istniejącym i nowym klientom.
- `.gitignore` już blokuje `tools/private.pem` i `tools/*.lic` przed
  przypadkowym commitem (zweryfikowane testami `tests/test_license_validator.py`
  dla samego mechanizmu podpisu) — to nie zastępuje ostrożności przy kopiowaniu
  pliku ręcznie (np. między komputerami, na pendrive).

## Build instalatora

Wymagania jednorazowe:

1. [Inno Setup 6](https://jrsoftware.org/isdl.php).
2. Skonfigurowane `venv` z zainstalowanymi zależnościami (`pip install -r requirements.txt`).
   Nuitka doinstaluje się automatycznie przy pierwszym uruchomieniu skryptu, jeśli jej brak.

Uruchomienie (z katalogu głównego projektu):

```powershell
.\tools\build.ps1
```

Skrypt: eksportuje EULA do `tools\eula.txt`, kompiluje aplikację przez Nuitka
(`--standalone`, PySide6, bez konsoli), a następnie buduje instalator przez
Inno Setup (`tools\installer.iss`). Wynik trafia do
`installer_output\Zarzadca_Setup_1.0.0.exe`.

Nuitka kompiluje Python → C++ → natywny kod maszynowy (trudniejszy do
zdekompilowania niż PyInstaller) i jest darmowa do użytku komercyjnego w
edycji Community.

⚠ Uruchomienie i przetestowanie zbudowanego instalatora na czystym systemie
Windows bez Pythona **nie zostało jeszcze zweryfikowane** (patrz `PLAN.md`,
Etap 11) — traktuj wynik `build.ps1` jako niesprawdzony do czasu takiego testu.

## Checklist przed wydaniem nowej wersji

1. `python -m pytest` — wszystkie testy zielone.
2. `python -m mypy database services utils gui importer main.py` — brak błędów.
3. Ręczny smoke test GUI: przejście po wszystkich panelach i zakładkach
   Ustawień na kopii realnej bazy (`tests/test_gui_smoke.py` pokrywa to
   automatycznie, ale warto też rzucić okiem na wygląd po większych zmianach GUI).
4. `python tools\diagnose_env.py` — sprawdź wersję Pythona/PySide6/Qt na maszynie budującej.
5. Zaktualizuj numer wersji w `gui/panels/ustawienia.py` (zakładka "O aplikacji")
   i w `tools\installer.iss`.
6. `.\tools\build.ps1` — zbuduj instalator.
7. Zainstaluj i uruchom zbudowany `.exe` na czystej maszynie (bez Pythona/venv)
   — potwierdź, że aplikacja startuje i podstawowe funkcje działają.
8. Zaktualizuj `PLAN.md`/`refactor_plan.md`, jeśli wydanie zamyka jakiś etap.

## Autor

Marcin Cichy
