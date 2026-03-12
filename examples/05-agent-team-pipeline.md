# Agent Team — Pipeline Build→Test→Fix

> Agent Team to tryb, w którym agenty komunikują się **bezpośrednio ze sobą**, bez przechodzenia przez mastera.
> Idealny do tight feedback loops: build → test → fix → retest.
> Zamiast subagentów (fire-and-forget), masz zespół który iteruje w pętli.

---

## Kiedy Team, a kiedy Subagenty?

| Kryterium | Subagenty (Task) | Team (TeamCreate) |
|-----------|-------------------|---------------------|
| Komunikacja | Przez mastera | Bezpośrednio między sobą |
| Iteracje | 1 (fire-and-forget) | 3+ (feedback loop) |
| Zależności | Niezależne taski | Pipeline (output jednego → input drugiego) |
| Przykład | "Zrefaktoryzuj 3 pliki" | "Zbuduj tool → przetestuj → napraw → retestuj" |

**Reguła kciuka:** Jeśli spodziewasz się 3+ rund komunikacji między agentami — użyj Team.

---

## Scenariusz: Budowanie File Merger Tool

**Cel:** Narzędzie webowe (single HTML file) do łączenia wielu plików tekstowych w jeden. Część zestawu narzędzi na stronabeby.pl.

**Wymagania:**
- Upload wielu plików (.txt, .csv, .md)
- Drag-and-drop reorder
- Podgląd zawartości każdego pliku
- Separator między plikami (konfigurowalny)
- Przycisk "Scal i Pobierz"
- Responsywny design (mobile + desktop)
- Dark mode

---

## Setup Teamu

```
TeamCreate:
  name: "merger-build"
  agents:
    - name: "Coder"
      role: "Full-stack developer. Budujesz narzędzia webowe jako single HTML files."
      model: sonnet
      directory: /home/beba/narzedzia-public

    - name: "Tester"
      role: "QA engineer. Testujesz narzędzia webowe z Playwright (Python, headless Chromium)."
      model: sonnet
      directory: /home/beba/narzedzia-public
```

**Dlaczego Team a nie subagenty?**
Coder i Tester będą iterować: Coder buduje → Tester testuje → Tester raportuje bugi → Coder fiksuje → Tester retestuje. To klasyczny pipeline z 3+ rundami. Subagenty wymagałyby ręcznego przekazywania raportów — Team robi to automatycznie.

---

## Runda 1: Build + Test

### Coder buduje

**Task do Codera:**
```
Zbuduj merge.html — narzędzie do łączenia plików tekstowych.

Wymagania:
- Upload: input type=file multiple, accept .txt,.csv,.md
- Lista plików: drag-and-drop reorder (vanilla JS, HTML5 Drag API)
- Podgląd: klik na plik → pokaż zawartość w panelu obok
- Separator: input text, default "---\n"
- Przycisk "Scal i Pobierz" → generuje i downloaduje merged.txt
- Style: ciemny motyw, max-width 800px, wycentrowany
- Zero zależności zewnętrznych — vanilla HTML/CSS/JS w jednym pliku

Po zbudowaniu powiedz Testerowi żeby przetestował.
```

Coder produkuje `merge.html` — ~250 linii HTML/CSS/JS. Wysyła wiadomość do Testera:

```
→ Tester: merge.html gotowy w /home/beba/narzedzia-public/merge.html
Funkcje: upload, drag-drop reorder, preview, separator config, download.
Przetestuj pełny flow.
```

### Tester testuje

Tester uruchamia Playwright:

```python
# test_merge.py (generowany przez Tester agenta)
from playwright.sync_api import sync_playwright

def test_merge_tool():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto("file:///home/beba/narzedzia-public/merge.html")

        # Test 1: Strona ładuje się
        assert page.title() != ""
        print("✅ Test 1: Strona załadowana")

        # Test 2: Upload plików
        page.set_input_files("#file-input", [
            "test_files/a.txt",
            "test_files/b.txt",
            "test_files/c.txt"
        ])
        items = page.query_selector_all(".file-item")
        assert len(items) == 3
        print("✅ Test 2: Upload 3 plików")

        # Test 3: Preview
        items[0].click()
        preview = page.text_content("#preview-panel")
        assert "content of a.txt" in preview
        print("✅ Test 3: Preview działa")

        # Test 4: Download
        with page.expect_download() as download_info:
            page.click("#merge-btn")
        download = download_info.value
        content = download.path().read_text()
        assert "content of a.txt" in content
        assert "content of b.txt" in content
        print("✅ Test 4: Download działa")

        # Test 5: Separator
        page.fill("#separator-input", "===BREAK===\\n")
        with page.expect_download() as download_info:
            page.click("#merge-btn")
        download = download_info.value
        content = download.path().read_text()
        assert "===BREAK===" in content
        print("✅ Test 5: Custom separator")

        # Test 6: Drag-and-drop reorder
        # ... drag item[2] before item[0]
        # BUG: po drag-drop kolejność w pobranym pliku się nie zmienia!
        print("❌ Test 6: Drag-drop reorder — kolejność w download nie zmieniona")

        # Test 7: Mobile viewport
        page.set_viewport_size({"width": 375, "height": 667})
        # BUG: Upload przycisk wychodzi poza ekran
        upload_btn = page.query_selector("#upload-btn")
        box = upload_btn.bounding_box()
        assert box["x"] + box["width"] <= 375
        print("❌ Test 7: Mobile — przycisk upload wychodzi poza viewport")

        # Test 8: Pusty upload
        page.set_input_files("#file-input", [])
        page.click("#merge-btn")
        # BUG: Generuje pusty plik zamiast pokazać alert
        print("❌ Test 8: Pusty upload — brak walidacji")

        browser.close()
```

### Tester raportuje do Codera

```
→ Coder: Raport testów merge.html

✅ PASS: 5/8 testów
❌ FAIL: 3 bugi

BUG 1 (🔴 KRYTYCZNY): Drag-drop reorder zmienia widok, ale NIE zmienia
kolejności w generowanym pliku. Plik zawsze ma oryginalną kolejność uploadu.
Przyczyna: downloadFile() czyta z oryginalnego array files[], nie z DOM order.

BUG 2 (🟡 WAŻNY): Na mobile (375px) przycisk upload wychodzi poza viewport
o ~30px w prawo. Brakuje max-width: 100% lub box-sizing: border-box.

BUG 3 (🟡 WAŻNY): Kliknięcie "Scal i Pobierz" bez załadowanych plików
generuje pusty plik. Powinien: alert lub disabled button gdy files.length === 0.
```

---

## Runda 2: Fix + Retest

### Coder fiksuje

```
→ Tester: Fixuję 3 bugi.

BUG 1: downloadFile() teraz czyta kolejność z DOM:
  const orderedFiles = Array.from(document.querySelectorAll('.file-item'))
    .map(el => files[el.dataset.index]);

BUG 2: Dodano box-sizing: border-box na container i max-width: 100% na buttony.

BUG 3: Merge button ma atrybut disabled gdy files.length === 0.
  Listener na file input toggleuje disabled.
  Dodany tooltip "Najpierw załaduj pliki".

Retestuj.
```

### Tester retestuje

```
→ Coder: Retest raport

✅ Test 6 (drag-drop): PASS — kolejność w pliku odpowiada kolejności po drag
✅ Test 7 (mobile): PASS — przycisk mieści się w viewport 375px
✅ Test 8 (pusty upload): PASS — przycisk disabled, tooltip widoczny

NOWY BUG znaleziony:
❌ Test 9 (edge case): Upload 50 plików → drag-drop staje się
niereagujący. Przeglądarka freezuje na ~2s przy każdym drag.
Przyczyna: prawdopodobnie brak debounce/throttle na drag events,
lub preview panel odświeża się przy każdym mouse move.
```

---

## Runda 3: Final Fix + Green

### Coder fiksuje edge case

```
→ Tester: Fix dla 50-plikowego edge case:

1. Preview panel aktualizuje się tylko na dragend, nie dragover
2. Dodano requestAnimationFrame() w drag handler
3. Dla >20 plików: preview panel wyłączony podczas drag (performance)

Retestuj final.
```

### Tester: Final Pass

```
→ Coder: FINAL RAPORT

✅ 9/9 testów PASS
✅ Performance: 50 plików drag-drop działa płynnie
✅ Mobile: responsywny na 375px, 768px, 1024px
✅ Edge cases: pusty upload, 1 plik, 50 plików, duże pliki (5MB)

Narzędzie gotowe do deploymentu.
```

---

## Podsumowanie przepływu

```
Runda 1:
  Coder → [build merge.html] → Tester
  Tester → [test: 5 pass, 3 fail] → Coder

Runda 2:
  Coder → [fix 3 bugów] → Tester
  Tester → [retest: 8 pass, 1 new fail] → Coder

Runda 3:
  Coder → [fix edge case] → Tester
  Tester → [final: 9/9 pass] → DONE
```

**Total: 3 rundy, 4 bugi znalezione i naprawione, zero interwencji człowieka.**

Z subagentami ten sam proces wymagałby:
1. Odpal Codera → czekaj → czytaj output
2. Odpal Testera ręcznie → czekaj → czytaj raport
3. Odpal Codera z raportem → czekaj → czytaj output
4. Odpal Testera ponownie → czekaj → czytaj raport
5. Powtórz 3-4...

Team zrobił to sam, w tle, bez twojego udziału.

---

## Kiedy Team jest gorszy niż subagenty?

1. **Proste taski** — "Zrefaktoryzuj ten plik" nie wymaga feedback loop. Subagent jest szybszy.
2. **Niezależne taski** — "Zbuduj 3 różne narzędzia" — żaden agent nie potrzebuje output drugiego. 3 subagenty równolegle.
3. **Drogie modele** — Team na opus z 5 rundami = dużo tokenów. Rozważ sonnet dla Testera.

---

## 💡 Tip: Dawaj Testerowi konkretne scenariusze

Nie mów "przetestuj to". Daj listę scenariuszy z expected results. Tester jest tak dobry jak jego specyfikacja testowa.

## 💡 Tip: Limituj rundy

Ustaw max 5 rund w Team. Jeśli po 5 rundach nie jest green — coś jest fundamentalnie źle i wymaga ludzkiej interwencji. Nieskończony loop Coder↔Tester to marnowanie tokenów.

## ⚠️ Uwaga: Team agents dzielą filesystem

Coder i Tester operują na tych samych plikach. Jeśli Coder edytuje `merge.html` w tym samym momencie co Tester go czyta — race condition. Team framework to zarządza (sekwencyjne tury), ale bądź świadomy.

---

*Wróć do [głównego README](../README.md) po więcej materiałów kursu.*
