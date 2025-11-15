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

101#11 → ##101#101#101##


---

## 4. Kódování Turingova Stroje

Po dokončení simulace je stroj zakódován pomocí třídy `TuringMachineCoder` do **unární podoby**.

### Základní symboly
- Hodnota: `0`
- Oddělovač dat v přechodu: `1`
- Oddělovač hlavních sekcí: `11`
- Začátek/Konec kódu: `111`

### Struktura kódu
1. `111` (Začátek)
2. Kódování **abecedy**
3. Kódování **stavů** (počet stavů, startovní stav, koncové stavy)
4. Kódování **přechodových funkcí**
5. `111` (Konec)

---

## 5. Spuštění

Program se spouští z metody `main()`:

```python
def main():
    mode = input("Zvol režim (sum nebo multi): ").strip().lower()
    raw_input = input("Zadej vstupní řetězec (např. 101#11): ").strip()
    # ...
```
## Doporučený scénář

- **Zvolte režim:** `multi`  
- **Zadejte vstup:** `101#11` (vstup pro 5 x 3)  
- **Očekávaný výstup:** binární reprezentace čísla **15** (`1111`) na výstupní pásce.

---

## 6. Výstupy Simulace

Po skončení simulace (`EndingStateReached`) program poskytne:

1. **Tabulku kroků:**
   - `Krok {n}, Stav: {q}, Čteno: {symboly}`
   - `Přechod: δ({q}, symboly) = (q', symboly', pohyby)`

2. **Koncový stav výstupní pásky:**  
   Obsah pásky 2 (`_output=True`) s pozicí hlavy.

3. **Kódování Turingova stroje:**  
   Binární řetězec unárního kódu stroje.

4. **Binární výsledek:**  
   Řetězec symbolů (očištěný od `#`) z výstupní pásky.

---

---

## Nedostatky

Projekt je funkční, ale má několik zásadních omezení, která je nutné do budoucna odstranit:

1. **Sčítání pouze pro tři čísla**  
   - Aktuální implementace zvládne sčítat jen tři čísla, nikoli dvě.  
   - **Plánované řešení:** Upravit logiku vstupního parseru a přechodů tak, aby se sčítání spustilo ihned po nalezení prvního oddělovače `#`. Tím bude možné provádět klasické binární sčítání dvou operandů.

2. **Násobení pouze pro dvě čísla**  
   - Stroj zatím neumí násobit více než dva operandy.  
   - **Plánované řešení:** Doplnit **třetí pásku**, která bude sloužit pro staging třetího čísla.  
     - Příklad vstupu: `###101#10#11###`  
       - Páska 1: obsahuje všechna čísla.  
       - Páska 2: mezivýsledek `101 * 10`.  
       - Páska 3: uložený násobitel `11`.  
     - Výpočet: nejprve se spočítá `101 * 10` na pásce 2, poté se výsledek násobí číslem `11` z pásky 3 opakovaným sčítáním.

---

