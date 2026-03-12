# Ćwiczenie 5: Stwórz Własny Skill

**Poziom:** Średni
**Czas:** 45–60 minut
**Cel:** Zidentyfikować powtarzalne zadanie w swojej pracy, zautomatyzować je
jako slash command (`/moj-skill`) i używać go w przyszłych sesjach.

---

## Czego się nauczysz

- Jak działają pliki skill w Claude Code
- Jak przekształcić ręczny prompt w wielokrotnie używalną komendę
- Jak używać `$ARGUMENTS` do parametryzowania skilla
- Jak iterować i ulepszać skill na podstawie użycia

---

## Czym Jest Skill?

Skill to plik Markdown przechowywany w `.claude/commands/`.
Gdy wpiszesz `/nazwa-skilla` w Claude Code, zawartość pliku staje się
promptem — z opcjonalnymi argumentami podanymi po nazwie.

```
.claude/
└── commands/
    ├── review.md        → /review
    ├── deploy.md        → /deploy
    └── test-report.md   → /test-report
```

---

## Krok 1: Wybierz Zadanie do Automatyzacji

Dobry kandydat na skill to zadanie, które:
- Robisz co najmniej raz w tygodniu
- Wymaga tego samego promptu (lub bardzo podobnego)
- Daje powtarzalny, przewidywalny wynik

### Przykłady Zadań

**Code Review:**
Przed każdym commitem prosisz Claude o przegląd zmian.

**Project Setup:**
Przy każdym nowym projekcie tworzysz tę samą strukturę plików.

**Deployment Check:**
Przed deploymentem sprawdzasz tę samą listę rzeczy.

**Test Reporter:**
Po testach prosisz o podsumowanie co się nie udało i dlaczego.

**Bug Triage:**
Wklejasz stacktrace i prosisz o diagnozę + plan naprawy.

**Refactor Helper:**
Prosisz o refaktoryzację konkretnego pliku według konwencji projektu.

💡 **Nie masz własnego pomysłu?** Użyj „Code Review" — jest universalny.

---

## Krok 2: Napisz Prompt Ręcznie Pierwszy

Zanim stworzysz skill, napisz prompt ręcznie w Claude Code i przetestuj go.

Przykład dla Code Review:
```
Przejrzyj mój kod pod kątem:
1. Poprawności logiki — czy jest jakiś oczywisty bug?
2. Czytelności — czy nazwy zmiennych/funkcji są opisowe?
3. Bezpieczeństwa — czy są oczywiste luki (SQL injection, hardcoded secrets)?
4. Wydajności — czy jest jakieś oczywiste wąskie gardło?
5. Zgodności z konwencjami z CLAUDE.md

Skoncentruj się na problemach krytycznych i poważnych.
Pomijaj kwestie stylystyczne jeśli nie są istotne.
Format odpowiedzi:
- 🔴 Krytyczne: [lista]
- 🟡 Poważne: [lista]
- 🟢 Drobne: [lista]
- ✅ Co jest dobrze zrobione: [1-2 rzeczy]
```

Przetestuj go na swoim kodzie. Czy wynik jest użyteczny?
Poprawiaj prompt aż będziesz zadowolony.

---

## Krok 3: Stwórz Plik Skilla

### Struktura Katalogów

```bash
# W katalogu projektu (lub katalogu domowym dla globalnych skilli)
mkdir -p .claude/commands
```

Skille globalne (działają w każdym projekcie):
```bash
mkdir -p ~/.claude/commands
```

### Szablon Pliku Skilla

```markdown
# [Nazwa Skilla]

[Opis co skill robi — jedna linia dla siebie, nie jest wysyłany do Claude]

---

[TREŚĆ PROMPTA — to jest wysyłane do Claude]

$ARGUMENTS
```

`$ARGUMENTS` zostaje zastąpione przez wszystko co wpiszesz po nazwie skilla.

### Przykład: `/review`

Stwórz plik `.claude/commands/review.md`:

```markdown
# Code Review

Wykonaj przegląd kodu i przekaż zwięzły raport.

---

Przejrzyj poniższy kod (lub jeśli nie podano — ostatnie zmiany w git diff)
pod kątem:

1. **Poprawności logiki** — czy jest jakiś oczywisty bug?
2. **Czytelności** — czy nazwy zmiennych/funkcji są opisowe?
3. **Bezpieczeństwa** — SQL injection, hardcoded secrets, brak walidacji?
4. **Wydajności** — oczywiste wąskie gardła, N+1 query?
5. **Zgodności z CLAUDE.md** — czy przestrzegane są konwencje projektu?

Skoncentruj się na problemach krytycznych i poważnych.
Nie wymyślaj problemów — jeśli kod jest OK, powiedz to.

Format odpowiedzi:
- 🔴 Krytyczne: [lista lub "brak"]
- 🟡 Poważne: [lista lub "brak"]
- 🟢 Drobne uwagi: [lista lub "brak"]
- ✅ Co jest dobrze zrobione: [1-2 rzeczy]

$ARGUMENTS
```

Użycie:
- `/review` — przejrzy `git diff` (ostatnie zmiany)
- `/review routes/users.py` — przejrzy konkretny plik

---

## Krok 4: Przetestuj Skill

```bash
cd twoj-projekt
claude
```

W sesji Claude Code:
```
/review
```

lub:
```
/review src/main.py
```

**Sprawdź:**
- [ ] Skill się uruchamia (brak błędu "nie znaleziono komendy")
- [ ] Wynik jest w oczekiwanym formacie
- [ ] `$ARGUMENTS` działa (test z argumentem i bez)
- [ ] Skill jest szybszy i wygodniejszy niż ręczny prompt

---

## Krok 5: Iteruj i Ulepszaj

Po kilku użyciach skill można ulepszyć. Typowe problemy:

**Problem: Skill daje za długie odpowiedzi**
Dodaj: „Bądź zwięzły. Maksymalnie 20 linii całkowita odpowiedź."

**Problem: Skill ignoruje konwencje projektu**
Dodaj na początku: „Zanim zaczniesz, przeczytaj CLAUDE.md w tym projekcie."

**Problem: $ARGUMENTS nie działa jak oczekujesz**
Dodaj instrukcję co zrobić gdy brak argumentu:
```
Jeśli podano plik jako argument ($ARGUMENTS), przejrzyj go.
Jeśli nie podano argumentu, przejrzyj `git diff HEAD` (niezacommitowane zmiany).
```

---

## Przykładowe Skille do Zbudowania

### `/setup` — Boilerplate Nowego Projektu

```markdown
# Project Setup

Stwórz strukturę nowego projektu Python.

---

Stwórz kompletny boilerplate projektu Python o nazwie $ARGUMENTS.

Struktura:
- README.md (tytuł, opis, quick start)
- requirements.txt (puste, gotowe do uzupełnienia)
- .gitignore (Python standard)
- src/__init__.py
- src/main.py (punkt wejścia z if __name__ == "__main__")
- tests/__init__.py
- tests/test_main.py (jeden przykładowy test)
- CLAUDE.md (szablon z sekcjami do wypełnienia)

Nie dodawaj żadnych zewnętrznych zależności — tylko Python stdlib.
```

### `/deploy-check` — Lista Przed Wdrożeniem

```markdown
# Deploy Checklist

Weryfikacja przed deploymentem.

---

Sprawdź kod i odpowiedz na poniższą listę kontrolną przed deploymentem.
Dla każdego punktu: ✅ OK / ⚠️ UWAGA / ❌ BLOKUJE deployment.

1. Czy .env lub secrets są w kodzie lub w commitach?
2. Czy wszystkie testy przechodzą? (sprawdź plik testów jeśli jest)
3. Czy są uncommitted zmiany? (git status)
4. Czy requirements.txt jest aktualny?
5. Czy są oczywiste błędy w obsłudze wyjątków?
6. Czy logging jest odpowiedni (nie za dużo, nie za mało)?
7. Czy jest obsługa błędów przy połączeniach zewnętrznych (API, DB)?

Na końcu: łączna ocena GOTOWE / NIE GOTOWE + lista rzeczy do zrobienia.

$ARGUMENTS
```

### `/explain` — Wyjaśnij Kod

```markdown
# Explain Code

Wyjaśnij fragment kodu po polsku.

---

Wyjaśnij poniższy kod ($ARGUMENTS) prostym językiem po polsku.

Struktura wyjaśnienia:
1. Co ten kod robi (1-2 zdania, dla kogoś kto nie zna projektu)
2. Jak to robi (krok po kroku, po 1 zdaniu na krok)
3. Jakie są edge case'y lub ważne założenia

Poziom wyjaśnienia: junior developer z 6 miesiącami doświadczenia.
Unikaj żargonu bez wyjaśnienia.
```

---

## Kryteria Sukcesu ✅

- [ ] Masz przynajmniej jeden działający skill w `.claude/commands/`
- [ ] Skill używa `$ARGUMENTS`
- [ ] Przetestowałeś go z argumentem i bez
- [ ] Skill jest szybszy w użyciu niż pisanie promptu ręcznie
- [ ] Zaktualizowałeś CLAUDE.md z informacją, że skill istnieje

---

## Pro Tip: Organizacja Skilli

Gdy masz >5 skilli, użyj konwencji nazewnictwa:

```
.claude/commands/
├── code-review.md      → /code-review
├── code-explain.md     → /code-explain
├── code-refactor.md    → /code-refactor
├── project-setup.md    → /project-setup
├── project-deploy.md   → /project-deploy
└── debug-bug.md        → /debug-bug
```

Grupowanie prefiksem (`code-`, `project-`, `debug-`) sprawia, że
autocomplete w Claude Code podpowiada powiązane skille razem.

💡 **Najlepsze skille powstają z frustracji.** Gdy po raz trzeci piszesz
ten sam prompt — to znak, że czas na skill.
