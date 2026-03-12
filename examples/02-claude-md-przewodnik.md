# CLAUDE.md — Kompletny Przewodnik

> CLAUDE.md to najważniejszy plik w twoim projekcie kiedy pracujesz z Claude Code.
> To "briefing" który agent czyta PRZED każdą interakcją. Dobry CLAUDE.md = agent który rozumie twój projekt od pierwszego prompta.

---

## Czym jest CLAUDE.md?

CLAUDE.md to plik markdown w katalogu głównym projektu, który Claude Code automatycznie wczytuje przy każdej sesji. Zawiera:

- Opis projektu i jego cel
- Stack technologiczny
- Kluczowe pliki i ich funkcje
- Konwencje kodu
- Jak uruchomić, jak testować
- Czego NIE robić

Claude traktuje ten plik jak **instrukcję operacyjną**. Bez niego — zgaduje. Z nim — wie.

---

## Pełny przykład z komentarzami

Oto prawdziwy CLAUDE.md dla projektu RL (wyścigi samochodów autonomicznych):

```markdown
# Occupancy Racer — SAC Agent for Autonomous Racing

## Project Overview
# ☝️ Jedno zdanie — co to jest i po co istnieje
RL agent learning to race on occupancy grid maps using Soft Actor-Critic.
Input: 1080-dim LIDAR scan. Output: continuous [steering, throttle].

## Tech Stack
# ☝️ Wersje są ważne — Claude nie zgaduje czy użyć torch.compile()
- Python 3.11, PyTorch 2.1
- Custom Gym environment (no gymnasium dependency)
- TensorBoard for logging
- YAML configs in config/

## Key Files
# ☝️ Nie WSZYSTKIE pliki — tylko te krytyczne. Agent potrzebuje mapy, nie encyklopedii.
- src/sac_agent.py — SAC implementation (twin critics, auto-alpha)
- src/racer_env.py — racing environment (reset, step, render)
- src/train.py — training loop with logging and checkpoints
- src/networks.py — policy and critic network architectures
- config/default.yaml — hyperparameters (DO NOT hardcode values)

## How to Run
# ☝️ Dokładne komendy — copy-paste ready
source /home/beba/aiaie/rl-env/bin/activate
python -m src.train --config config/default.yaml

# Watch trained agent:
python -m src.watch --checkpoint runs/best_model.pth

## Conventions
# ☝️ Reguły, których Claude MUSI przestrzegać
- All hyperparameters in YAML config — never hardcode in source files
- Type hints on all function signatures
- Docstrings on classes, not individual methods (unless complex)
- Logging via TensorBoard — no print() for metrics

## What NOT to Do
# ☝️ Sekcja krytyczna — powstała z bolesnych doświadczeń
- Do NOT modify racer_env.py without explicit permission
- Do NOT add gymnasium as dependency (custom env on purpose)
- Do NOT change reward function without discussing tradeoffs first
- Do NOT use global variables for config
- Do NOT commit model weights (.pth files)

## Testing
# ☝️ Jak weryfikować zmiany
python -m pytest tests/ -v
# Quick sanity check (10 episodes, no training):
python -m src.watch --checkpoint runs/latest.pth --episodes 10
```

---

## Co wrzucać do CLAUDE.md

| Sekcja | Priorytet | Przykład |
|--------|-----------|---------|
| Opis projektu | 🔴 Obowiązkowy | "Tetris AI z Afterstate V-Learning" |
| Stack + wersje | 🔴 Obowiązkowy | "Python 3.11, PyTorch 2.1, PyGame 2.5" |
| Kluczowe pliki | 🔴 Obowiązkowy | "src/agent.py — główny agent RL" |
| Jak uruchomić | 🔴 Obowiązkowy | "python -m src.train" |
| Konwencje | 🟡 Ważny | "Config w YAML, nie hardcode" |
| Czego nie robić | 🟡 Ważny | "Nie modyfikuj env bez pytania" |
| Jak testować | 🟡 Ważny | "pytest tests/ -v" |
| Architektura | 🟢 Opcjonalny | "Agent → Env → Replay Buffer → Train" |
| Historia decyzji | 🟢 Opcjonalny | "Przeszliśmy z DQN na SAC bo continuous action space" |

## Czego NIE wrzucać

- **Cały kod źródłowy** — Claude i tak czyta pliki, nie potrzebuje kopii w CLAUDE.md
- **Eseje o architekturze** — 5 zdań wystarczy, nie 5 akapitów
- **Oczywistości** — "Python to język programowania" — nie
- **Nieaktualne informacje** — stary CLAUDE.md jest gorszy niż brak CLAUDE.md
- **Sekrety** — klucze API, hasła, tokeny. Nigdy.

---

## Ewolucja CLAUDE.md — od Dnia 1 do Miesiąca 1

### Dzień 1: Minimum Viable CLAUDE.md (~10 linii)

```markdown
# Snake DQN

Python 3.11, PyTorch 2.1, PyGame for rendering.

## Key Files
- src/env.py — Snake environment
- src/agent.py — DQN agent
- src/train.py — training loop

## Run
python -m src.train
```

To wystarczy na start. Claude wie co to za projekt i jak go odpalić.

### Tydzień 1: Dojrzały CLAUDE.md (~30 linii)

```markdown
# Snake DQN

DQN agent learning to play Snake on a 10x10 grid.
State: (10,10,3) numpy array — head/body/food channels.
Action: discrete {UP, DOWN, LEFT, RIGHT}.

## Stack
Python 3.11, PyTorch 2.1, PyGame 2.5, NumPy, PyYAML

## Key Files
- src/env.py — Gym-like Snake (reset/step/render)
- src/agent.py — DoubleDQN + NoisyLinear
- src/replay_buffer.py — uniform replay buffer (100k)
- src/train.py — training loop + TensorBoard logging
- config/default.yaml — all hyperparameters

## Run
python -m src.train --config config/default.yaml
python -m src.watch --checkpoint runs/best.pth

## Conventions
- Hyperparameters in YAML only
- Type hints everywhere
- No print() for metrics — use TensorBoard

## Do NOT
- Change observation shape without updating agent input layer
- Hardcode reward values — they're in config
```

Dodane: observation space, konwencje, ograniczenia wynikające z doświadczenia.

### Miesiąc 1: Produkcyjny CLAUDE.md (~60+ linii)

Wszystko powyżej plus:
- Sekcja "Known Issues" (np. "NoisyNet warmup biases exploration for first 1k steps")
- Sekcja "Architecture Decisions" (np. "DoubleDQN zamiast vanilla DQN — patrz commit abc123")
- Sekcja "Performance Notes" (np. "Training 100k steps ≈ 45 min na RTX 3060")
- Linki do Tasks.md dla bieżących priorytetów

---

## Szablony

### Minimalny starter (kopiuj-wklej)

```markdown
# [Nazwa Projektu]

[Jedno zdanie opisu]

## Stack
[Język, framework, wersje]

## Key Files
- [plik] — [co robi]

## Run
[komenda]
```

### Pełny produkcyjny szablon

```markdown
# [Nazwa Projektu]

## Overview
[2-3 zdania: co, po co, jak]

## Stack
- [Język + wersja]
- [Framework + wersja]
- [Kluczowe zależności]

## Architecture
[Krótki opis flow: Input → Processing → Output]

## Key Files
- [plik 1] — [opis]
- [plik 2] — [opis]
- [plik 3] — [opis]

## How to Run
[Dokładne komendy, copy-paste ready]

## How to Test
[Komenda testowa + co oznacza sukces]

## Conventions
- [Reguła 1]
- [Reguła 2]
- [Reguła 3]

## Do NOT
- [Zakaz 1 — i dlaczego]
- [Zakaz 2 — i dlaczego]

## Known Issues
- [Issue 1]

## Performance Notes
- [Benchmark / czas treningu / limity]
```

---

## Folder `.context/` — Rozszerzony Kontekst

Dla większych projektów sam CLAUDE.md nie wystarczy. Folder `.context/` trzyma dodatkowe pliki wiedzy:

### INDEX.md — Punkt wejścia
```markdown
# Snake DQN — Context Index

- KNOWLEDGE.md — architektura, stack, konwencje (rzadko się zmienia)
- STATE.md — bieżący stan, WIP, ostatnie zmiany (aktualizowany często)

Last updated: 2026-03-10
```

### KNOWLEDGE.md — Wiedza statyczna
```markdown
# Knowledge Base

## Architecture
DQN agent z DoubleDQN i NoisyLinear. 3-layer MLP (128,128,4).
Replay buffer uniform 100k. Target network sync co 1000 kroków.

## Key Files
- src/agent.py — cały agent: sieci, select_action, learn
- src/env.py — środowisko Snake, Gym-like API
- config/default.yaml — wszystkie hiperparametry

## Tech Stack
Python 3.11, PyTorch 2.1, PyGame 2.5

## How to Run
python -m src.train --config config/default.yaml

## Conventions
- Config w YAML, nigdy hardcode
- Type hints obowiązkowe
- TensorBoard do logowania metryk
```

### STATE.md — Stan dynamiczny
```markdown
# Current State

## What Works
- Training loop działa, agent uczy się do ~50 score
- TensorBoard logging kompletny
- Watch mode renderuje gameplay

## Work in Progress
- Migracja z DQN na SAC (branch: feature/sac)
- PER (Prioritized Experience Replay)

## Recent Changes
- 2026-03-08: Dodano NoisyLinear (zastąpił epsilon-greedy)
- 2026-03-05: Fix reward shaping — zmniejszono survival reward z 0.1 na 0.01

## Known Issues
- Watch mode nie przekazuje deterministic=True — agent eksploruje podczas ewaluacji
- Gradient clipping za niski dla CNN wariantu (1.0 vs normy 5-15)

## Next Steps
1. Dokończyć SAC branch
2. Dodać PER
3. CNN policy zamiast MLP
```

---

## `lessons.md` — Uczenie się z Błędów

Plik `lessons.md` (w katalogu domowym lub projektu) to baza wiedzy o błędach, które popełniłeś ty i agent. Claude czyta go i **nie powtarza tych samych pomyłek**.

### Format wpisu

```markdown
## [Data] Tytuł problemu
**Kontekst:** Co robiłeś
**Błąd:** Co poszło nie tak
**Fix:** Co zadziałało
**Reguła:** Zasada na przyszłość
```

### 5 prawdziwych przykładów

```markdown
## 2026-01-15 Survival reward dominuje task reward
**Kontekst:** Flappy Bird DQN, reward +0.1 za krok przeżycia
**Błąd:** Agent nauczył się latać w miejscu — survival reward z gamma=0.99 daje Q-contribution 10.0, pipe reward tylko 0.51
**Fix:** Zmniejszono survival z 0.1 na 0.01 (Q-contribution 1.0)
**Reguła:** Zawsze licz effective Q-contribution każdego reward component: reward/(1-gamma) dla continuing tasks

## 2026-02-03 NoisyNet biased exploration na starcie
**Kontekst:** Zamiana epsilon-greedy na NoisyLinear w DQN
**Błąd:** Wszystkie sigma zainicjalizowane tą samą wartością → agent preferuje te same akcje na początku
**Fix:** Dodano warmup period 5k kroków z uniform random actions
**Reguła:** NoisyNet wymaga warmup — nie ufaj eksploracji w pierwszych krokach

## 2026-02-10 select_action psuje eval mode
**Kontekst:** Funkcja select_action() kończy się na model.train()
**Błąd:** Nawet w bloku with torch.no_grad() po wywołaniu select_action model jest w train mode
**Fix:** select_action nie zmienia mode — caller decyduje
**Reguła:** Funkcje narzędziowe nie powinny mieć side effects na model state

## 2026-02-20 git amend po failed hook niszczy historię
**Kontekst:** Pre-commit hook failuje, próbuję --amend
**Błąd:** --amend modyfikuje POPRZEDNI commit (bo nowy nie powstał), tracę zmiany
**Fix:** Po failed hook: napraw → git add → NOWY commit (nie --amend)
**Reguła:** NIGDY git commit --amend po failed pre-commit hook

## 2026-03-01 Subagent bez kontekstu marnuje czas
**Kontekst:** Odpalono subagenta do fixowania buga bez podania stack/plików/symptomów
**Błąd:** Agent spędził 5 minut czytając cały projekt zanim zaczął pracę
**Fix:** Podano w prompcie: plik, linia, symptom, podejrzenie
**Reguła:** Subagent dostaje MINIMUM: plik + opis problemu + stack. Nigdy "napraw to" bez kontekstu.
```

---

## 💡 Tip: CLAUDE.md to żywy dokument

Aktualizuj go po KAŻDEJ sesji, w której Claude zrobił coś głupiego z powodu braku kontekstu. Dodaj regułę do "Do NOT" lub nowy wpis do "Conventions".

## ⚠️ Uwaga: Hierarchia plików

Claude Code czyta pliki w kolejności:
1. `~/.claude/CLAUDE.md` — globalne reguły (dla wszystkich projektów)
2. `./CLAUDE.md` — reguły projektu
3. `./.context/` — rozszerzony kontekst

Globalne reguły NIE powinny zawierać specyfiki projektów. Projektowe reguły NIE powinny powtarzać globalnych.

## ⚠️ Uwaga: Nie kopiuj tutoriala

CLAUDE.md to NIE README dla ludzi. To instrukcja dla agenta. Pisz jak brief dla kolegi z zespołu, nie jak dokumentację open-source.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
