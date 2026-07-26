# Zarzadca — Aplikacja do zarządzania kamienicą

## Opis

Desktopowa aplikacja do kompleksowego zarządzania starą kamienicą czynszową.
Umożliwia prowadzenie ewidencji lokali i najemców, wystawianie miesięcznych rachunków,
coroczną waloryzację czynszu według wskaźnika GUS, ewidencję przeglądów i remontów
oraz automatyczne rozliczenia ze współwłaścicielami.

Baza danych SQLite może działać lokalnie lub na dysku sieciowym — każdy komputer
w sieci uruchamia tę samą aplikację i wskazuje ścieżkę do wspólnego pliku bazy.

---

## Funkcjonalności

### Pulpit
Strona startowa z kartami podsumowującymi: liczba lokali, aktywni najemcy, rachunki z bieżącego miesiąca, zbliżające się przeglądy. Kliknięcie karty przechodzi do odpowiedniego panelu.

### Lokale i Najemcy
- Ewidencja lokali (numer, piętro, powierzchnia, liczba pokoi)
- Domyślne stawki dla lokalu (czynsz, prąd, MPGK, sprzątanie) — automatycznie wypełniają nowy rachunek
- Najemcy: osoby fizyczne i firmy (NIP), historia najemców dla każdego lokalu

### Rachunki
- Wystawianie miesięcznych rachunków: czynsz, prąd, woda, MPGK, sprzątanie, inne
- Podgląd rachunku i eksport do PDF (z polskimi znakami, konto bankowe, kwota słownie)
- Wiele kont bankowych — wybór konta przy wystawianiu rachunku
- Filtrowanie po miesiącu, roku, lokalu i statusie
- Oznaczanie rachunków jako opłacone
- Widok wszystkich rachunków wybranego najemcy (z filtrem roku i podglądem PDF)

### Import rachunków z pliku
Kreator 3-krokowy: wybór pliku → mapowanie kolumn → podgląd i import.
- **Excel** (`.xlsx`, `.xls`) — dowolny układ kolumn, ręczne mapowanie
- **PDF z tabelą danych** — generyczny ekstraktor tabel
- **PDF-faktura** (format `RACHUNEK NR X/RRRR`) — dedykowany parser: automatycznie wyciąga rok, miesiąc i kwoty z tytułu i tabeli; lokalizuje lokal po nazwie najemcy

### Waloryzacja czynszu
- Pobieranie aktualnego wskaźnika CPI z API BDL GUS (zmienna 217230)
- Podgląd nowych stawek dla wszystkich lokali przed zastosowaniem
- Jednorazowe zastosowanie waloryzacji z zapisem w historii
- Historia poprzednich waloryzacji z datami i procentami

### Przeglądy i Remonty
- Ewidencja przeglądów technicznych z datą następnego przeglądu
- Alerty o zbliżających się terminach (widoczne na pulpicie)
- Historia remontów z opisem prac i kosztami

### Rozliczenia ze współwłaścicielami
- Ręczne pozycje kosztowe i przychodowe budynku (podatek od nieruchomości, ubezpieczenie, wywóz śmieci itp.) — data, numer dokumentu/FV, nazwa, kwota, typ (wydatek / przychód)
- Automatyczne rozliczenie za wybrany okres (zakres miesięcy): przychody z rachunków + przychody dodatkowe, koszty remontów, przeglądów i dodatkowe
- Podział przychodów i kosztów według udziałów procentowych
- Eksport rozliczenia do PDF z zestawieniem ręcznych pozycji

### Ustawienia
- Ścieżka do bazy danych (lokalna lub sieciowa `\\SERWER\...`)
- Dane kamienicy: adres, właściciel, domyślne konto bankowe

---

## Technologia

| Element | Technologia |
|---|---|
| Język | Python 3.11+ |
| GUI | PyQt6 |
| Baza danych | SQLite (WAL mode) |
| Import Excel | openpyxl |
| Import / analiza PDF | pdfplumber |
| Eksport PDF | reportlab |
| Dane GUS | REST API BDL (bdl.stat.gov.pl) |

---

## Instalacja

```bash
git clone https://gitlab.com/MarcinCichy/zarzadca.git
cd zarzadca
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

## Uruchomienie

```bash
python main.py
```

Przy pierwszym uruchomieniu aplikacja tworzy plik `zarzadca.db` w katalogu projektu.
Ścieżkę można zmienić w **Ustawienia → Baza danych**.

---

## Konfiguracja sieci lokalnej

Aby korzystać z jednej bazy na wielu komputerach:

1. Umieść plik `zarzadca.db` na dysku sieciowym (np. `\\SERWER\Wspolny\zarzadca.db`)
2. Na każdym komputerze zainstaluj aplikację
3. W **Ustawienia → Baza danych** wpisz ścieżkę sieciową i kliknij **Zapisz i połącz**

Baza działa w trybie WAL (Write-Ahead Logging), który umożliwia jednoczesny odczyt
przez wielu użytkowników bez blokowania.

---

## Autor

[MarcinCichy](https://gitlab.com/MarcinCichy)
