# Ćwiczenie 3: Zbuduj Projekt z Równoległymi Agentami

**Poziom:** Średni
**Czas:** 60–120 minut
**Cel:** Zbudować działającą aplikację, gdzie trzy niezależne agenty pracują
równolegle nad osobnymi komponentami, a następnie zintegrować wyniki.

---

## Czego się nauczysz

- Jak dzielić projekt na niezależne komponenty dla równoległych agentów
- Jak definiować kontrakty (interfejsy) między komponentami PRZED implementacją
- Jak unikać konfliktów gdy wiele agentów pracuje na tym samym projekcie
- Jak integrować kod z różnych agentów

---

## Projekt: Mini Dashboard Zadań

Zbudujesz prostą aplikację webową do zarządzania zadaniami (TODO list),
podzieloną na trzy niezależne komponenty:

```
┌─────────────────────────────────────────┐
│           Frontend UI (Agent 2)         │
│  - HTML/CSS/JS                          │
│  - Formularz dodawania zadań            │
│  - Lista zadań z checkboxami            │
│  - Wywołania do API                     │
├─────────────────────────────────────────┤
│           Backend API (Agent 1)         │
│  - Flask REST API                       │
│  - GET /api/tasks                       │
│  - POST /api/tasks                      │
│  - PATCH /api/tasks/:id                 │
│  - DELETE /api/tasks/:id                │
├─────────────────────────────────────────┤
│         Baza Danych (Agent 3)           │
│  - SQLite + schemat                     │
│  - Skrypt inicjalizacyjny               │
│  - Dane testowe (seed)                  │
└─────────────────────────────────────────┘
```

---

## Krok 1: Zaprojektuj Architekturę (Zrób Sam)

Zanim odpalisz agentów, zdefiniuj kontrakty między komponentami.
To jest NAJWAŻNIEJSZY krok — błędy tutaj powodują niekompatybilność.

### Kontrakt API (format JSON)

Zdecyduj teraz jak wygląda obiekt zadania:

```json
{
  "id": 1,
  "title": "Kupić mleko",
  "done": false,
  "created_at": "2025-01-15T10:30:00"
}
```

### Endpointy API

| Metoda | Ścieżka | Co robi | Zwraca |
|--------|---------|---------|--------|
| GET | `/api/tasks` | Lista wszystkich zadań | `{"tasks": [...]}` |
| POST | `/api/tasks` | Dodaj zadanie | `{"task": {...}}` (201) |
| PATCH | `/api/tasks/<id>` | Zmień `done` na true/false | `{"task": {...}}` |
| DELETE | `/api/tasks/<id>` | Usuń zadanie | `{"message": "deleted"}` (204) |

### Schemat Bazy Danych

```sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    done BOOLEAN DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

💡 **Zapisz te kontrakty.** Wkleisz je do promptów każdego agenta.

---

## Krok 2: Przygotuj Katalog Projektu

```bash
mkdir dashboard-zadan
cd dashboard-zadan
mkdir -p backend frontend database
```

Struktura docelowa:
```
dashboard-zadan/
├── backend/
│   ├── app.py
│   └── requirements.txt
├── frontend/
│   └── index.html
├── database/
│   ├── schema.sql
│   ├── seed.sql
│   └── init_db.py
└── README.md
```

---

## Krok 3: Napisz Prompty dla Agentów

Otwórz trzy osobne terminale. W każdym:

```bash
cd dashboard-zadan
claude
```

### Prompt dla Agenta 1 — Backend

```
Zbuduj backend Flask REST API dla aplikacji TODO list.

KONTRAKT API (musisz dokładnie go przestrzegać):

Format obiektu zadania (JSON):
{
  "id": 1,
  "title": "Kupić mleko",
  "done": false,
  "created_at": "2025-01-15T10:30:00"
}

Endpointy do implementacji:
- GET  /api/tasks         → {"tasks": [...]}
- POST /api/tasks         → {"task": {...}} + status 201
                            Body: {"title": "tekst"}
- PATCH /api/tasks/<id>   → {"task": {...}}
                            Body: {"done": true/false}
- DELETE /api/tasks/<id>  → {"message": "deleted"} + status 204

Schemat bazy danych (plik: database/tasks.db):
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    done BOOLEAN DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Wymagania:
- Plik: backend/app.py
- Plik: backend/requirements.txt
- Flask 3.0, SQLite3 (wbudowany), flask-cors
- CORS włączony (frontend będzie na innym porcie)
- Port: 5000
- Obsługa błędów: 404 gdy zadanie nie istnieje, 400 gdy brak tytułu
- Nie twórz frontendu — tylko API

Wygeneruj kompletne pliki gotowe do uruchomienia.
```

### Prompt dla Agenta 2 — Frontend

```
Zbuduj frontend aplikacji TODO list jako pojedynczy plik HTML.

KONTRAKT API (backend działa na http://localhost:5000):

Endpointy:
- GET  /api/tasks         → {"tasks": [{id, title, done, created_at}, ...]}
- POST /api/tasks         → Body: {"title": "tekst"}
- PATCH /api/tasks/<id>   → Body: {"done": true}
- DELETE /api/tasks/<id>  → usuwa zadanie

Wymagania:
- Plik: frontend/index.html
- Jeden plik HTML z osadzonym CSS i JS (zero zewnętrznych zależności)
- Design: czysty, minimalny, biało-szary
- Funkcje:
  * Wyświetlanie listy zadań (pobiera z API przy załadowaniu)
  * Formularz dodawania nowego zadania (Enter lub przycisk "Dodaj")
  * Checkbox przy każdym zadaniu — zaznaczenie wysyła PATCH do API
  * Przycisk "×" przy każdym zadaniu — usuwa przez DELETE
  * Licznik: "X zadań do zrobienia" aktualizowany dynamicznie
  * Obsługa błędów: wyświetl komunikat gdy API nie odpowiada
- Używaj fetch() z async/await
- Nie twórz backendu — tylko frontend

Wygeneruj kompletny plik frontend/index.html.
```

### Prompt dla Agenta 3 — Baza Danych

```
Stwórz skrypty inicjalizacyjne bazy danych SQLite dla aplikacji TODO list.

Schemat:
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    done BOOLEAN DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Wymagania — stwórz trzy pliki:

1. database/schema.sql
   - Czysty SQL tworzący tabelę
   - DROP TABLE IF EXISTS przed CREATE TABLE

2. database/seed.sql
   - 10 przykładowych zadań (mieszanka: 3 ukończone, 7 nieukończonych)
   - Zadania po polsku, realistyczne

3. database/init_db.py
   - Skrypt Python inicjalizujący bazę
   - Tworzy plik database/tasks.db
   - Wykonuje schema.sql
   - Wykonuje seed.sql
   - Wypisuje "Baza danych gotowa. Załadowano X zadań."
   - Uruchamialne: python database/init_db.py

Nie twórz ani backendu, ani frontendu.
```

---

## Krok 4: Uruchom Agenty Równolegle

Wklej każdy prompt do odpowiedniego terminala i naciśnij Enter.
Nie czekaj — odpalisz wszystkie trzy jednocześnie.

⚠️ **Każdy agent pracuje w tym samym katalogu `dashboard-zadan`.**
Przypisaliśmy im osobne podkatalogi (`backend/`, `frontend/`, `database/`),
więc nie będą sobie przeszkadzać.

Monitoruj postęp w każdym terminalu. Gdy wszystkie skończą — przejdź dalej.

---

## Krok 5: Zintegruj i Przetestuj

```bash
# Terminal 1: Zainicjuj bazę danych
python database/init_db.py

# Terminal 2: Uruchom backend
cd backend
pip install -r requirements.txt
python app.py
# → Flask działa na http://localhost:5000

# Terminal 3: Otwórz frontend
xdg-open frontend/index.html   # Linux
open frontend/index.html        # macOS
```

**Lista kontrolna integracji:**
- [ ] Backend startuje bez błędów
- [ ] `curl http://localhost:5000/api/tasks` zwraca JSON z listą zadań
- [ ] Frontend ładuje się w przeglądarce
- [ ] Lista zadań pojawia się po załadowaniu strony
- [ ] Można dodać nowe zadanie
- [ ] Checkbox zmienia stan zadania
- [ ] Przycisk × usuwa zadanie
- [ ] Odświeżenie strony zachowuje zmiany

---

## Rozwiązywanie Problemów

**Problem: CORS error w przeglądarce**
```
Access to fetch at 'http://localhost:5000' has been blocked by CORS policy
```
Rozwiązanie: zapytaj Agenta 1 o dodanie flask-cors.

**Problem: Frontend widzi inne pola niż backend zwraca**
Sprawdź `Network` tab w DevTools (F12) — porównaj co backend faktycznie
zwraca z tym, co frontend próbuje odczytać.

**Problem: Baza danych nie istnieje**
Upewnij się, że uruchomiłeś `python database/init_db.py` przed backendem.

---

## Kryteria Sukcesu ✅

- [ ] Wszystkie trzy komponenty działają razem
- [ ] Można wykonać pełny CRUD (Create, Read, Update, Delete) przez UI
- [ ] Dane są persystowane (przeżywają restart backendu)

## Bonus

Dodaj czwarty agent — Agent 4: Testy:
```
Napisz testy integracyjne dla API z pliku backend/app.py używając pytest.
Przetestuj każdy endpoint: GET, POST, PATCH, DELETE.
Użyj tymczasowej bazy danych w pamięci (nie nadpisuj tasks.db).
Plik: backend/test_api.py
```

---

## Refleksja

Po ćwiczeniu zastanów się:
1. Który komponent był najtrudniejszy do zintegrowania i dlaczego?
2. Co byś zmienił w kontraktach API wiedząc to co wiesz teraz?
3. Czy równoległe agenty były szybsze niż jeden agent sekwencyjny?
