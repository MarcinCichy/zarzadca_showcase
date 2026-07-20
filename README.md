# Zarzadca — Aplikacja do zarządzania kamienicą

![Zrzut ekranu](screenshots/about.png)

## Opis

Desktopowa aplikacja do kompleksowego zarządzania starą kamienicą. Umożliwia prowadzenie dokumentacji lokali i najemców, wystawianie miesięcznych rachunków, ewidencję przeglądów i remontów oraz ręczne rozliczanie kamienicy ze współwłaścicielami według ich udziałów.

Baza danych SQLite może znajdować się na jednym komputerze w sieci lokalnej — pozostałe komputery łączą się z nią przez ścieżkę sieciową, korzystając z tego samego programu.

---

## Funkcjonalności

- **Lokale i Najemcy** — ewidencja lokali, przypisywanie najemców, historia najemców dla każdego lokalu
- **Rachunki** — wystawianie miesięcznych rachunków (czynsz, prąd, woda, gaz, inne), podgląd, eksport do PDF, filtrowanie, oznaczanie jako opłacone
- **Import rachunków** — wczytywanie starych rachunków z plików Excel (`.xlsx`) i PDF z interaktywnym mapowaniem kolumn
- **Przeglądy i Remonty** — ewidencja przeglądów z alertami o zbliżających się terminach, historia remontów z kosztami
- **Rozliczenia** — automatyczne rozliczenie ze współwłaścicielami według udziałów procentowych za wybrany okres, eksport do PDF
- **Ustawienia** — konfiguracja ścieżki do bazy danych (lokalna lub sieciowa), informacje o kamienicy

---

## Technologia

| Element | Technologia |
|---|---|
| Język | Python 3 |
| GUI | PyQt6 |
| Baza danych | SQLite |
| Import Excel | openpyxl |
| Import / analiza PDF | pdfplumber |
| Eksport PDF | reportlab |

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

Przy pierwszym uruchomieniu aplikacja tworzy plik bazy danych `zarzadca.db` w katalogu projektu. Ścieżkę do bazy można zmienić w **Ustawienia → Baza danych**.

## Konfiguracja sieci lokalnej

Aby korzystać z jednej bazy na wielu komputerach:

1. Umieść plik `zarzadca.db` na dysku sieciowym (np. `\\SERWER\Wspolny\zarzadca.db`)
2. Na każdym komputerze zainstaluj aplikację
3. W **Ustawienia → Baza danych** wpisz ścieżkę sieciową i kliknij **Zapisz i połącz**

---

## Autor

[MarcinCichy](https://gitlab.com/MarcinCichy)
