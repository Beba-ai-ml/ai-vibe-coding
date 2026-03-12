# /ceo — Budowanie Tetris AI z 6-Agentowym Teamem

> /ceo to pattern orkiestracji, w którym jeden agent (CEO) analizuje zadanie, rozbija je na workstreamy,
> tworzy org chart agentów i koordynuje ich pracę. Zamiast jednego agenta pracującego sekwencyjnie,
> masz zespół pracujący równolegle — jak prawdziwy team developerski.

> Ta sesja dokumentuje realne budowanie tetris-ai — od zera do grającego agenta w jednej sesji.

---

## Zadanie wejściowe

```
Zbuduj Tetris AI, który uczy się grać używając Afterstate V-Learning.

Wymagania:
- Silnik Tetris (7 klocków, rotacje, wall kicks, line clears, scoring)
- Agent RL: V-Learning na afterstate (ocenia planszę PO położeniu klocka)
- Feature extraction: wysokości kolumn, dziury, bumpiness, complete lines
- Pygame wizualizacja w czasie rzeczywistym
- Training pipeline z logowaniem, checkpointami, CLI
- Testy i code review na końcu
```

---

## Faza 1: Analiza CEO

CEO agent dostaje zadanie i robi dekompozycję. Oto co produkuje:

### Analiza zależności

```
Tetris AI — Dependency Graph:

1. SYSTEM DESIGN (Architekt)
   └── definiuje kontrakty API, strukturę plików, interfejsy
       ├── 2. GAME ENGINE (Game Dev) — implementuje silnik Tetris
       ├── 3. AI/RL AGENT (RL Dev) — implementuje V-Learning
       └── 4. RENDERER (Renderer) — implementuje wizualizację
           └── 5. TRAINING PIPELINE (Pipeline Dev) — integruje 2+3+4
               └── 6. QA + REVIEW (Tester) — testuje całość
```

### Org Chart

| # | Rola | Model | Zakres | Zależy od |
|---|------|-------|--------|-----------|
| 1 | Architekt | opus | Design systemu, kontrakty API, file structure | — |
| 2 | Game Engine Dev | sonnet | Silnik Tetris: plansza, klocki, rotacje, scoring | #1 |
| 3 | AI/RL Dev | opus | Afterstate V-Learning, feature extraction, agent | #1 |
| 4 | Renderer | sonnet | PyGame wizualizacja, real-time display | #1 |
| 5 | Training Pipeline Dev | sonnet | Pętla treningowa, CLI, logging, checkpointy | #1, #2, #3 |
| 6 | Tester + Krytyk | opus | Unit testy, integration testy, code review | #2, #3, #4, #5 |

**Decyzje CEO:**
- Architekt na opus — fundamentalne decyzje, musi być precyzyjny
- Game Engine na sonnet — dobrze zdefiniowane zadanie, sonnet wystarczy
- AI/RL na opus — algorytm RL wymaga głębokiego zrozumienia
- Renderer na sonnet — standardowy PyGame, sonnet sobie poradzi
- Pipeline na sonnet — integracja, nie innowacja
- Tester na opus — krytyczny agent, musi znaleźć subtelne bugi

---

## Faza 2: Wykonanie

### Phase A: Architekt

**Prompt wysłany do architekta:**

```
Jesteś system architect. Zaprojektuj Tetris AI z Afterstate V-Learning.

Wyprodukuj:
1. File structure (wszystkie pliki z 1-linijkowym opisem)
2. API kontrakty (klasy, metody, sygnatury z type hints)
3. Data flow diagram (ASCII)

Wymagania:
- Silnik Tetris musi być ODDZIELONY od AI (clean interface)
- Agent dostaje afterstate (planszę PO położeniu klocka) i ocenia
- Feature extraction: heights, holes, bumpiness, complete_lines, max_height
- Renderer to obserwator — nie modyfikuje stanu gry
- Config w YAML, nie hardcode

Format odpowiedzi: jeden markdown z trzema sekcjami.
Nie pisz kodu — tylko design.
```

**Output architekta (skrócony):**

```
## 1. File Structure

tetris-ai/
├── src/
│   ├── __init__.py
│   ├── tetris_engine.py    # silnik gry: Board, Piece, TetrisGame
│   ├── agent.py            # V-Learning agent z afterstate evaluation
│   ├── features.py         # feature extraction z planszy
│   ├── renderer.py         # PyGame vizualizacja
│   ├── train.py            # pętla treningowa
│   └── play.py             # watch trained agent
├── config/
│   └── default.yaml        # wszystkie hiperparametry
├── tests/
│   ├── test_engine.py
│   ├── test_agent.py
│   └── test_features.py
└── requirements.txt

## 2. API Contracts

class Board:
    """10x20 Tetris board (width x height)"""
    def __init__(self) -> None: ...
    def is_valid_position(self, piece: Piece, x: int, y: int) -> bool: ...
    def place_piece(self, piece: Piece, x: int, y: int) -> int: ...  # returns lines cleared
    def get_afterstate(self, piece: Piece, x: int, y: int, rotation: int) -> np.ndarray: ...
    def copy(self) -> 'Board': ...

class Piece:
    """Tetromino piece with rotation"""
    def __init__(self, piece_type: str) -> None: ...  # I, O, T, S, Z, J, L
    def rotate(self, direction: int = 1) -> 'Piece': ...
    def get_cells(self) -> List[Tuple[int, int]]: ...

class TetrisGame:
    """Main game loop manager"""
    def __init__(self, config: dict) -> None: ...
    def get_all_afterstates(self) -> List[Tuple[np.ndarray, Action]]: ...
    def apply_action(self, action: Action) -> Tuple[int, bool]: ...  # (lines, done)

class VLearningAgent:
    """Afterstate V-Learning with linear function approximation"""
    def __init__(self, config: dict) -> None: ...
    def extract_features(self, board: np.ndarray) -> np.ndarray: ...
    def evaluate(self, afterstate: np.ndarray) -> float: ...
    def select_action(self, afterstates: List) -> Action: ...
    def learn(self, features: np.ndarray, reward: float, next_features: np.ndarray, done: bool) -> float: ...

## 3. Data Flow

┌─────────────┐     get_all_afterstates()     ┌──────────┐
│ TetrisGame   │ ──────────────────────────── → │  Agent   │
│ (Board+Piece)│                                │ V-Learn  │
│              │ ← ──────────────────────────── │          │
│              │     apply_action(best_action)  │          │
└──────┬───────┘                                └────┬─────┘
       │ board state                                 │ features
       ↓                                             ↓
┌──────────────┐                              ┌──────────────┐
│   Renderer   │                              │   Features   │
│   (PyGame)   │                              │  Extraction  │
└──────────────┘                              └──────────────┘
```

### Phase B: Równoległe budowanie (3 agenty naraz)

CEO uruchamia trzy agenty równolegle — każdy dostaje output architekta jako kontekst.

**Prompt do Game Engine Dev:**

```
Zaimplementuj silnik Tetris na podstawie tego designu:

[... wklejony output architekta ...]

Pliki do stworzenia:
- src/tetris_engine.py — Board, Piece, TetrisGame

Wymagania:
- 7 standardowych tetromino (I, O, T, S, Z, J, L)
- SRS rotacja (Super Rotation System) z wall kicks
- Line clear: single, double, triple, tetris
- Scoring: 100/300/500/800 za 1/2/3/4 linie
- get_all_afterstates() — kluczowa metoda, zwraca WSZYSTKIE legalne pozycje dla bieżącego klocka
- Plansza 10x20, spawn na górze

Nie implementuj renderera ani agenta — tylko silnik.
Napisz również tests/test_engine.py z testami:
- Rotacja każdego klocka
- Wall kick przy ścianie
- Line clear (single, tetris)
- Game over detection
```

**Prompt do AI/RL Dev:**

```
Zaimplementuj agenta V-Learning na podstawie tego designu:

[... wklejony output architekta ...]

Pliki do stworzenia:
- src/agent.py — VLearningAgent
- src/features.py — feature extraction

Agent Afterstate V-Learning:
- Agent NIE widzi raw board — widzi features extracted z afterstate
- Features (6-dim vector):
  1. aggregate_height — suma wysokości wszystkich kolumn
  2. complete_lines — ile pełnych linii
  3. holes — puste komórki z pełną komórką nad nimi
  4. bumpiness — suma |height[i] - height[i+1]| między kolumnami
  5. max_height — najwyższa kolumna
  6. wells — suma głębokości "studni" (kolumna niższa niż obie sąsiednie)

V-Learning update:
  V(s) ← V(s) + alpha * (reward + gamma * V(s') - V(s))

Funkcja wartości: linear (weights @ features + bias) — CELOWO proste.
Nie używaj sieci neuronowej — linear approximation wystarczy dla Tetris.

select_action:
  Dla każdego afterstate → extract_features → evaluate → wybierz best.
  Epsilon-greedy: z prawdopodobieństwem epsilon wybierz losowy afterstate.
```

**Prompt do Renderer:**

```
Zaimplementuj renderer PyGame na podstawie tego designu:

[... wklejony output architekta ...]

Plik: src/renderer.py

Wymagania:
- Okno: plansza 10x20 bloków, każdy blok 30x30px = 300x600
- Panel boczny 200px: score, lines, level, next piece
- Kolory bloków: standardowe Tetris (cyan=I, yellow=O, purple=T, green=S, red=Z, blue=J, orange=L)
- Ghost piece (przeźroczysty, gdzie klocek wyląduje)
- Animacja line clear (flash 3x)
- FPS: 60, game tick co 500ms (level 1)

Renderer jest OBSERWATOREM — nie zmienia stanu gry.
Interface: renderer.draw(board, current_piece, next_piece, score, lines)
```

### Phase C: Training Pipeline

Po zakończeniu Phase B, CEO uruchamia Pipeline Dev z outputami wszystkich trzech agentów.

**Prompt do Pipeline Dev:**

```
Zintegruj silnik, agenta i renderer w pętlę treningową.

Masz gotowe:
- src/tetris_engine.py (Board, Piece, TetrisGame)
- src/agent.py (VLearningAgent)
- src/features.py (extract_features)
- src/renderer.py (TetrisRenderer)

Stwórz:
- src/train.py — pętla treningowa
- src/play.py — watch trained agent
- config/default.yaml — hiperparametry

train.py:
- CLI args: --config, --episodes, --render (bool), --checkpoint
- Pętla: episode → agent plays game → update V-function → log metrics
- Co 100 epizodów: log avg score, avg lines, epsilon
- Co 1000 epizodów: save checkpoint
- TensorBoard logging: score, lines, epsilon, V-values, loss
- Graceful shutdown: Ctrl+C zapisuje checkpoint

play.py:
- Załaduj checkpoint, graj z render=True, deterministic=True
- Pokaż score i linie na bieżąco

config/default.yaml:
  agent:
    alpha: 0.01
    gamma: 0.99
    epsilon_start: 1.0
    epsilon_end: 0.01
    epsilon_decay: 0.9995
  training:
    episodes: 50000
    log_every: 100
    save_every: 1000
    render: false
```

### Phase D: Tester + Krytyk

CEO uruchamia Tester/Krytyk na opus — przegląda CAŁY kod.

**Prompt do Testera:**

```
Jesteś QA engineer + code reviewer. Przejrzyj cały projekt tetris-ai.

Twoimi celami są:
1. Uruchom istniejące testy (tests/)
2. Napisz brakujące testy:
   - test_features.py — czy features są poprawnie liczone
   - test_agent.py — czy V-Learning update jest poprawny matematycznie
3. Code review — szukaj:
   - Bugi logiczne (szczególnie w get_all_afterstates i wall kicks)
   - Off-by-one errors (indeksy planszy 0-9 x 0-19)
   - Brakujące edge cases (game over na spawn, I-piece rotacja przy suficie)
   - Performance bottlenecks (get_all_afterstates musi być szybkie)
   - Consistency — czy API kontrakty z designu są zachowane

Format raportu:
🔴 KRYTYCZNE: bugi które psują grę/trening
🟡 WAŻNE: problemy które wpływają na jakość
🟢 SUGESTIE: usprawnienia
Na końcu: lista fixów do zastosowania.
```

---

## Faza 3: Wyniki

### Co zostało zbudowane

```
tetris-ai/
├── src/
│   ├── tetris_engine.py    # 280 linii — pełny silnik z SRS
│   ├── agent.py            # 95 linii — V-Learning agent
│   ├── features.py         # 65 linii — 6 features
│   ├── renderer.py         # 180 linii — PyGame viz
│   ├── train.py            # 150 linii — training loop + CLI
│   └── play.py             # 45 linii — watch mode
├── config/
│   └── default.yaml        # 25 linii
├── tests/
│   ├── test_engine.py      # 120 linii — 15 testów
│   ├── test_agent.py       # 60 linii — 8 testów
│   └── test_features.py    # 80 linii — 10 testów
└── requirements.txt
Total: ~1100 linii kodu + ~260 linii testów
```

### Statystyki treningu

Po 50,000 epizodów:
- **Średni score:** 12,400 punktów
- **Średnie lines cleared:** 1,766
- **Najlepszy epizod:** 4,200 linii
- **Czas treningu:** ~45 min na CPU (linear approx jest szybki)

### Co poszło dobrze

1. **Równoległa praca** — Game Engine, Agent i Renderer budowane jednocześnie. Zamiast ~45 min sekwencyjnie, Phase B trwała ~15 min (najdłuższy agent).

2. **Architekt jako single source of truth** — Wszystkie agenty pracowały z tymi samymi API kontraktami. Zero konfliktów przy integracji.

3. **Tester złapał 3 bugi:**
   - `get_all_afterstates()` nie uwzględniał I-piece w pionowej rotacji przy prawej ścianie
   - `holes` feature liczył źle gdy klocek miał "dziurę" w środku (T-piece)
   - `renderer.draw()` crashował gdy `next_piece` był None (koniec gry)

4. **Linear approximation wystarczył** — Nie trzeba było sieci neuronowej. V-Learning z 6 features i liniową funkcją wartości nauczył się grać na poziomie ~1,700 linii.

### Co wymagało interwencji człowieka

1. **Architektura afterstate** — Architekt początkowo zaproponował standard Q-Learning (state → action). Trzeba było skorygować: Tetris to afterstate problem (oceniasz plansza PO położeniu klocka, nie PRZED).

2. **Reward tuning** — Pierwszy reward: +lines_cleared. Agent uczył się czyścić 1 linię zamiast czekać na tetris. Fix: reward = lines_cleared^2 (1→1, 2→4, 3→9, 4→16). Zachęca do czekania na tetris.

3. **Feature weights inicjalizacja** — Losowe wagi na starcie = agent kładzie klocki na górze. Fix: ręczna inicjalizacja (holes=-5.0, height=-1.0, bumpiness=-1.0, lines=+3.0) jako starting point, potem agent fine-tune'uje.

---

## Komentarz: Decyzje CEO

### Dlaczego Architekt pierwszy?

Bez wspólnego designu agenty produkują niekompatybilny kod. Game Engine mógłby zwracać board jako `list[list[int]]`, a Agent oczekiwać `np.ndarray`. Architekt wymusza kontrakt.

### Dlaczego opus dla AI/RL?

Afterstate V-Learning to mniej popularny algorytm. Sonnet mógłby zaimplementować standard Q-Learning i "przerobić" na afterstate, ale opus rozumie niuanse: dlaczego afterstate jest lepszy dla Tetris (deterministyczne przejścia), jak prawidłowo zaimplementować update rule, kiedy bootstrap a kiedy Monte Carlo.

### Dlaczego Tester na końcu, nie w trakcie?

CEO mógłby uruchomić Testera równolegle z Phase B, testując każdy komponent osobno. Ale najcenniejsze bugi to **bugi integracyjne** — a te wymagają kompletnego kodu. Dlatego Tester czeka na Phase C.

### Kiedy /ceo jest overkill?

- Mały feature (< 100 linii) — nie potrzebujesz 6 agentów
- Dobrze zdefiniowane zadanie — jeden agent wystarczy
- Brak równoległości — jeśli każdy krok zależy od poprzedniego, CEO nie przyspiesza

### Kiedy /ceo błyszczy?

- Nowy projekt od zera (jak ten Tetris)
- Dużo niezależnych komponentów (silnik + AI + renderer + testy)
- Potrzebujesz code review na końcu (dedykowany Tester agent)
- Czas jest kluczowy — równoległość skraca czas 2-3x

---

## 💡 Tip: Prompt do Architekta to fundament

Jeśli Architekt wyprodukuje słaby design — WSZYSTKIE kolejne agenty będą cierpieć. Poświęć czas na dobry prompt do Architekta. Zdefiniuj interfejsy, data flow, ograniczenia. To inwestycja, która zwraca się wielokrotnie.

## 💡 Tip: CEO nie musi być automatyczny

W tym przykładzie CEO to agent, ale możesz być CEO sam — ręcznie uruchamiasz agenty, przekazujesz outputy, decydujesz o kolejności. Ważny jest PATTERN (dekompozycja → równoległość → integracja → review), nie automatyzacja.

## ⚠️ Uwaga: Kontekst się gubi

Każdy agent w /ceo ma ograniczone okno kontekstowe. Nie wrzucaj im pełnego kodu poprzednich agentów — wrzucaj API kontrakty i interfejsy. Agent potrzebuje wiedzieć CO dostaje i CO zwraca, nie JAK to jest zaimplementowane wewnątrz.

## ⚠️ Uwaga: Koordynacja kosztuje

6 agentów opus to sporo tokenów. Dla prostych projektów koszt /ceo przewyższy zysk z równoległości. Używaj dla projektów > 500 linii z jasnymi komponentami.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
