# Ćwiczenie 2: Napisz CLAUDE.md dla Swojego Projektu

**Poziom:** Początkujący–Średni
**Czas:** 45–90 minut
**Cel:** Stworzyć plik CLAUDE.md, który sprawia, że Claude rozumie Twój projekt
i zachowuje się jak senior developer z doświadczeniem właśnie w tym kodzie.

---

## Czego się nauczysz

- Jak CLAUDE.md zmienia jakość odpowiedzi Claude Code
- Jakie informacje są naprawdę potrzebne (a jakie są szumem)
- Jak iterować CLAUDE.md na podstawie obserwowanego zachowania Claude

---

## Przygotowanie

Wybierz projekt, na którym będziesz pracować. Opcje:

**Opcja A — Istniejący projekt (preferowana)**
Użyj projektu z Ćwiczenia 1 lub dowolnego projektu, który już masz.
Korzyść: od razu zobaczysz różnicę przed/po.

**Opcja B — Nowy projekt**
```bash
mkdir moj-projekt && cd moj-projekt && claude
```
Poproś Claude o stworzenie prostej aplikacji Flask z 2-3 trasami.

---

## Krok 1: Zrób Baseline Test (Bez CLAUDE.md)

Zanim napiszesz CLAUDE.md, przetestuj jak Claude radzi sobie BEZ niego.
Zapisz odpowiedzi, żeby później porównać.

Otwórz Claude Code w katalogu projektu i zadaj te trzy pytania:

**Test 1 — Zrozumienie projektu:**
```
Opisz krótko ten projekt: co robi, jaka jest jego architektura,
jakich technologii używa?
```

**Test 2 — Dodanie funkcji:**
```
Dodaj możliwość filtrowania listy po dacie. Jak byś to zaimplementował?
```

**Test 3 — Konwencje:**
```
Jak powinnam nazwać nową funkcję, która pobiera użytkownika po emailu?
```

Zapisz odpowiedzi lub zrób screenshoty. Wrócimy do nich w Kroku 4.

---

## Krok 2: Wypełnij Szablon CLAUDE.md

Skopiuj plik `templates/CLAUDE.md` do katalogu projektu:

```bash
cp /ścieżka/do/kursu/templates/CLAUDE.md ./CLAUDE.md
```

Teraz wypełnij każdą sekcję. Poniżej wskazówki do każdej z nich:

### Przegląd Projektu
Napisz 2-3 zdania odpowiadające na: co robi, dla kogo, co jest w nim
wyjątkowego lub trudnego. Unikaj ogólników.

❌ Źle: „Aplikacja webowa"
✅ Dobrze: „API do zarządzania zadaniami dla małych zespołów. Każde zadanie
ma przypisanego właściciela i priorytet. Brak frontendu — tylko REST API
konsumowane przez aplikację mobilną."

### Tech Stack
Bądź konkretny co do wersji. Claude zachowuje się inaczej dla Flask 2 i Flask 3.

### Kluczowe Pliki
Wymień maksymalnie 6-8 plików. Więcej = szum. Wybierz te, które Claude
będzie najczęściej edytować lub które są krytyczne dla architektury.

### Jak Uruchomić
Wklej dosłownie komendy, które sam używasz. Przetestuj je od nowa w pustym
terminalu żeby upewnić się, że działają.

### Konwencje
To jest najważniejsza sekcja. Przykłady konkretnych konwencji:
- „Odpowiedzi API zawsze mają strukturę `{"data": ..., "error": null}`"
- „Trasy w blueprintach, nie w `app.py`"
- „Nazwy testów zaczynają się od `test_` i opisują zachowanie: `test_returns_404_when_user_not_found`"

### Czego NIE Robić
Zapisuj tutaj błędy, które Claude już popełnił w Twoim projekcie.
Ta sekcja rośnie z czasem — to normalne.

---

## Krok 3: Zweryfikuj CLAUDE.md Razem z Claude

Po napisaniu CLAUDE.md, poproś Claude o ocenę:

```
Przeczytaj plik CLAUDE.md w tym projekcie i powiedz mi:
1. Czy jest coś niejasnego lub sprzecznego?
2. Jakiej informacji brakuje, żebyś mógł efektywnie pracować z tym projektem?
3. Czy sekcja "Czego NIE Robić" jest wystarczająco konkretna?
```

Popraw na podstawie feedbacku.

---

## Krok 4: Powtórz Baseline Test (Z CLAUDE.md)

Zadaj dokładnie te same trzy pytania co w Kroku 1.

**Porównaj odpowiedzi:**

| Aspekt | Bez CLAUDE.md | Z CLAUDE.md |
|--------|---------------|-------------|
| Trafność opisu projektu | | |
| Zgodność z konwencjami | | |
| Jakość propozycji implementacji | | |

---

## Krok 5: Dodaj Pierwszą Lekcję

Po sesji pracy z projektem, gdy Claude coś zrobi źle — zapisz to.

Otwórz (lub stwórz) plik `lessons.md` w katalogu projektu i dodaj wpis:

```markdown
### [Kategoria] Co się stało i jak zapobiec

Claude [opisz błąd].
**Dlaczego:** [co spowodowało ten błąd].
**Jak stosować:** [kiedy/gdzie pamiętać o tej regule — dodaj też do CLAUDE.md].
```

💡 Przenieś regułę zarówno do `lessons.md` (dla siebie), jak i do sekcji
„Czego NIE Robić" w `CLAUDE.md` (dla Claude).

---

## Lista Kontrolna Jakości CLAUDE.md ✅

Oceń swój plik odpowiadając tak/nie na poniższe pytania:

- [ ] Czy ktoś obcy mógłby uruchomić projekt TYLKO na podstawie sekcji „Jak Uruchomić"?
- [ ] Czy konwencje są wystarczająco konkretne żeby Claude mógł je zastosować bez pytania?
- [ ] Czy „Kluczowe Pliki" zawierają maksymalnie 8 pozycji?
- [ ] Czy sekcja „Czego NIE Robić" zawiera przynajmniej 3 konkretne zakazy?
- [ ] Czy podałeś dokładne wersje (nie „Flask", ale „Flask 3.0")?
- [ ] Czy plik jest krótszy niż 80 linii? (dłuższe = Claude ignoruje fragmenty)
- [ ] Czy po przeczytaniu pliku można odpowiedzieć: „Jaka jest odpowiedzialność pliku `X`"?
- [ ] Czy po przeczytaniu pliku można odpowiedzieć: „Jak powinien wyglądać błąd API 404"?
- [ ] Czy sekcja przeglądu odpowiada na pytanie: co to robi i dla kogo?
- [ ] Czy plik nie zawiera żadnych placeholder'ów `[WYPEŁNIJ]`?

**Wynik:**
- 9-10 ✅ — Doskonały CLAUDE.md, gotowy do pracy
- 7-8 ✅ — Dobry, popraw brakujące punkty
- 5-6 ✅ — Wymaga istotnych uzupełnień
- <5 ✅ — Zacznij od nowa z większą dokładnością

---

## Wskazówki Końcowe

⚠️ **CLAUDE.md to żywy dokument.** Nie piszesz go raz i zapominasz.
Aktualizuj go gdy:
- Zmieniasz architekturę
- Odkrywasz, że Claude robi coś źle
- Dodajesz nową technologię do stacku

💡 **Krótki i konkretny bije długi i ogólny.** Plik 40 linii, który Claude
w całości rozumie, jest lepszy niż 200-liniowy elaborat.

💡 **Testuj CLAUDE.md jak testujesz kod.** Jeśli Claude nadal robi błędy,
które opisujesz w pliku — to znak, że opis jest niewystarczająco jasny.
