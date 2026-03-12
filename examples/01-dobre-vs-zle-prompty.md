# Dobre vs Złe Prompty — Przewodnik z Przykładami

> Różnica między promptem junior a senior developera to różnica między "zrób mi stronę" a precyzyjnym briefem.
> Ten przewodnik pokazuje realne pary promptów i ich efekty.

---

## 1. Tworzenie gry

❌ **Zły prompt:**
```
Zrób mi grę w Flappy Bird
```
Dlaczego zły: Brak kontekstu — jaki język? Jaki framework? Czy ma być AI? Czy terminal czy GUI? Claude zgaduje i produkuje generyczny tutorial z PyGame.

✅ **Dobry prompt:**
```
Zbuduj grę Flappy Bird w Pythonie z PyGame. Wymagania:
- Okno 288x512, ptaszek 34x24px, rury co 200px
- Grawitacja 0.5, skok -8, przerwa między rurami 100px
- Kolizje pixel-perfect z maskami PyGame
- Wynik na ekranie (font: monospace 32px)
- Game over screen z "Press R to restart"
- FPS lock na 60
Nie dodawaj dźwięków ani menu — sam gameplay.
```
Dlaczego działa: Konkretne wymiary, fizyka, mechaniki. Claude produkuje grę, w którą można grać od razu po uruchomieniu.

**Efekt złego:** Generyczny kod z TODO komentarzami, brak kolizji, magiczne liczby.
**Efekt dobrego:** Kompletna, grywalna gra na ~150 linii.

---

## 2. Budowanie API

❌ **Zły prompt:**
```
Napisz mi REST API do zarządzania zadaniami
```
Dlaczego zły: Flask czy FastAPI? Jaka baza? Jakie endpointy? Autentykacja? Claude wybiera za ciebie — i wybiera źle.

✅ **Dobry prompt:**
```
Zbuduj REST API w FastAPI z SQLite (aiosqlite). Endpointy:
- POST /tasks — tworzy zadanie {title: str, priority: 1-5}
- GET /tasks — lista z filtrem ?priority=X i paginacją ?page=1&limit=20
- PATCH /tasks/{id} — update dowolnych pól
- DELETE /tasks/{id} — soft delete (pole deleted_at)

Walidacja Pydantic. Kody błędów: 404 jeśli nie znaleziono, 422 jeśli walidacja.
Baza w pliku tasks.db, auto-create przy starcie.
Nie dodawaj autentykacji — to wewnętrzne narzędzie.
```
Dlaczego działa: Stack, endpointy, walidacja, edge cases — wszystko zdefiniowane. Zero zgadywania.

**Efekt złego:** Flask z in-memory dict, brak walidacji, brak paginacji.
**Efekt dobrego:** Produkcyjne API z testowalnymi endpointami i poprawną obsługą błędów.

---

## 3. Naprawianie buga

❌ **Zły prompt:**
```
Mój trening RL nie działa, napraw to
```
Dlaczego zły: Jaki agent? Jaki błąd? Czy w ogóle odpala? Czy uczy się ale źle? Claude nie ma telepatii.

✅ **Dobry prompt:**
```
Mój agent DQN w flappy-bird-ai uczy się, ale po 50k kroków reward spada.

Konfiguracja:
- DoubleDQN z NoisyNet, CNN (3x conv + 2x fc)
- gamma=0.99, lr=1e-4, batch=64, replay buffer 100k
- reward: +1.0 za rurę, +0.1 za przeżycie, -1.0 za śmierć

Symptomy:
- Q-values rosną do 200+ (podejrzanie wysoko)
- Loss oscyluje między 0.5 a 50
- Epsilon nie jest używany (NoisyNet), ale agent wydaje się eksplorować za mało

Plik: src/rl_agent.py, metoda learn()
Podejrzewam reward shaping — survival reward może dominować.
```
Dlaczego działa: Agent widzi symptomy, konfigurację, podejrzenia. Może od razu sprawdzić matematykę: survival 0.1/(1-0.99) = 10.0 vs pipe 1.0*0.99^avg_steps ≈ 0.51. Bingo — survival dominuje 20x.

**Efekt złego:** Generyczne porady "zwiększ buffer, zmniejsz lr" — nie trafią w problem.
**Efekt dobrego:** Precyzyjna diagnoza + fix reward shapingu w 5 minut.

---

## 4. Refaktoryzacja kodu

❌ **Zły prompt:**
```
Zrefaktoryzuj ten plik, jest brzydki
```
Dlaczego zły: "Brzydki" to nie kryteria. Chcesz rozbić na moduły? Wyciągnąć config? Dodać typy?

✅ **Dobry prompt:**
```
Zrefaktoryzuj src/train.py (430 linii). Cel: rozbić na moduły.

Wyciągnij:
1. Konfigurację → config/train_config.yaml (wszystkie hiperparametry)
2. Logowanie → src/logger.py (TensorBoard + CSV)
3. Checkpointy → src/checkpoint.py (save/load z walidacją)
4. Główna pętla zostaje w train.py ale używa nowych modułów

Zachowaj DOKŁADNIE tę samą logikę — zero zmian w zachowaniu.
Nie zmieniaj nazw zmiennych w pętli treningowej.
Po refaktoryzacji: python -m src.train --config config/train_config.yaml musi dać identyczne wyniki.
```
Dlaczego działa: Jasna struktura docelowa, zasada zero zmian w zachowaniu, konkretny test weryfikacyjny.

**Efekt złego:** Claude przenosi losowe kawałki, zmienia nazwy, coś się psuje.
**Efekt dobrego:** Czysta dekompozycja, ten sam output, łatwiejszy do utrzymania kod.

---

## 5. Pisanie testów

❌ **Zły prompt:**
```
Napisz testy do mojego projektu
```
Dlaczego zły: Jakie testy? Unit? Integration? E2E? Do czego? Jaki framework?

✅ **Dobry prompt:**
```
Napisz testy Playwright (Python) dla strony na localhost:5000.

Testuj:
1. Strona ładuje się w <3s, tytuł zawiera "Narzędzia"
2. Nawigacja: kliknij każdy link w menu, sprawdź że strona się zmienia
3. Dark mode: kliknij toggle, sprawdź że body ma class "dark"
4. Mobile: viewport 375x667, sprawdź że hamburger menu działa
5. Formularz: wypełnij input, kliknij submit, sprawdź response

Konfiguracja:
- Headless Chromium
- Screenshot przy każdym FAIL (zapisz w ./test-screenshots/)
- Timeout: 10s per test
- Na końcu: wypisz PASS/FAIL summary

Plik: test_strona.py, odpalenie: python test_strona.py
```
Dlaczego działa: Framework, scenariusze, konfiguracja, obsługa błędów — kompletna specyfikacja testów.

**Efekt złego:** 3 testy z pytest które importują moduły, których nie masz.
**Efekt dobrego:** 5 scenariuszy E2E ze screenshotami i przejrzystym raportem.

---

## 6. Setup projektu

❌ **Zły prompt:**
```
Stwórz mi projekt Pythonowy
```
Dlaczego zły: Do czego? Jaka struktura? Jakie zależności? Claude tworzy generyczny hello-world.

✅ **Dobry prompt:**
```
Stwórz projekt RL do trenowania agenta grającego w Snake.

Struktura:
├── src/
│   ├── env.py          # środowisko Snake (gym-like API: reset, step, render)
│   ├── agent.py        # DQN agent z replay bufferem
│   ├── train.py        # pętla treningowa z logowaniem
│   └── watch.py        # odtwarzanie zapisanego modelu
├── config/
│   └── default.yaml    # hiperparametry
├── requirements.txt    # torch, numpy, pygame, pyyaml
└── README.md           # jak odpalić trening i watch

Env: plansza 10x10, state = numpy array (10,10,3) — głowa/ciało/jabłko.
Reward: +1 jabłko, -1 ściana/ogon, -0.01 za krok (anty-kręcenie).
Agent: DQN, 3-layer MLP (128,128), epsilon-greedy.
Config YAML: lr, gamma, epsilon_start/end/decay, buffer_size, batch_size.
```
Dlaczego działa: Kompletna specyfikacja od struktury po reward shaping. Claude buduje gotowy-do-trenowania projekt.

**Efekt złego:** Jeden plik z komentarzem "# TODO: implement training loop".
**Efekt dobrego:** 6 plików, spójny projekt, `python -m src.train` startuje od razu.

---

## 7. Debugging ML treningu

❌ **Zły prompt:**
```
Mój model się nie uczy
```
Dlaczego zły: Jaki model? Jakie dane? Co znaczy "nie uczy"? Loss stoi? Spada i wraca? Accuracy 50%?

✅ **Dobry prompt:**
```
SAC agent na custom racing env, po 200k kroków średni reward stoi na -50.

Setup:
- Observation: 1080-dim LIDAR scan (float32)
- Action: continuous [steering, throttle] w [-1, 1]
- Reward: velocity * cos(heading_error) - 0.1 * abs(steering)
- Networks: 2x critic (256,256), policy (256,256), alpha auto-tune

Co widzę:
- Policy entropy spada z 2.0 do 0.3 w 50k kroków (za szybko?)
- Alpha spada do 0.001 (target_entropy = -2.0)
- Q-values: Q1 ≈ Q2 ≈ -48 (bliskie reward, brak discounting?)
- Steering output: mean ≈ 0.0, std ≈ 0.02 (agent się nie rusza)

Podejrzewam: alpha decay za szybki → agent traci eksplorację za wcześnie.
Albo: target_entropy za niskie → wymusza deterministic policy.
```
Dlaczego działa: Metryki, konfiguracja, symptomy, hipotezy. Claude ma wszystko żeby zdiagnozować — i rzeczywiście oba podejrzenia są trafne.

**Efekt złego:** "Spróbuj zwiększyć lr" — generyczna porada, nie pomoże.
**Efekt dobrego:** "Alpha decay ok, ale target_entropy powinno być -dim(action)=-2.0 co jest poprawne. Problem: Q-values ≈ reward sugerują gamma bliskie 0 lub brak bootstrappingu — sprawdź done mask w replay buffer."

---

## 8. Code review

❌ **Zły prompt:**
```
Sprawdź mój kod
```
Dlaczego zły: Cały projekt? Jeden plik? Na co patrzeć — bugi, styl, performance, security?

✅ **Dobry prompt:**
```
Zrób code review pliku src/rl_agent.py pod kątem:
1. Poprawność RL — czy algorytm jest dobrze zaimplementowany
2. Numeryczna stabilność — overflow, underflow, division by zero
3. Performance — niepotrzebne kopie tensorów, brak .no_grad() gdzie powinien być
4. Czytelność — czy inny dev zrozumie za 6 miesięcy

Format odpowiedzi:
- 🔴 KRYTYCZNE: bugi które psują trening
- 🟡 WAŻNE: problemy performance/stabilności
- 🟢 SUGESTIE: czytelność, styl

Nie komentuj importów ani formatowania — tylko logikę.
```
Dlaczego działa: Jasne kryteria, priorytetyzacja, format odpowiedzi. Claude wie na co patrzeć i jak raportować.

**Efekt złego:** Lista 30 nitpicków o PEP8 i nazewnictwie.
**Efekt dobrego:** 3 krytyczne bugi (brak .detach() na target, gradient leak, wrong dim in softmax) + 5 sugestii performance.

---

## Anatomia Doskonałego Prompta

Każdy dobry prompt zawiera 5 elementów:

### 1. Rola — Kim jest Claude w tym momencie?
```
Jesteś senior RL engineer z doświadczeniem w SAC i continuous control.
```
Ustawia "tryb myślenia" — agent RL myśli inaczej niż agent web dev.

### 2. Kontekst — Co już istnieje?
```
Projekt: flappy-bird-ai, Python 3.11, PyTorch 2.1
Pliki: src/rl_agent.py (DQN), src/env.py (PyGame env)
Stan: Agent uczy się ale reward plateau po 50k steps
```
Claude nie musi zgadywać ani pytać — od razu pracuje.

### 3. Zadanie — Co dokładnie zbudować/naprawić?
```
Zmień DQN na DoubleDQN. Dodaj NoisyLinear zamiast epsilon-greedy.
Zachowaj ten sam interfejs: agent.select_action(state), agent.learn(batch).
```
Jedno, jasne zadanie. Nie "ulepsz agenta" — to zbyt otwarte.

### 4. Ograniczenia — Czego NIE robić?
```
Nie zmieniaj env.py. Nie dodawaj PER (Prioritized Replay) — to osobny PR.
Nie zmieniaj struktury replay buffera.
```
Chroni przed "overengineering" — Claude lubi dodawać rzeczy, których nie prosiłeś.

### 5. Weryfikacja — Jak sprawdzić?
```
Po zmianie: python -m src.train powinno startować bez błędów.
Po 10k kroków: loss powinien spadać (< 1.0).
Q-values nie powinny rosnąć powyżej 20.
```
Daje Claude (i tobie) kryteria sukcesu.

---

## 💡 Tip: Reguła 30 sekund

Jeśli pisanie prompta zajmuje ci mniej niż 30 sekund — prawdopodobnie jest za krótki. Dobry prompt to mini-specyfikacja. 2 minuty pisania prompta oszczędza 20 minut poprawiania złego outputu.

## ⚠️ Uwaga: Nie przesadzaj w drugą stronę

Prompt na 500 linii to też problem — Claude gubi się w szczegółach. Sweet spot to 10-30 linii dla typowego zadania. Jeśli potrzebujesz więcej — rozbij na mniejsze taski.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
