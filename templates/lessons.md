# lessons.md — Lekcje z Pracy z Claude Code

> Ten plik to Twoja osobista baza wiedzy o błędach i odkryciach.
> Po każdej sesji, w której coś poszło nie tak lub nauczyłeś się czegoś nowego —
> zapisz to tutaj. Claude czyta ten plik i unika powtarzania tych samych błędów.
>
> Format każdego wpisu:
> ### [Kategoria] Krótka nazwa reguły
> Opis reguły w jednym zdaniu.
> **Dlaczego:** skąd wynika ta reguła (co się stało).
> **Jak stosować:** kiedy i gdzie pamiętać o tej regule.

---

## Workflow

### [Workflow] Zawsze czytaj plik przed edycją
Przed wprowadzeniem zmiany w pliku najpierw go przeczytaj w całości.
**Dlaczego:** Claude nadpisał funkcję `save_user()`, bo nie widział, że niżej
w pliku jest jej przeciążona wersja. Straciliśmy 30 minut na debugowanie.
**Jak stosować:** Każda sesja edycji pliku zaczyna się od Read, nie od Edit.

### [Workflow] Jeden commit = jedna zmiana logiczna
Nie commituj kilku niezależnych zmian w jednym commicie.
**Dlaczego:** Przy rollbacku musieliśmy cofnąć też niezwiązane poprawki,
co wprowadził dodatkowe błędy.
**Jak stosować:** Przed commitem sprawdź `git diff` — jeśli zmiany dotyczą
więcej niż jednej funkcji/modułu, rozdziel je na osobne commity.

---

## Jakość Kodu

### [Jakość] Nie akceptuj „na razie działa"
Jeśli rozwiązanie wygląda hackowo — poproś o eleganckie przepisanie.
**Dlaczego:** Claude dodał globalną zmienną `_temp_fix = True` jako obejście
buga. Zmienna przeżyła trzy sprinty i spowodowała trudny do wykrycia błąd w
produkcji.
**Jak stosować:** Gdy widzisz `# TODO`, `# hack`, `# tymczasowo` — zatrzymaj
się i poproś o właściwe rozwiązanie zanim pójdziesz dalej.

### [Jakość] Proś o testy razem z kodem
Zawsze dodawaj do promptu: „napisz też testy dla tej funkcji".
**Dlaczego:** Kod bez testów był pięciokrotnie częściej źródłem regresji
w projekcie.
**Jak stosować:** Szablon prompta: „Zaimplementuj X. Napisz też testy
jednostkowe/integracyjne weryfikujące przypadki brzegowe."

---

## Architektura

### [Architektura] Zdefiniuj interfejsy przed implementacją
Przed pisaniem kodu uzgodnij format danych między komponentami.
**Dlaczego:** Agent backendowy i agent frontendowy zrobiły różne struktury
JSON (`user_id` vs `userId`). Integracja zajęła dwa razy dłużej niż sama
implementacja.
**Jak stosować:** W promptach dla agentów zawsze podaj przykładowy JSON
lub sygnaturę funkcji jako kontrakt.

---

## Debugowanie

### [Debugowanie] Podaj logi, nie opisy
Wklejaj do prompta dokładny komunikat błędu, nie jego parafrazę.
**Dlaczego:** „Coś nie działa z bazą danych" dało ogólne odpowiedzi.
`sqlalchemy.exc.IntegrityError: UNIQUE constraint failed: user.email`
wskazało od razu na duplikat przy rejestracji.
**Jak stosować:** Zawsze kopiuj pełny stacktrace. Jeśli jest długi —
wklej pierwsze i ostatnie 10 linii.
