# KNOWLEDGE.md — Wiedza Statyczna o Projekcie

> Zawiera informacje, które rzadko się zmieniają.
> Aktualizuj gdy zmienia się architektura, stack lub konwencje.

---

## Architektura

<!-- Opisz jak komponenty projektu łączą się ze sobą. -->
<!-- Możesz użyć diagramu ASCII lub listy. Przykład: -->

```
[Przeglądarka]
     ↓ HTTP REST
[Flask API — app.py]
     ↓ SQLAlchemy ORM
[SQLite / PostgreSQL]
```

Główne komponenty:
- **[Komponent A]** — [co robi, jaka odpowiedzialność]
- **[Komponent B]** — [co robi, jaka odpowiedzialność]
- **[Komponent C]** — [co robi, jaka odpowiedzialność]

---

## Kluczowe Pliki

<!-- Nie wszystkie pliki — tylko te, które są najważniejsze. -->

| Plik | Rola |
|------|------|
| `[plik1]` | [co zawiera / co robi] |
| `[plik2]` | [co zawiera / co robi] |
| `[plik3]` | [co zawiera / co robi] |
| `[plik4]` | [co zawiera / co robi] |

---

## Stack Technologiczny

<!-- Języki, frameworki, biblioteki, zewnętrzne serwisy. -->

- **Język:** [Python 3.11 / Node 20 / itp.]
- **Framework:** [Flask / FastAPI / Express / itp.]
- **Baza danych:** [SQLite / PostgreSQL / MongoDB / itp.]
- **Frontend:** [React / Vue / Vanilla JS / itp.]
- **Testy:** [pytest / jest / itp.]
- **Inne:** [lista ważnych bibliotek]

---

## Jak Uruchomić

<!-- Dokładne komendy. Ktoś obcy powinien móc uruchomić projekt po przeczytaniu tej sekcji. -->

```bash
# [krok 1 — np. aktywacja środowiska]
# [krok 2 — np. instalacja zależności]
# [krok 3 — np. uruchomienie]
```

Port: `[numer]` | URL: `http://localhost:[port]`

---

## Konwencje

<!-- Ważne reguły kodowania specyficzne dla tego projektu. -->

- [Konwencja 1 — np. nazewnictwo funkcji]
- [Konwencja 2 — np. struktura odpowiedzi API]
- [Konwencja 3 — np. organizacja testów]
- [Konwencja 4 — np. obsługa błędów]

---

*Ostatnia aktualizacja: [YYYY-MM-DD]*
