# Zapieczętowane prognozy kursów PLN

Każdy katalog `rundy/<id>/` to jedna runda prognoz zapisana **przed** poznaniem wyniku.
Prognozy są w zaszyfrowanym archiwum; hasło jest publikowane dopiero po ogłoszeniu przez NBP
kursu z dnia rozstrzygnięcia. Poniżej: jak sprawdzić, że plik nie został zmieniony, jak go otworzyć
i jak sprawdzić, kiedy powstał.

## Pliki

| Plik | Zawartość |
|---|---|
| `kursy.csv` | kursy średnie NBP (tabela A, z numerem tabeli) od 2002 r. — wiersze są tylko dopisywane |
| `rundy.csv` | spis rund: id, dzień odcięcia, czas publikacji (UTC), skróty SHA-256 |
| `rozstrzygniecia.csv` | spis rozstrzygniętych rund: id, dzień rozstrzygnięcia, hasło |
| `rundy/<id>/prognozy.7z` | zapieczętowane prognozy (7z, AES-256, zaszyfrowane także nazwy plików) |
| `rundy/<id>/prognozy.7z.ots` | znacznik czasu OpenTimestamps dla `prognozy.7z` |
| `rundy/<id>/SHA256SUMS` | skrót SHA-256 pliku `prognozy.7z` |
| `rundy/<id>/haslo.txt`, `rozstrzygniecie.csv` | po terminie: hasło oraz kurs rzeczywisty i ocena |

Prognoza rundy korzysta wyłącznie z danych dostępnych w dniu odcięcia (kolumna `data_odciecia`).
Kursy z `kursy.csv` można porównać ze źródłem: <https://api.nbp.pl/> (tabela A).

## 1. Sprawdzenie skrótu

W katalogu rundy:

```bash
sha256sum -c SHA256SUMS            # Linux
shasum -a 256 -c SHA256SUMS        # macOS
```

```powershell
Get-FileHash prognozy.7z -Algorithm SHA256   # Windows (PowerShell) — porównaj z SHA256SUMS
```

Ten sam skrót `prognozy.7z` jest w `rundy.csv` (kolumna `sha256_pieczeci`).

## 2. Otwarcie archiwum (po publikacji hasła)

Hasło jest w `rundy/<id>/haslo.txt` i w `rozstrzygniecia.csv`.

- Windows: [7-Zip](https://www.7-zip.org/) lub PeaZip — „Rozpakuj”, wpisz hasło.
- macOS: Keka lub The Unarchiver.
- Linux: `7z x prognozy.7z -p<hasło>` (pakiet `7zip` lub `p7zip-full`).

Wbudowane narzędzia Windows i macOS mogą nie obsługiwać archiwów 7z z szyfrowaniem AES-256.

Po rozpakowaniu skrót SHA-256 pliku `prognozy.csv` musi być równy kolumnie `sha256_prognoz` w `rundy.csv`
(opublikowanej przed terminem).

## 3. Sprawdzenie daty

Plik `prognozy.7z.ots` to dowód OpenTimestamps: skrót archiwum zapisany w łańcuchu bloków Bitcoina.
Potwierdzenie pojawia się kilka godzin po utworzeniu rundy.

```bash
pip install opentimestamps-client
ots verify prognozy.7z.ots        # w katalogu z prognozy.7z
```

Można też przeciągnąć oba pliki na stronę <https://opentimestamps.org>. Wynik podaje najwcześniejszy
moment, w którym plik musiał już istnieć. Daty commitów w historii tego repozytorium są tylko
pomocnicze — w gicie można je ustawić dowolnie.

Pliki `kursy.csv`, `rundy.csv` i `rozstrzygniecia.csv` są wyłącznie dopisywane: wcześniejsze wiersze
nigdy się nie zmieniają, co widać w historii zmian tych plików.
