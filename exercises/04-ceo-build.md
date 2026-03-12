# Ćwiczenie 4: Użyj /ceo do Zbudowania Projektu

**Poziom:** Zaawansowany
**Czas:** 90–180 minut
**Cel:** Użyć agenta /ceo do zaplanowania i zbudowania nietrywialnego projektu,
obserwując jak dzieli pracę między specjalistycznych agentów.

---

## Czego się nauczysz

- Jak pisać brief dla /ceo, który prowadzi do dobrego planu wykonania
- Jak oceniać org chart i plan proponowany przez /ceo
- Jak prowadzić projekt przez kilka rund poprawek
- Na co zwracać uwagę gdy agenty pracują równolegle

---

## Czym Jest /ceo?

`/ceo` to meta-agent: czyta Twój brief, projektuje zespół specjalistów
(org chart), przydziela zadania i koordynuje równoległą pracę.

Zamiast Ty → jeden agent → wynik, dostajesz:
```
Ty → /ceo → [Agent A + Agent B + Agent C] → wynik
```

---

## Wybierz Projekt

Wybierz jeden z czterech projektów lub zaproponuj własny:

### Projekt A: CLI do Zarządzania Plikami
Narzędzie wiersza poleceń z komendami: `list`, `find`, `clean`, `stats`.
- Dobry dla: osób lubiących terminal i automatyzację
- Stack: Python, Click lub argparse, Rich (ładne wyjście)

### Projekt B: Web App z Auth i CRUD
Aplikacja webowa z logowaniem, rejestracją i zarządzaniem zasobem
(np. notatki, zadania, zakładki).
- Dobry dla: osób uczących się fullstack
- Stack: Flask/FastAPI, SQLite, HTML/JS lub Jinja2

### Projekt C: Pipeline ETL
Skrypt pobierający dane z API (np. pogoda, kursy walut), transformujący
je i zapisujący do bazy + generujący raport CSV.
- Dobry dla: data science, automatyzacja
- Stack: Python, requests, pandas, SQLite

### Projekt D: Gra z Przeciwnikiem AI
Gra turowa (np. kółko-krzyżyk, czwórki w rzędzie) z AI używającym
algorytmu minimax.
- Dobry dla: osób lubiących algorytmy i gry
- Stack: Python lub HTML5/JS

---

## Krok 1: Napisz Brief dla /ceo

Brief to 3-5 zdań opisujących projekt. Dobry brief odpowiada na:
- Co budujemy?
- Dla kogo?
- Jakie są główne funkcje?
- Jakie ograniczenia technologiczne?

### Szablon Briefu

```
Zbuduj [NAZWA PROJEKTU].

Opis: [Co to jest, co robi, dla kogo].

Główne funkcje:
- [Funkcja 1]
- [Funkcja 2]
- [Funkcja 3]

Stack technologiczny: [Język, framework, baza danych].

Ograniczenia:
- [Ograniczenie 1 — np. "bez zewnętrznych API", "działa offline"]
- [Ograniczenie 2 — np. "jeden plik HTML", "Python stdlib only"]

Definicja "gotowe": [Jak wygląda sukces — np. "można zainstalować
i uruchomić jedną komendą, wszystkie funkcje działają"].
```

### Przykład Wypełnionego Briefu (Projekt A — CLI)

```
Zbuduj narzędzie CLI do zarządzania plikami o nazwie "czystka".

Opis: Narzędzie wiersza poleceń dla deweloperów, które pomaga
porządkować katalogi projektów.

Główne funkcje:
- czystka list [ścieżka]   — wylistuj pliki z rozmiarem i datą
- czystka find [wzorzec]   — znajdź pliki pasujące do wzorca (glob)
- czystka clean [ścieżka]  — usuń pliki tymczasowe (__pycache__, *.pyc, .DS_Store)
- czystka stats [ścieżka]  — statystyki katalogu (ile plików, łączny rozmiar)

Stack technologiczny: Python 3.11, Click dla CLI, Rich dla kolorowego wyjścia.

Ograniczenia:
- Tylko standardowe biblioteki + Click + Rich (brak innych zależności)
- Działa na Linux i macOS
- Instalowalne przez: pip install -e .

Definicja "gotowe": pip install -e . działa, wszystkie 4 komendy działają
z --help, czystka clean pyta o potwierdzenie przed usunięciem.
```

---

## Krok 2: Uruchom /ceo i Przejrzyj Plan

```bash
mkdir moj-ceo-projekt
cd moj-ceo-projekt
claude
```

Wklej do Claude Code:
```
/ceo [TWÓJ BRIEF TUTAJ]
```

/ceo pokaże Ci org chart i plan. Zanim powiesz „zaczynaj" — sprawdź:

### Lista Kontrolna Oceny Planu

**Org Chart:**
- [ ] Czy każdy agent ma jasno zdefiniowaną odpowiedzialność?
- [ ] Czy są agenty, które mogą pracować równolegle (faza 1 + faza 2)?
- [ ] Czy żaden agent nie ma za dużo zadań? (max 3-4 na agenta)
- [ ] Czy jest agent do testowania?

**Zadania:**
- [ ] Czy zadania mają wyraźne zależności (co musi być gotowe przed czym)?
- [ ] Czy jest zdefiniowany kontrakt między komponentami?
- [ ] Czy plan kończy się integracją i weryfikacją?

**Jeśli plan jest zły** — powiedz /ceo co poprawić:

```
Zmień plan: Agent Backend i Agent Frontend mogą pracować równolegle —
zdefiniuj kontrakt API najpierw jako osobne zadanie, potem odpalisz ich
jednocześnie. Dodaj też agenta do testów integracyjnych jako ostatni krok.
```

---

## Krok 3: Zatwierdź i Uruchom

Gdy plan jest dobry:

```
Wygląda dobrze. Zaczynaj.
```

/ceo uruchomi agentów. Twoja rola podczas wykonania:
- Obserwuj postęp
- Odpowiadaj na pytania jeśli agent utknął
- NIE ingeruj w kod — zostaw to agentom

⚠️ **Jeśli agent utknął na >5 minut bez postępu**, zapytaj /ceo:
```
Agent [nazwa] nie robi postępu. Co się dzieje?
```

---

## Krok 4: Review Wyników

Gdy /ceo zgłosi zakończenie, sprawdź wyniki:

```bash
# Przetestuj że wszystko działa
[komendy zależne od projektu]
```

**Czego szukać:**

1. **Czy agenty dostały właściwe modele?**
   Proste zadania (CSS, konfiguracja) powinny iść do mniejszych modeli.
   Złożona logika — do większych. /ceo powinien to robić automatycznie.

2. **Czy fazy równoległe faktycznie działały równolegle?**
   Sprawdź logi — czy agenty z tej samej fazy startowały jednocześnie?

3. **Czy integracja była potrzebna?**
   Czy komponenty pasowały do siebie od razu, czy była potrzebna poprawka?

4. **Jakość kodu:**
   Przejrzyj losowo wybrany plik z każdego agenta. Czy jest czytelny?

---

## Krok 5: Runda Poprawek

Zgłoś do /ceo listę rzeczy do poprawienia:

```
Proszę o poprawki:
1. Komenda `czystka find` zwraca ścieżki względne zamiast bezwzględnych — popraw
2. Brak obsługi błędu gdy podana ścieżka nie istnieje — dodaj informacyjny komunikat
3. Styl wyjścia `czystka stats` jest zbyt surowy — użyj Rich Table do formatowania
```

/ceo powinien delegować każdą poprawkę do odpowiedniego agenta.

---

## Kryteria Sukcesu ✅

- [ ] Projekt działa zgodnie z definicją „gotowe" z briefu
- [ ] Można go uruchomić/zainstalować jedną komendą
- [ ] /ceo użył przynajmniej 3 agentów
- [ ] Przynajmniej jedna faza miała równoległe zadania
- [ ] Przeprowadziłeś co najmniej 1 rundę poprawek

---

## Pytania do Refleksji

1. Gdzie /ceo pomylił się w planowaniu? Co naprawiłeś?
2. Który agent wykonał najlepszą robotę? Dlaczego?
3. Czy brief był wystarczająco konkretny? Co byś zmienił?
4. Ile czasu zajęło vs ilu agentów pracowało — czy to była dobra proporcja?

---

## Bonus: Porównanie

Weź ten sam projekt i zbuduj go BEZ /ceo — sam, z jednym agentem.
Porównaj:
- Czas do ukończenia
- Jakość kodu
- Ile razy musiałeś interweniować

💡 /ceo wychodzi na plus przy projektach z >3 komponentami i gdy możliwa
jest równoległa praca. Dla małych, liniowych zadań — jeden agent jest
często szybszy.
