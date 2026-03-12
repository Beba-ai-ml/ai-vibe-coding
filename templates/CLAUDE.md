# CLAUDE.md — Instrukcja Projektu

> Skopiuj ten plik do katalogu głównego swojego projektu i wypełnij sekcje.
> Claude Code czyta ten plik automatycznie przy każdej sesji.
> Im dokładniejszy opis, tym lepiej Claude rozumie Twój projekt.

---

## Przegląd Projektu

<!-- Co robi ten projekt? Napisz 2-3 zdania. -->
<!-- Przykład: -->
Aplikacja webowa do śledzenia nawyków. Użytkownik loguje się, dodaje nawyki
do śledzenia i widzi statystyki w formie wykresów. Backend Flask, frontend
czysty HTML/JS bez frameworków.

---

## Stack Technologiczny

<!-- Jakich technologii używasz? -->
<!-- Przykład: -->
- **Backend:** Python 3.11, Flask 3.0, SQLAlchemy 2.0
- **Frontend:** HTML5, CSS3, Vanilla JS (bez frameworków)
- **Baza danych:** SQLite (dev), PostgreSQL (prod)
- **Testy:** pytest, pytest-flask
- **Inne:** python-dotenv, Flask-Login

---

## Kluczowe Pliki

<!-- Wymień pliki, które Claude powinien znać jako pierwsze. -->
<!-- Przykład: -->
- `app.py` — punkt wejścia, rejestracja blueprintów, konfiguracja Flask
- `models.py` — modele SQLAlchemy (User, Habit, Entry)
- `routes/habits.py` — CRUD dla nawyków (najczęściej edytowany plik)
- `static/app.js` — cała logika frontendu
- `tests/test_habits.py` — testy integracyjne tras

---

## Jak Uruchomić

<!-- Dokładne komendy do uruchomienia projektu. -->
<!-- Przykład: -->
```bash
# Aktywuj środowisko
source venv/bin/activate

# Zainstaluj zależności (tylko pierwszy raz)
pip install -r requirements.txt

# Zainicjuj bazę danych (tylko pierwszy raz)
flask db init && flask db migrate && flask db upgrade

# Uruchom serwer deweloperski
flask run --debug
# → http://localhost:5000
```

---

## Jak Testować

<!-- Jak uruchamiać testy? -->
<!-- Przykład: -->
```bash
pytest                          # wszystkie testy
pytest tests/test_habits.py     # konkretny plik
pytest -k "test_create"         # konkretny test
pytest --cov=app                # z pokryciem kodu
```

---

## Konwencje Kodowania

<!-- Ważne zasady, których Claude musi przestrzegać. -->
<!-- Przykład: -->
- Nazwy tras: małe litery z myślnikami (`/user-profile`, nie `/userProfile`)
- Nazwy funkcji: snake_case, opisowe (`get_habit_by_id`, nie `getHabit`)
- Każda trasa musi mieć test integracyjny
- Błędy zwracamy jako JSON: `{"error": "opis błędu"}` z odpowiednim kodem HTTP
- Komentarze w kodzie po polsku
- Jeden plik = jedna odpowiedzialność (nie tworzyć plików-worków)

---

## Czego NIE Robić

<!-- Rzeczy, których Claude absolutnie NIE powinien robić. -->
<!-- Przykład: -->
- ⚠️ NIE używaj `flask.g` do przechowywania stanu — używaj sesji Flask-Login
- ⚠️ NIE commituj pliku `.env` — jest w `.gitignore`
- ⚠️ NIE zmieniaj schematu bazy bez migracji (`flask db migrate`)
- ⚠️ NIE używaj `SELECT *` w zapytaniach — zawsze wskazuj kolumny
- ⚠️ NIE instaluj jQuery ani innych bibliotek bez pytania
