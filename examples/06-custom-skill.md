# Tworzenie Custom Skilla — Od Pomysłu do Użycia

> Skille w Claude Code to wielorazowe slash commands — zapisujesz prompt jako plik,
> a potem wywołujesz go jednym poleceniem. Zamiast pisać ten sam prompt za każdym razem,
> masz gotowy template z parametrami.

---

## Czym są skille?

Skill = plik markdown w `.claude/commands/`, który Claude Code rozpoznaje jako slash command.

```
.claude/
└── commands/
    ├── test.md          →  /test
    ├── streszczenie.md  →  /streszczenie
    ├── krytyk.md        →  /krytyk
    └── deploy.md        →  /deploy
```

Kiedy wpiszesz `/test` w Claude Code, agent wczytuje treść `test.md` jako swój prompt. Argument `$ARGUMENTS` to wszystko co wpiszesz po nazwie komendy.

---

## Przykład 1: `/test` — Automatyczne testowanie strony

### Problem

Co sesję ręcznie piszesz:
```
Uruchom Playwright na headless Chromium i przetestuj stronę na localhost:5000.
Sprawdź czy się ładuje, czy nawigacja działa, czy dark mode toggle działa,
czy na mobile menu hamburger się otwiera...
```

Za dużo powtórzeń. Zróbmy z tego skill.

### Plik skilla

`.claude/commands/test.md`:

```markdown
Jesteś QA Tester. Testujesz stronę webową z Playwright (Python, headless Chromium).

## Target
Przetestuj stronę/aplikację pod adresem lub ścieżką: $ARGUMENTS

## Setup
- Upewnij się że Playwright jest zainstalowany: pip install playwright && playwright install chromium
- Użyj sync_playwright API

## Scenariusze testowe

### 1. Loading
- Otwórz stronę, czekaj max 10s na załadowanie
- Sprawdź że tytuł nie jest pusty
- Sprawdź że nie ma JavaScript errors w konsoli

### 2. Nawigacja
- Znajdź wszystkie linki w nawigacji
- Kliknij każdy, sprawdź że strona się zmienia (URL lub content)
- Wróć na stronę główną po każdym kliknięciu

### 3. Responsywność
- Viewport 1920x1080 (desktop) — screenshot
- Viewport 768x1024 (tablet) — screenshot
- Viewport 375x667 (mobile) — screenshot
- Sprawdź że żaden element nie wychodzi poza viewport

### 4. Interakcje
- Jeśli jest formularz — wypełnij i wyślij
- Jeśli jest dark mode toggle — kliknij i sprawdź zmianę klasy na body
- Jeśli jest hamburger menu — kliknij i sprawdź że menu się otwiera

### 5. Performance
- Zmierz czas ładowania strony (navigation timing API)
- Jeśli > 3s — oznacz jako warning

## Output
- Screenshot przy każdym FAIL (zapisz w ./test-screenshots/)
- Na końcu wypisz summary:
  ✅ PASS: [opis]
  ❌ FAIL: [opis + screenshot path]
  ⚠️ WARN: [opis]

Plik testowy: test_page.py
```

### Użycie

```
/test http://localhost:5000
```

```
/test file:///home/beba/narzedzia-public/merge.html
```

```
/test https://stronabeby.pl
```

Claude Code wczytuje `test.md`, podmienia `$ARGUMENTS` na URL, i uruchamia pełny test suite. Zero pisania — jeden command.

### Co się dzieje po wywołaniu

1. Claude czyta `test.md` jako instrukcję
2. Tworzy `test_page.py` z Playwright testami
3. Uruchamia testy
4. Robi screenshoty przy failach
5. Wypisuje raport PASS/FAIL/WARN

---

## Przykład 2: `/streszczenie` — Pipeline 3 agentów

### Problem

Tworzysz streszczenia lektur jako HTML cheat sheets + RSVP reader. Każde streszczenie wymaga:
1. Przeczytanie materiału źródłowego
2. Wygenerowanie wizualnego cheat sheeta (HTML)
3. Wygenerowanie tekstu RSVP (speed reading)

To 3 różne zadania — idealny przypadek na multi-agent skill.

### Plik skilla

`.claude/commands/streszczenie.md`:

```markdown
Tworzysz streszczenie lektury. Input: $ARGUMENTS (tytuł lub ścieżka do tekstu źródłowego).

## Pipeline

Uruchom 3 agenty sekwencyjnie:

### Agent 1: Researcher
Przeczytaj materiał źródłowy (jeśli podano ścieżkę — przeczytaj plik; jeśli tytuł — użyj
swojej wiedzy o lekturze). Wyprodukuj:
- Streszczenie w 10-15 punktach (najważniejsze wątki, postacie, motywy)
- 5 kluczowych cytatów z kontekstem
- Tematy do interpretacji (3-5)
- Kontekst historyczny/literacki (3-5 zdań)

Zapisz output w zmiennej — przekaż do Agenta 2 i 3.

### Agent 2: HTML Generator
Na podstawie streszczenia z Agenta 1, stwórz wizualny cheat sheet jako HTML.

Wymagania:
- Single HTML file, zero zależności
- Layout: karty/sekcje, kolorowe nagłówki
- Sekcje: Bohaterowie (karty), Wątki (timeline), Cytaty (blockquote), Motywy (tagi)
- Kolory: pastelowe, czytelne, print-friendly
- Responsywny (mobile OK)
- Footer: "Wygenerowano z Claude Code"

Zapisz jako: lektury/[tytuł-kebab-case].html

### Agent 3: RSVP Generator
Na podstawie streszczenia z Agenta 1, stwórz tekst do speed-readingu.

Wymagania:
- Zwięzły tekst (500-800 słów) pokrywający całą lekturę
- Krótkie zdania (max 15 słów)
- Brak wtrąceń, nawiasów, przypisów
- Naturalny flow — jak opowiadanie
- Format: plain text, jedno zdanie na linię

Dołącz tekst RSVP do pliku HTML jako ukrytą sekcję z przyciskiem "RSVP Reader".

## Output
Plik HTML w lektury/[nazwa].html z cheat sheetem i wbudowanym RSVP.
Wypisz ścieżkę do pliku i krótkie podsumowanie co zawiera.
```

### Użycie

```
/streszczenie Ferdydurke
```

```
/streszczenie /home/beba/books/zbrodnia-i-kara.txt
```

### Co się dzieje

1. Agent 1 (Researcher) generuje strukturalne streszczenie
2. Agent 2 (HTML) tworzy wizualny cheat sheet z kartami, kolorami, cytatami
3. Agent 3 (RSVP) pisze zwięzły tekst i dodaje RSVP reader do HTML
4. Output: jeden plik HTML z wszystkim w środku

---

## Jak zaprojektować własny skill

### Krok 1: Znajdź powtarzalne zadanie

Zadaj sobie pytanie: "Czy pisałem podobny prompt więcej niż 3 razy?"

Przykłady:
- "Przetestuj stronę" → `/test`
- "Zrób code review" → `/review`
- "Zrób streszczenie" → `/streszczenie`
- "Zdeployuj na serwer" → `/deploy`
- "Zaktualizuj CLAUDE.md" → `/kontekst`

### Krok 2: Napisz prompt, który normalnie byś dał

Dosłownie — wklej prompt, który działał. Nie wymyślaj od zera.

### Krok 3: Sparametryzuj

Zamień zmienne części na `$ARGUMENTS`:

```
# Przed:
Przetestuj stronę na http://localhost:5000

# Po:
Przetestuj stronę na $ARGUMENTS
```

### Krok 4: Dodaj strukturę

Podziel na sekcje: Setup, Tasks, Output, Constraints. Dodaj edge cases.

### Krok 5: Testuj i iteruj

Uruchom skill 3 razy z różnymi argumentami. Popraw co nie działa.

---

## Template na nowy skill

`.claude/commands/moj-skill.md`:

```markdown
[Rola agenta — kim jest w kontekście tego taska]

## Input
$ARGUMENTS — [opis co to jest: URL, ścieżka, tytuł, itp.]

## Kontekst
- [Stack/technologia]
- [Pliki/katalogi do użycia]
- [Ograniczenia]

## Zadania
1. [Krok 1 — co zrobić]
2. [Krok 2 — co zrobić]
3. [Krok 3 — co zrobić]

## Output
- [Format outputu]
- [Gdzie zapisać]
- [Co wypisać użytkownikowi]

## Czego NIE robić
- [Zakaz 1]
- [Zakaz 2]
```

---

## Zaawansowane: Skill z wieloma parametrami

`$ARGUMENTS` to jeden string. Jeśli potrzebujesz wielu parametrów, użyj konwencji:

```
/deploy production v2.1.0
```

W skillu:
```markdown
## Input
$ARGUMENTS — format: [environment] [version]
Parsuj: pierwszy wyraz to environment (staging/production), drugi to version tag.
```

Claude zrozumie i sparsuje. Nie potrzebujesz formalnego parsera.

---

## Przykłady skillów z prawdziwych projektów

| Skill | Plik | Co robi |
|-------|------|---------|
| `/test` | `test.md` | Playwright testy strony webowej |
| `/streszczenie` | `streszczenie.md` | 3-agent pipeline do cheat sheetów |
| `/krytyk` | `krytyk.md` | Adversarial code review (2 perspektywy) |
| `/ceo` | `ceo.md` | Multi-agent project buildout |
| `/kontekst` | `kontekst.md` | Generuje/aktualizuje .context/ folder |
| `/deploy` | `deploy.md` | Build + test + push + deploy pipeline |

---

## 💡 Tip: Skille to dokumentacja procesu

Pisanie skilla zmusza cię do formalizacji workflow. Nawet jeśli nigdy go nie uruchomisz — sam akt opisania procesu jest wartościowy. To jak pisanie runbooka.

## 💡 Tip: Iteruj na skillach jak na kodzie

Twój pierwszy skill będzie niedoskonały. Po 3 użyciach zauważysz luki — dodaj je. Po 10 użyciach będziesz mieć dopracowany tool. Commituj zmiany w skillach tak jak w kodzie.

## ⚠️ Uwaga: Skill to nie skrypt

Skill to prompt, nie program. Nie próbuj w nim pisać logiki warunkowej, pętli ani error handling na poziomie kodu. Claude interpretuje skill jako instrukcję i sam decyduje jak ją wykonać. Pisz intencje, nie implementację.

## ⚠️ Uwaga: $ARGUMENTS jest opcjonalne

Jeśli skill nie wymaga argumentów (np. `/krytyk` który sam znajduje pliki do review) — po prostu nie używaj `$ARGUMENTS`. Claude Code przekaże pusty string.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
