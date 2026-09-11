# Instrukcja krok po kroku: Jak uruchomić warsztat na MyBinder

Ten dokument wyjaśnia, jak w 5 minut opublikować materiały szkoleniowe na platformie **MyBinder (mybinder.org)**, aby kursanci mogli pracować w przeglądarce **bez konta Google, bez konta GitHub i bez instalowania czegokolwiek na komputerze**.

---

## Dlaczego ta konfiguracja zadziała od ręki?

Folder `Spark_Workshop_Binder` zawiera już wszystkie pliki konfiguracyjne wymagane przez platformę Binder:
* `.binder/apt.txt` & `apt.txt` -> automatycznie instaluje środowisko **Java (OpenJDK)** niezbędne dla silnika Apache Spark.
* `.binder/requirements.txt` & `requirements.txt` -> instaluje **`pyspark`**, **`apache-airflow`**, **`pandas`**, **`requests`**.
* `.binder/runtime.txt` & `runtime.txt` -> wymusza stabilną wersję **Python 3.11**.
* Wszystkie 5 notebooków `.ipynb` dla uczestników (ćwiczenia `# TODO` + testy `assert`).
* Autentyczne dane **StatsBomb Bundesliga** preinstalowane w `data/raw/`.

---

## KROK 1: Utwórz publiczne repozytorium na GitHubie

1. Zaloguj się na swoje konto na [github.com](https://github.com).
2. Kliknij **New repository** (Nowe repozytorium).
3. Ustawienia:
   * **Repository name:** np. `spark-workshop-binder` (lub dowolna inna nazwa).
   * **Public:** Ustaw na **Public** (MyBinder wymaga, aby repozytorium było publiczne).
   * Nie zaznaczaj opcji *Add a README file* (mamy już gotowy README w folderze).
4. Kliknij **Create repository**.

---

## KROK 2: Wypchnij folder `Spark_Workshop_Binder` na GitHuba

W terminalu (PowerShell lub Git Bash) wejdź do folderu `Spark_Workshop_Binder` i wykonaj:

```bash
cd "C:\Users\Wojciech.Sledziewski\OneDrive - ONWELO S.A\Projects\CF\ApacheSpark\Spark_Workshop_Binder"

# Inicjalizacja gita
git init
git add .
git commit -m "Add PySpark Binder workshop setup with StatsBomb data"

# Ustawienie głównej gałęzi na main
git branch -M main

# Podepnij swoje repozytorium z GitHuba (zamień TWOJ_LOGIN i NAZWA_REPO na swoje dane):
git remote add origin https://github.com/TWOJ_LOGIN/spark-workshop-binder.git

# Wypchnij kod
git push -u origin main
```

---

## KROK 3: Wygeneruj link na MyBinder.org

1. Otwórz w przeglądarce stronę: **[mybinder.org](https://mybinder.org)**.
2. W polu **GitHub repository name or URL** wklej adres swojego repozytorium, np.:
   ```text
   https://github.com/TWOJ_LOGIN/spark-workshop-binder
   ```
3. W polu **Git ref (branch, tag, or commit)** wpisz:
   ```text
   main
   ```
4. *(Opcjonalnie, rekomendowane)* W sekcji **Path to a notebook file (optional)** wybierz z listy **URL** i wpisz:
   ```text
   lab
   ```
   *(Dzięki temu Binder od razu otworzy nowoczesny interfejs JupyterLab zamiast klasycznego widoku plików).*
5. Kliknij pomarańczowy przycisk **launch**.

---

## KROK 4: Pierwsze budowanie obrazu (Build)

* **Czas trwania:** Za pierwszym razem Binder musi pobrać Javę i zainstalować PySparka – potrwa to **około 3–4 minuty**.
* W oknie zobaczysz logi instalacji (pobieranie Javy i bibliotek).
* Po zakończeniu strona automatycznie przekieruje Cię do działającego środowiska **JupyterLab** w chmurze.
* **Ważne:** Binder **zapamiętuje (cachuje)** ten zbudowany kontener! Gdy kursanci klikną ten link w dniu szkolenia, środowisko otworzy im się w **10–20 sekund**.

---

## KROK 5: Co udostępniasz uczestnikom?

Na stronie [mybinder.org](https://mybinder.org) skopiuj link z pola **Copy the URL below to share your Binder that will open directly to your environment**.

Będzie on wyglądał tak:
```text
https://mybinder.org/v2/gh/TWOJ_LOGIN/spark-workshop-binder/main?urlpath=lab
```

Ten jeden link wysyłasz uczestnikom (na Teams, mailu lub wklejasz na czacie spotkania).

### Co widzi uczestnik?
* Klika link.
* **Nie musi się logować.**
* Nie ma znaczenia, czy ma konto Google, Microsoft czy GitHub.
* Od razu po lewej stronie widzi listę plików:
  * `01_spark_foundations_and_architecture.ipynb`
  * `02_dataframes_complex_types_and_sql.ipynb`
  * `03_performance_shuffle_and_storage.ipynb`
  * `04_airflow_spark_orchestration.ipynb`
  * `05_production_framework_and_capstone.ipynb`
  * folder `data/raw/` z autentycznymi danymi Bundesligi.
* Klika w plik i od razu uruchamia komórki kodu `Shift + Enter`!

---

## ⚠️ Kluczowa zasada dla uczestników (Zapisywanie pracy)

Środowisko Binder jest **ulotne (ephemeral)**. Jeśli uczestnik zamknie kartę lub odejdzie na obiad na dłużej niż 15 minut bezczynności, sesja wygaśnie i zresetuje się do stanu początkowego.

**Przekaż kursantom prostą zasadę:**
> *„Gdy rozwiążecie zadania w danym notatniku, kliknijcie prawym przyciskiem na plik po lewej stronie i wybierzcie **Download**, aby zapisać swój plik `.ipynb` z rozwiązaniem na dysku laptopa.”*
