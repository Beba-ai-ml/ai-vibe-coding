# /krytyk — Sesja Adversarial Code Review

> /krytyk to pattern, w którym dwóch niezależnych agentów robi code review z różnych perspektyw.
> Krytyk 1 patrzy na **teorię** (algorytmy, matematykę, RL correctness).
> Krytyk 2 patrzy na **inżynierię** (bugi, edge cases, performance).
> Potem szukają **konsensusu** — i dopiero wtedy wdrażasz fixy.

> Ta sesja jest oparta na prawdziwych bugach znalezionych w projekcie flappy-bird-ai.
> Każdy z tych bugów był realny i wpływał na trening.

---

## Setup

**Projekt:** `flappy-bird-ai` — DQN agent uczący się grać w Flappy Bird.

**Stack:** Python 3.11, PyTorch 2.1, PyGame, DoubleDQN z NoisyLinear.

**Proponowane zmiany (diff):**

```python
# src/rl_agent.py — konfiguracja agenta

class DQNAgent:
    def __init__(self, config):
        self.gamma = 0.99
        self.grad_clip = 1.0           # gradient clipping norm
        self.action_repeat = 4          # powtórz każdą akcję 4 razy
        self.train_every_n_steps = 4    # trenuj co 4 kroki env
        # NoisyLinear zamiast epsilon-greedy
        self.network = NoisyDQN(input_dim=84*84*4, output_dim=2)

    def select_action(self, state, deterministic=False):
        with torch.no_grad():
            self.network.eval()
            q_values = self.network(state)
            action = q_values.argmax(dim=1)
        self.network.train()  # ← zawsze przywraca train mode
        return action.item()

    def learn(self, batch):
        # ... standard DQN loss ...
        loss.backward()
        torch.nn.utils.clip_grad_norm_(self.network.parameters(), self.grad_clip)
        self.optimizer.step()

# src/env.py — reward shaping
REWARD_PIPE = 1.0       # przejście przez rurę
REWARD_ALIVE = 0.1      # za każdy krok przeżycia
REWARD_DEATH = -1.0     # śmierć

# watch.py — odtwarzanie modelu
def watch(checkpoint_path):
    agent = DQNAgent.load(checkpoint_path)
    env = FlappyBirdEnv()
    state = env.reset()
    while True:
        action = agent.select_action(state)  # ← brak deterministic=True
        state, reward, done, info = env.step(action)
        env.render()
        if done:
            state = env.reset()
```

**Użytkownik uruchamia:**
```
/krytyk src/rl_agent.py src/env.py watch.py
```

---

## Krytyk 1: Teoria (RL Correctness)

```
🔴 KRYTYK TEORETYCZNY — Analiza poprawności algorytmu RL
══════════════════════════════════════════════════════════
```

### Issue T1: Survival reward dominuje task reward

**Problem:** `REWARD_ALIVE = 0.1` z `gamma = 0.99` daje nieskończony horyzont contribution:

```
Q_survival = reward / (1 - gamma) = 0.1 / (1 - 0.99) = 10.0
```

Porównajmy z reward za przejście przez rurę. Rury pojawiają się średnio co ~25 kroków:

```
Q_pipe = 1.0 * gamma^25 = 1.0 * 0.99^25 ≈ 0.78
```

**Stosunek: 10.0 : 0.78 = 12.8:1**

Agent otrzymuje 12.8x większy sygnał za "po prostu żyj" niż za "przeleć przez rurę". Racjonalna strategia: lataj w kółko, unikaj rur (bo ryzyko śmierci), zbieraj survival reward.

**Objaw:** Agent uczy się utrzymywać wysokość ale nie leci do przodu / unika rur.

**Fix:** Zmniejsz `REWARD_ALIVE` do `0.01`:
```
Q_survival = 0.01 / 0.01 = 1.0
Q_pipe ≈ 0.78
Stosunek: 1.0 : 0.78 = 1.3:1  ← zdrowa proporcja
```

**Severity: 🔴 KRYTYCZNY** — to bug, który sprawia, że agent uczy się **złej strategii**.

---

### Issue T2: Gradient clipping za niski dla CNN

**Problem:** `grad_clip = 1.0`, ale sieć `NoisyDQN` z warstwami konwolucyjnymi generuje gradienty o normie 5-15 (typowe dla CNN z batchnorm-free architekturami).

```python
# Typowe normy gradientów w trakcie treningu:
# MLP (128,128):    norm ≈ 0.5-3.0  → clip 1.0 OK
# CNN (3xConv+2xFC): norm ≈ 5.0-15.0 → clip 1.0 OBCINA 80%+ gradientów
```

Clipping na 1.0 oznacza, że każdy update jest skalowany w dół 5-15x. Agent się uczy, ale **5-15x wolniej** niż powinien.

**Objaw:** Trening "działa" ale loss spada ekstremalnie powoli. Wygląda jak zbyt niski learning rate.

**Fix:** Podnieś `grad_clip` do `10.0` dla CNN lub użyj adaptive clipping:
```python
# Opcja 1: Stały, wyższy clip
self.grad_clip = 10.0

# Opcja 2: Monitoruj i dostosuj
grad_norm = torch.nn.utils.clip_grad_norm_(params, self.grad_clip)
if grad_norm > self.grad_clip * 0.9:  # clipping aktywny >90% czasu
    logger.warning(f"Grad clip active: {grad_norm:.1f} > {self.grad_clip}")
```

**Severity: 🟡 WAŻNY** — trening działa, ale 5-15x wolniej.

---

### Issue T3: NoisyNet warmup produkuje biased exploration

**Problem:** `NoisyLinear` inicjalizuje wszystkie `sigma` tą samą wartością (typowo `sigma_init = 0.5`). W pierwszych krokach treningu noise jest identyczny na wszystkich neuronach — agent nie eksploruje prawdziwie losowo, tylko deterministycznie z przesunięciem.

```python
# NoisyLinear.__init__:
self.sigma_weight = nn.Parameter(torch.full((out_features, in_features), sigma_init))
# Wszystkie sigma = 0.5 → noise * 0.5 → biased w jedną stronę
```

Dopiero po kilku tysiącach kroków `sigma` zaczynają się różnicować przez gradient updates.

**Objaw:** Pierwsze 1-5k kroków agent wykonuje te same (lub bardzo podobne) akcje, potem nagle zaczyna eksplorować. Replay buffer na starcie jest mocno biased.

**Fix:** Dodaj warmup z uniform random actions:
```python
def select_action(self, state, step, warmup_steps=5000):
    if step < warmup_steps:
        return random.randint(0, self.action_dim - 1)
    # ... normalny NoisyNet select_action
```

**Severity: 🟡 WAŻNY** — subtlelny bias w early exploration. Może powodować "martwy start" w treningu.

---

### Issue T4: Effective update ratio mocno zaniżony

**Problem:** `action_repeat = 4` + `train_every_n_steps = 4`. Na pozór: agent zbiera 4 kroki, potem trenuje. Ale te parametry **mnożą się**:

```
Env steps per action decision: 4 (action_repeat)
Training frequency: co 4 decyzje (train_every_n_steps)
Effective: 1 gradient update na 4 × 4 = 16 env steps
```

Typowy DQN: 1 update na 1-4 env steps. Tu mamy **1 na 16** — agent uczy się 4-16x wolniej niż standard.

**Objaw:** Trening wymaga 4x więcej environment steps niż oczekiwano.

**Fix:** Zmień `train_every_n_steps` na 1 (trenuj po każdej decyzji, nie po każdym env step):
```python
# Albo: train_every_n_steps = 1
# Albo: zmień logikę — licz decyzje, nie env steps
if decisions_made % train_every_n_steps == 0:
    agent.learn(batch)
```

**Severity: 🟡 WAŻNY** — trening działa, ale wymaga 4x więcej czasu.

---

## Krytyk 2: Inżynieria (Bugi i Edge Cases)

```
🔵 KRYTYK INŻYNIERYJNY — Analiza kodu pod kątem bugów
══════════════════════════════════════════════════════════
```

### Issue E1: select_action() psuje eval mode

**Problem:** Funkcja zawsze kończy się na `self.network.train()`:

```python
def select_action(self, state, deterministic=False):
    with torch.no_grad():
        self.network.eval()
        q_values = self.network(state)
        action = q_values.argmax(dim=1)
    self.network.train()  # ← ZAWSZE przywraca train mode
    return action.item()
```

Jeśli gdziekolwiek w kodzie masz:
```python
agent.network.eval()
for state in eval_states:
    action = agent.select_action(state)  # ← przełącza na train!
    # od tego momentu network jest w train mode
    # BatchNorm, Dropout zachowują się inaczej
```

**Objaw:** Ewaluacja daje niestabilne wyniki. Metryki eval są zaszumione (bo model jest w train mode — NoisyNet dodaje noise, Dropout dropuje neurony).

**Fix:** Nie zmieniaj mode w select_action — niech caller decyduje:
```python
def select_action(self, state, deterministic=False):
    with torch.no_grad():
        q_values = self.network(state)
        if deterministic:
            action = q_values.argmax(dim=1)
        else:
            # NoisyNet handles exploration via noise
            action = q_values.argmax(dim=1)
    return action.item()
```

**Severity: 🔴 KRYTYCZNY** — cichy bug, trudny do wykrycia, psuje ewaluację i może maskować inne problemy.

---

### Issue E2: watch.py nie używa deterministic mode

**Problem:**
```python
action = agent.select_action(state)  # ← brak deterministic=True
```

W watch mode chcemy zobaczyć **najlepszą** grę agenta. Bez `deterministic=True`, NoisyNet dodaje losowy noise do Q-values — agent podejmuje suboptymalne decyzje.

**Objaw:** Watch mode wygląda gorzej niż faktyczne umiejętności agenta. Użytkownik myśli, że trening nie zadziałał, a tak naprawdę agent jest lepszy niż widać.

**Fix:**
```python
action = agent.select_action(state, deterministic=True)
```

**Severity: 🟡 WAŻNY** — nie psuje treningu, ale daje fałszywy obraz umiejętności agenta.

---

### Issue E3: Brak seed consistency

**Problem:** Nigdzie nie ma ustawionego `torch.manual_seed()`, `np.random.seed()`, `random.seed()`. Każdy run daje inne wyniki — nie da się porównywać eksperymentów.

**Fix:**
```python
def set_seed(seed=42):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
```

**Severity: 🟢 SUGESTIA** — nie bug, ale essential dla reproducibility.

---

### Issue E4: Brak graceful shutdown

**Problem:** Jeśli trening zostanie przerwany (Ctrl+C), checkpoint nie jest zapisywany. Godziny treningu stracone.

**Fix:**
```python
import signal

def save_on_interrupt(signum, frame):
    print("Saving checkpoint before exit...")
    agent.save(f"runs/interrupted_{step}.pth")
    sys.exit(0)

signal.signal(signal.SIGINT, save_on_interrupt)
```

**Severity: 🟢 SUGESTIA** — quality of life, ale bardzo ważne przy długich treningach.

---

## Konsensus — Spotkanie Krytyków

```
🤝 KONSENSUS — Oba krytyki zgadzają się co do tych problemów:
═════════════════════════════════════════════════════════════
```

| # | Problem | Krytyk 1 | Krytyk 2 | Severity |
|---|---------|----------|----------|----------|
| 1 | Survival reward dominuje 12.8:1 | ✅ T1 (matematyka) | ✅ (potwierdza) | 🔴 KRYTYCZNY |
| 2 | select_action psuje eval mode | ✅ (zauważony) | ✅ E1 (szczegóły) | 🔴 KRYTYCZNY |
| 3 | Grad clip za niski | ✅ T2 (analiza) | ✅ (potwierdza) | 🟡 WAŻNY |
| 4 | watch.py brak deterministic | ✅ T3 (NoisyNet) | ✅ E2 (implementacja) | 🟡 WAŻNY |
| 5 | Effective update ratio 1/16 | ✅ T4 (interakcja) | ✅ (potwierdza) | 🟡 WAŻNY |

**Priorytet fixów:**
1. 🔴 `REWARD_ALIVE`: 0.1 → 0.01
2. 🔴 `select_action`: usunąć side-effect na model mode
3. 🟡 `grad_clip`: 1.0 → 10.0
4. 🟡 `watch.py`: dodać `deterministic=True`
5. 🟡 `train_every_n_steps`: zmienić logikę liczenia

---

## Rezolucja — Fixy

### Fix 1: Reward shaping
```python
# src/env.py
REWARD_PIPE = 1.0
REWARD_ALIVE = 0.01    # było 0.1 → Q-contribution 10.0 → teraz 1.0
REWARD_DEATH = -1.0
```

### Fix 2: select_action bez side-effectów
```python
# src/rl_agent.py
def select_action(self, state, deterministic=False):
    was_training = self.network.training
    self.network.eval()
    with torch.no_grad():
        q_values = self.network(state)
        action = q_values.argmax(dim=1)
    if was_training:
        self.network.train()
    return action.item()
```

### Fix 3: Gradient clipping
```python
self.grad_clip = 10.0  # było 1.0 — za niskie dla CNN
```

### Fix 4: Watch deterministic
```python
# watch.py
action = agent.select_action(state, deterministic=True)
```

### Fix 5: Update ratio
```python
# Licz decyzje, nie env steps
if agent_decisions % train_every_n_steps == 0:
    agent.learn(batch)
```

---

## Dlaczego /krytyk łapie bugi, których człowiek nie łapie

1. **Dwa różne "oczy":** Teoretyk patrzy na matematykę (Q-contributions, gradient norms), inżynier patrzy na control flow (mode switching, missing flags). Człowiek rzadko robi oba jednocześnie.

2. **Brak ego:** Agent nie ma przywiązania do kodu. Nie myśli "to ja napisałem, na pewno jest ok". Patrzy czysto na fakty.

3. **Interakcje między komponentami:** Bug T4 (action_repeat × train_every_n_steps) to interakcja dwóch sensownych decyzji, które razem dają zły wynik. Każda osobno wygląda ok. Dopiero analiza kombinacji ujawnia problem.

4. **Ciche bugi:** E1 (select_action psuje eval) to bug, który NIE daje errora. Kod działa, wyniki są "ok-ish", ale suboptymalne. Bez code review nikt by tego nie znalazł przez tygodnie.

5. **Matematyczna weryfikacja:** Człowiek patrzy na `REWARD_ALIVE = 0.1` i myśli "niewielki bonus". Krytyk liczy: `0.1 / (1-0.99) = 10.0` i widzi, że to **dominujący sygnał**. Intuicja vs algebra.

---

## 💡 Tip: Kiedy używać /krytyk

- **Przed merge'em dużego PR** — szczególnie jeśli zmienia reward function lub architekturę
- **Kiedy trening "nie działa" a nie wiesz dlaczego** — krytyk znajdzie subtelne bugi
- **Po dodaniu nowego komponentu** — interakcje z istniejącym kodem są najczęstszym źródłem bugów

## ⚠️ Uwaga: /krytyk nie zastępuje testów

Krytyk to code review, nie test suite. Łapie bugi logiczne, nie regresje. Nadal potrzebujesz unit testów i integration testów.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
