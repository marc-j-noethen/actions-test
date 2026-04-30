# GitHub Secrets „Zero‑Plaintext“ – Schritt für Schritt (Git Bash)

Ziel: Du nutzt ein Secret in GitHub Actions, **ohne** dass es im Log im Klartext erscheint.  
Erwartung: Wenn der Workflow versucht das Secret auszugeben, siehst du im Log **Maskierung** (meist `***`) statt z.B. `SuperSecret123`.

---

## A) Secret in GitHub anlegen (Web)

1. Öffne dein Repository: `https://github.com/marc-j-noethen/actions-test`
2. Gehe zu **Settings** (Einstellungen)
3. Links: **Secrets and variables** → **Actions**
4. Klick **New repository secret**
5. Name: `MY_SENSITIVE_TOKEN`
6. Secret value: z.B. `SuperSecret123`
7. Klick **Add secret**

---

## B) Lokal sicherstellen, dass du wirklich im Git-Repo bist

In **Git Bash**:

```bash
cd /c/Users/lukas/Desktop/codex-projekte/actions-test
git status
```

### Wenn `git status` sagt: „not a git repository“

Dann ist in diesem Ordner keine `.git/` vorhanden. Am saubersten: neu klonen.

1) In den Eltern-Ordner:

```bash
cd /c/Users/lukas/Desktop/codex-projekte
```

2) Den aktuellen Ordner sichern (kein Löschen):

```bash
mv actions-test actions-test_backup_$(date +%Y%m%d_%H%M%S)
```

3) Repo klonen:

```bash
git clone https://github.com/marc-j-noethen/actions-test.git
cd actions-test
```

---

## C) Workflow-Datei erstellen: `.github/workflows/secret_test.yml`

```bash
mkdir -p .github/workflows
cat > .github/workflows/secret_test.yml <<'YML'
name: secret-test

on:
  workflow_dispatch:

jobs:
  secret_test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Attempt to print secret (should be masked)
        run: echo "MY_SENSITIVE_TOKEN=${{ secrets.MY_SENSITIVE_TOKEN }}"
YML
```

Optional prüfen:

```bash
cat .github/workflows/secret_test.yml
```

---

## D) Commit & Push

```bash
git add .github/workflows/secret_test.yml
git commit -m "Add secret_test workflow (masked secret demo)"
git push
```

---

## E) Workflow manuell ausführen + Screenshot (Einreichung)

1. Öffne: `https://github.com/marc-j-noethen/actions-test`
2. Tab **Actions**
3. Workflow **secret-test** auswählen
4. Button **Run workflow** → ausführen
5. Den Run öffnen → Job `secret_test` öffnen
6. Screenshot von der Log-Zeile machen, die zeigt, dass das Secret **maskiert** ist:
   - Erwartet: `MY_SENSITIVE_TOKEN=***` (oder ähnlich maskiert)
   - Nicht erlaubt: `MY_SENSITIVE_TOKEN=SuperSecret123`

---

## Hinweis (Sicherheits-Realität)

- GitHub maskiert Secrets in Logs standardmäßig, aber **nur** wenn die exakte Zeichenfolge erkannt wird.
- Wenn du Secrets transformierst (z.B. Base64, Teile abschneidest, mit Spaces mischst), kann Maskierung in manchen Fällen ausfallen.

