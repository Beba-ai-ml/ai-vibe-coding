# Ćwiczenie 1: Zbuduj Grę Jednym Promptem

**Poziom:** Początkujący
**Czas:** 30–60 minut
**Cel:** Zbudować działającą grę w jednej sesji Claude Code, używając jednego głównego prompta.

---

## Czego się nauczysz

- Jak napisać skuteczny prompt dla Claude Code
- Jak Claude Code generuje kompletne projekty od zera
- Jak iterować na wynikach za pomocą follow-up promptów

---

## Wybierz Grę

Wybierz jedną z czterech gier:

| Gra | Trudność | Stack |
|-----|----------|-------|
| **Snake** | ⭐ łatwa | HTML5 Canvas + JS |
| **Pong** | ⭐⭐ średnia | HTML5 Canvas + JS |
| **Breakout** | ⭐⭐ średnia | HTML5 Canvas + JS |
| **Space Invaders** | ⭐⭐⭐ trudna | HTML5 Canvas + JS |

💡 **Rekomendacja dla pierwszego razu:** Snake — prosta mechanika, widoczny efekt.

---

## Krok 1: Przygotuj Katalog

```bash
mkdir moja-gra
cd moja-gra
claude
```

---

## Krok 2: Napisz Główny Prompt

Skopiuj poniższy szablon, uzupełnij `[POLA W NAWIASACH]` i wklej do Claude Code:

```
Zbuduj grę [NAZWA GRY] w przeglądarce.

Wymagania:
- Jeden plik HTML z osadzonym CSS i JavaScript (zero zależności zewnętrznych)
- Płynna animacja 60 FPS
- Sterowanie: [OPISZ STEROWANIE — np. "strzałki klawiatury" / "WASD" / "mysz"]
- System punktacji widoczny na ekranie
- Ekran startowy z instrukcją sterowania
- Ekran końca gry (game over) z wynikiem i opcją restart
- Responsywny design — działa na ekranach od 400px do 1200px szerokości

Dodatkowe szczegóły:
- [OPCJONALNIE: dodaj coś specyficznego, np. "snake zaczyna z długością 3" / "piłka przyspiesza co 5 punktów"]

Proszę stwórz kompletny, gotowy do uruchomienia plik index.html.
```

**Przykład wypełnionego promptu dla Snake:**
```
Zbuduj grę Snake w przeglądarce.

Wymagania:
- Jeden plik HTML z osadzonym CSS i JavaScript (zero zależności zewnętrznych)
- Płynna animacja 60 FPS
- Sterowanie: strzałki klawiatury
- System punktacji widoczny na ekranie
- Ekran startowy z instrukcją sterowania
- Ekran końca gry (game over) z wynikiem i opcją restart
- Responsywny design — działa na ekranach od 400px do 1200px szerokości

Dodatkowe szczegóły:
- Wąż zaczyna z długością 3
- Prędkość rośnie co 10 punktów
- Tło w stylu retro (ciemne, neonowe kolory)

Proszę stwórz kompletny, gotowy do uruchomienia plik index.html.
```

---

## Krok 3: Uruchom i Przetestuj

Po wygenerowaniu pliku przez Claude Code:

```bash
# Opcja A: Otwórz bezpośrednio w przeglądarce
xdg-open index.html       # Linux
open index.html            # macOS
start index.html           # Windows

# Opcja B: Lokalny serwer (lepszy do debugowania)
python3 -m http.server 8080
# → otwórz http://localhost:8080
```

**Przetestuj:**
- [ ] Gra otwiera się bez błędów w konsoli (F12 → Console)
- [ ] Można wystartować grę
- [ ] Sterowanie działa
- [ ] Punkty się liczą
- [ ] Gra kończy się (game over)
- [ ] Można zagrać ponownie

---

## Krok 4: Iteruj z Follow-Up Promptami

Jeśli coś nie działa lub chcesz poprawki, użyj follow-up promptów:

**Naprawa buga:**
```
Gra działa, ale [OPISZ PROBLEM]. Proszę napraw to.
```

**Przykład:**
```
Gra działa, ale wąż może zawrócić o 180 stopni w jednej klatce
(np. idzie w prawo i można natychmiast nacisnąć w lewo).
Proszę napraw to — ruch w przeciwnym kierunku powinien być ignorowany.
```

**Zmiana wizualna:**
```
Zmień wygląd gry: [OPISZ ZMIANĘ]. Zachowaj całą logikę bez zmian.
```

**Poprawa UX:**
```
Dodaj krótkie opóźnienie 1 sekundy po naciśnięciu "Restart"
zanim gra się zacznie, żeby gracz zdążył zabrać palec z klawiatury.
```

---

## Kryteria Sukcesu ✅

Ćwiczenie jest zaliczone gdy:
- [ ] Gra uruchamia się w przeglądarce bez błędów
- [ ] Jest grywalna (można przegrać i wygrywać)
- [ ] Wyświetla wynik
- [ ] Ma ekran game over z możliwością restartu

---

## Bonus: Dodaj Funkcję

Wybierz jedno z poniższych rozszerzeń i poproś Claude Code o implementację:

**Poziom trudności:**
```
Dodaj trzy poziomy trudności (łatwy / normalny / trudny),
wybierane na ekranie startowym. Różnią się prędkością gry.
```

**Najlepszy wynik:**
```
Dodaj system high score zapisywany w localStorage przeglądarki.
Wyświetl top 5 wyników na ekranie startowym.
```

**Dźwięk (bez zewnętrznych plików):**
```
Dodaj efekty dźwiękowe używając Web Audio API (bez zewnętrznych plików).
Dźwięk przy jedzeniu, inny przy game over.
```

---

## Podsumowanie

Po ćwiczeniu odpowiedz sobie na pytania:
1. Ile linii kodu wygenerował Claude Code? Czy przejrzałeś je?
2. Który prompt był najskuteczniejszy — główny czy follow-up?
3. Co byś zmienił w promptach przy następnej grze?

💡 **Wskazówka na przyszłość:** Im dokładniejszy opis wymagań w pierwszym prompcie, tym mniej iteracji potrzebujesz. Czas spędzony na pisaniu dobrego prompta to oszczędność czasu na debugowaniu.
