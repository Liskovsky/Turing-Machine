# Simulace Deterministického Turingova Stroje (TS)

Tento projekt implementuje simulátor **deterministického 2‑páskového Turingova stroje** v jazyce Python.
Stroj je navržen k demonstraci výpočtu funkce **součinu** ($\Pi_{i=1}^{n} x_i$) binárně kódovaných čísel.

---

## 1. Architektura a Modulární Komponenty

Projekt využívá objektově orientovaný přístup, kde každá klíčová součást Turingova stroje je reprezentována samostatnou třídou.

### Základní Konstanty a Pohyby
- `Movement(Enum)`: Definuje možné pohyby čtecí/zapisovací hlavy:
  - `LEFT` (−1)
  - `RIGHT` (+1)
  - `NONE` (0)

### Abeceda a Stavy
- `Alphabet(list)`: Rozšiřuje standardní seznam. Udržuje abecedu stroje (`0`, `1`) a definuje **prázdný symbol** (`#`).
- `State`: Reprezentuje jeden stav s identifikátorem a příznaky pro **počáteční** (`starting`) a **koncový** (`ending`) stav.
- `States(dict[int, State])`: Kolekce stavů. Zajišťuje, že je definován právě jeden počáteční stav a minimálně jeden koncový stav.

### Pásky
- `Tape(list[str])`: Implementuje pásku jako seznam symbolů. Udržuje **pozici hlavy** (`_position`) a příznaky pro **vstupní** (`_input=True`) a **výstupní** (`_output=True`) pásku.
  - Při inicializaci vstupní pásky je hlava nastavena na **první neprázdný symbol**.
  - Metoda `move_head()` řídí posun hlavy a vyvolává chybu `OutOfRangeError` při pokusu o posun mimo meze.
- `Tapes(list[Tape])`: Kolekce pásek. Poskytuje rozhraní pro čtení/zápis symbolů (`tape_values`) a posun hlav (`move_heads`) napříč všemi páskami současně.

### Přechodové Funkce
- `Transition`: Definuje jedno pravidlo přechodu.
  - **Klíč** (`Key`): `(aktuální_stav, aktuální_symboly_pásek)`
  - **Hodnota** (`Value`): `(nový_stav, nové_symboly_pásek, pohyby_hlav)`
- `Transitions(dict[Key, Transition])`: Slovník přechodových funkcí, umožňující rychlé vyhledávání.

---

## 2. Simulátor (`TuringMachine`)

Třída `TuringMachine` řídí chod stroje.

- **Validace:** Před spuštěním kontroluje konzistenci:
  - Všechny stavy a symboly v přechodech musí existovat ve specifikaci stroje.
  - Obsah pásek musí odpovídat definované abecedě.
- **Logování:** V každém kroku simulace je volána metoda `_write_info`, která loguje:
  - Číslo kroku, aktuální stav.
  - Obsah a pozici hlavy každé pásky.
  - Konkrétní použité přechodové pravidlo.
- **Chod stroje (`iterate`):**
  1. Zarovnání pásek (`resize_tapes`).
  2. Výpis počátečního stavu pásek.
  3. Rekurzivní volání `_iterate()` dokud není dosažen koncový stav (`EndingStateReached`).
  4. Po dosažení koncového stavu výpis výsledku z výstupní pásky, unárního kódování stroje a tabulky všech kroků.

---

## 3. Implementovaná Funkce (Součin)

Simulátor je demonstrován na 2‑páskovém stroji realizujícím funkci:

<img width="269" height="92" alt="image" src="https://github.com/user-attachments/assets/24188218-bc8a-4eff-ba49-d7adb2f287e7" />

### Konfigurace (`FunConfig`)
- Konfigurace je navržena pro 2‑páskový TS.
- Stavy **6** a **7** implementují **binární sčítací logiku** (s přenosem), což umožňuje realizovat násobení ($x_1 \cdot x_2$) jako **opakované sčítání**.

### Předzpracování Vstupu (Režim `multi`)
Protože násobení je implementováno jako opakované sčítání, je vstup upraven funkcí `preprocess_product_input(raw_input)`.

- **Vstupní formát (uživatelský):** `x1#x2` (např. `101#11`).
- **Zpracování:** Druhé číslo (`x2`) je interpretováno jako binární násobitel.
- **Vstup pro TS:** Je vytvořena sekvence, kde je první číslo (`x1`) zopakováno `x2` krát a odděleno `#`.

**Příklad:**

> 101#11 → ##101#101#101##

### 3.1 Režim `fun` — Součin libovolného počtu čísel

Projekt byl rozšířen o novou funkci **`fun`**, která umožňuje spočítat součin libovolného počtu binárně kódovaných čísel:

$$FUN(x_1, x_2, \dots, x_n) = x_1 \cdot x_2 \cdot \dots \cdot x_n$$

Tato funkce využívá již existující logiku násobení dvou čísel (režim `multi`) a skládá ji **iterativně**:

```python
prod = 1
for každé číslo xi:
    prod = multiply_two(prod, xi)
