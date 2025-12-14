# GENETIČKI ALGORITAM - Kompletan edukativni vodič

## Sadržaj
1. [Uvod u genetičke algoritme](#uvod)
2. [Osnovna terminologija](#terminologija)
3. [Kako radi GA - korak po korak](#kako-radi-ga)
4. [Kodiranje rješenja (Lab 7)](#kodiranje)
5. [Selekcijski operatori](#selekcija)
6. [Genetički operatori](#geneticki-operatori)
7. [Kompletna implementacija](#implementacija)
8. [Testiranje i analiza](#testiranje)
9. [Savjeti i trikovi](#savjeti)

---

## <a name="uvod"></a>1. UVOD U GENETIČKE ALGORITME

### Šta je genetički algoritam?

**Genetički algoritam (GA)** je metaheuristički algoritam za optimizaciju koji imitira proces **prirodne evolucije**. Inspirisan je Darvinovom teorijom evolucije:

> "Survival of the fittest" - Preživljavanje najsposobnijih

### Zašto koristimo GA?

Genetički algoritmi su korisni za:
- **Globalnu optimizaciju** - pronalaženje globalnih optimuma u kompleksnim prostorima pretraživanja
- **Probleme sa mnogo lokalnih optimuma** - gdje tradicionalni metodi (gradient descent) često zapnu
- **Crne kutije** - kada ne znamo gradijent funkcije
- **Kombinatornu optimizaciju** - TSP, Knapsack, Scheduling, itd.

### Osnovna analogija

| Biologija | Genetički Algoritam |
|-----------|---------------------|
| Individua | Kandidatno rješenje |
| Gen | Jedan bit u hromozomu |
| Hromozom | Binarni niz koji predstavlja rješenje |
| Populacija | Skup kandidatnih rješenja |
| Fitness | Kvalitet rješenja |
| Prirodna selekcija | Selekcijski operator |
| Reprodukcija | Ukrštanje (crossover) |
| Mutacija | Slučajna promjena gena |
| Generacija | Iteracija algoritma |

---

## <a name="terminologija"></a>2. OSNOVNA TERMINOLOGIJA

### 2.1 Individua (Chromosome)

**Individua** je jedno kandidatno rješenje problema, predstavljeno kao **hromozom** - binarni niz.

**Primjer:**
```
Hromozom: [1, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1]
          ↑                                          ↑
         bit 13                                    bit 0
```

### 2.2 Populacija (Population)

**Populacija** je skup individua (kandidatnih rješenja).

**Primjer:** Populacija sa 4 individue
```
Individua 1: [1, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1]
Individua 2: [0, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 0]
Individua 3: [1, 1, 1, 0, 0, 1, 1, 1, 0, 1, 0, 1, 1, 0]
Individua 4: [0, 0, 0, 1, 1, 0, 0, 0, 1, 0, 1, 0, 0, 1]
```

### 2.3 Fitness funkcija

**Fitness funkcija** mjeri kvalitet rješenja. U našem slučaju (Lab 7), to je **modificirana Rastrigin funkcija**:

$$f(x_1, x_2) = 80 - \sum_{i=1}^{2} (x_i^2 - 10\cos(2\pi x_i))$$

- **Veći fitness = bolje rješenje**
- **Cilj:** Pronaći $f(0, 0) = 100$ (globalni maksimum)

### 2.4 Generacija

**Generacija** je jedna iteracija algoritma - ciklus selekcije, ukrštanja, mutacije i zamjene populacije.

---

## <a name="kako-radi-ga"></a>3. KAKO RADI GA - KORAK PO KORAK

### Pseudokod

```
GENETIČKI ALGORITAM:

1. INICIJALIZACIJA
   - Generiši slučajnu početnu populaciju
   - Evaluiraj fitness svake individue

2. PONAVLJAJ za maxgen generacija:

   a) SELEKCIJA
      - Odaberi roditelje na osnovu fitnesa

   b) UKRŠTANJE (Crossover)
      - Kombiniraj roditelje → kreiraj djecu

   c) MUTACIJA
      - Nasumično promijeni neke gene

   d) EVALUACIJA
      - Izračunaj fitness djece

   e) ZAMJENA POPULACIJE
      - Formiranje nove generacije
      - Primjena elitizma (prenesi najbolje)

   f) PROVJERA KONVERGENCIJE
      - Ako je pronađeno dovoljno dobro rješenje → STOP

3. VRATI najbolju individuu
```

### Vizualni prikaz jedne generacije

```
GENERACIJA N                    GENERACIJA N+1
┌─────────────┐                ┌─────────────┐
│  Populacija │                │  Nova       │
│             │                │  Populacija │
│  Ind 1 ███  │  Selekcija    │             │
│  Ind 2 ███  │ ────────────> │  Elite  ███ │
│  Ind 3 ███  │  Ukrštanje    │  Dijete ███ │
│  Ind 4 ███  │  Mutacija     │  Dijete ███ │
│  ...        │                │  ...        │
└─────────────┘                └─────────────┘
```

---

## <a name="kodiranje"></a>4. KODIRANJE RJEŠENJA (Lab 7)

U Lab 7, tražimo maksimum funkcije $f(x_1, x_2)$ gdje su $x_1, x_2 \in [-5.12, 5.12]$.

### 4.1 Struktura hromozoma

Hromozom dužine **d** kodira **dvije koordinate** (x1, x2):

```
Hromozom: [bd-1, bd-2, ..., br, br-1, ..., b0]
           └───────x1────────┘ └─────x2──────┘
```

gdje:
- **d = 2k + 2** (uvijek paran broj)
- **r = d/2** (polovina)
- **Prvih r bita** kodiraju **x1**
- **Drugih r bita** kodiraju **x2**

**Primjer:** d=14 → r=7
```
Hromozom: [b13, b12, b11, b10, b9, b8, b7 | b6, b5, b4, b3, b2, b1, b0]
           └────────── x1 ───────────────┘  └────────── x2 ──────────┘
              7 bita za x1                      7 bita za x2
```

### 4.2 Kodiranje jedne koordinate (x)

Za svaku koordinatu koristimo **r bita**:

```
[predznak | cijeli_dio | decimalni_dio]
    1 bit   p bita       (r-1-p) bita
```

**Struktura:**
1. **Bit 0 (najznačajniji):** Predznak
   - `0` → pozitivan broj (x ≥ 0)
   - `1` → negativan broj (x < 0)

2. **Sljedećih p bita:** Cijeli dio broja |x|
   - Primjer: `p=3` → može kodirati 0-7

3. **Preostalih (r-1-p) bita:** Decimalni dio broja |x|
   - Primjer: `r-1-p=3` → može kodirati 0.000-0.875

**Formula za dekodiranje:**

$$x = (-1)^{predznak} \times \left( \sum_{i=0}^{p-1} b_i \cdot 2^{p-1-i} + \sum_{j=0}^{r-1-p-1} b_j \cdot 2^{-(j+1)} \right)$$

### 4.3 Primjer dekodiranja

**Postavke:** d=14, r=7, p=3

**Hromozom za x1:** `[0, 0, 1, 0, 1, 1, 0]`

```
Pozicija:    6    5    4    3    2    1    0
Bit:       [ 0 |  0    1    0 |  1    1    0 ]
           znak  cijeli dio    decimalni dio
```

**Dekodiranje:**

1. **Predznak:** bit[6] = 0 → pozitivan broj

2. **Cijeli dio:** biti [5, 4, 3] = [0, 1, 0]
   ```
   = 0×2² + 1×2¹ + 0×2⁰
   = 0 + 2 + 0
   = 2
   ```

3. **Decimalni dio:** biti [2, 1, 0] = [1, 1, 0]
   ```
   = 1×2⁻¹ + 1×2⁻² + 0×2⁻³
   = 0.5 + 0.25 + 0
   = 0.75
   ```

4. **Ukupno:**
   ```
   x1 = (+1) × (2 + 0.75) = 2.75
   ```

### 4.4 Zašto p=3 za opseg [-5.12, 5.12]?

Za p bita, možemo kodirati cijele brojeve od **0 do 2^p - 1**:

| p | Max cijeli dio | Može kodirati [-5.12, 5.12]? |
|---|----------------|------------------------------|
| 2 | 3              | ❌ Ne (max 3.999...)          |
| 3 | 7              | ✅ Da (max 7.875)             |
| 4 | 15             | ✅ Da (ali nepotrebno)        |

Biramo **p=3** jer je dovoljan i efikasan.

---

## <a name="selekcija"></a>5. SELEKCIJSKI OPERATORI

Selekcija je proces **odabira roditelja** za reprodukciju. Bolje individue imaju **veću šansu** da budu odabrane.

### 5.1 Ruletska selekcija (Roulette Wheel Selection)

**Ideja:** Vjerovatnoća selekcije je **proporcionalna fitnesu**.

**Formula:**
$$p_i = \frac{fitness_i}{\sum_{j=1}^{N} fitness_j}$$

**Vizualizacija:**
```
      Ruletski točak

    ╱─────────────╲
   │   Ind 3 (20%) │
   │               │
   │ Ind 1 (50%)   │  ← Najveća šansa (najviši fitness)
   │               │
   │ Ind 2 (20%)   │
    ╲─ Ind 4 (10%)/
```

**Algoritam:**
```python
1. Izračunaj ukupan fitness: S = Σ fitness_i
2. Generiši slučajan broj r ∈ [0, S]
3. Prolazi kroz individue:
   - Saberi njihove fitness-e
   - Kad suma premaši r → selektuj tu individuu
```

**Karakteristike:**
- ✅ Jednostavna implementacija
- ✅ Dobre individue imaju veću šansu
- ❌ Problem sa negativnim fitness-ima
- ❌ Preuranjena konvergencija (dominantna individua preuzme populaciju)

### 5.2 Rang selekcija (Rank Selection)

**Ideja:** Rangir individue po fitnesu, vjerovatnoća zavisi od **ranga**, ne direktno od fitnesa.

**Algoritam:**
```python
1. Sortiraj populaciju po fitnesu
2. Dodijeli rangove:
   - Najgora individua: rang 0
   - Najbolja individua: rang N-1
3. Izračunaj vjerovatnoću za svaki rang:
   p_i = (2 - SP)/N + 2·i·(SP - 1)/(N·(N-1))
   gdje je SP = selekcijski pritisak
```

**Selekcijski pritisak (SP):**
- **SP ∈ [1, 2]**
- **SP = 1:** Sve individue imaju istu šansu (random selekcija)
- **SP = 2:** Linearno rangiranje (najbolja ima 2× veću šansu od prosječne)
- **Tipično:** SP = 1.5

**Primjer:** N=4, SP=1.5
```
Rang  Individua  Fitness  Vjerovatnoća
  3   Najbolja     98        33%
  2   Druga        75        28%
  1   Treća        50        22%
  0   Najgora      20        17%
```

**Karakteristike:**
- ✅ Nema problema sa negativnim fitness-ima
- ✅ Kontrola selekcijskog pritiska (SP parametar)
- ✅ Sprječava preuranjenu konvergenciju
- ❌ Malo sporija od ruletske (zbog sortiranja)

---

## <a name="geneticki-operatori"></a>6. GENETIČKI OPERATORI

### 6.1 Ukrštanje (Crossover)

**Svrha:** Kombinuj **dobre osobine** dva roditelja da bi kreirali potomke.

#### Ukrštanje u jednoj tački (Single-Point Crossover)

**Algoritam:**
```python
1. Odaberi slučajnu tačku presjecanja t ∈ [1, d-1]
2. Dijete 1 = Roditelj1[0:t] + Roditelj2[t:d]
3. Dijete 2 = Roditelj2[0:t] + Roditelj1[t:d]
```

**Primjer:** t=7
```
Roditelj 1: [1, 0, 1, 1, 0, 0, 1 | 0, 1, 1, 0, 0, 1, 1]
Roditelj 2: [0, 1, 0, 0, 1, 1, 0 | 1, 0, 0, 1, 1, 0, 0]
                                 ↑ tačka presjecanja

Dijete 1:   [1, 0, 1, 1, 0, 0, 1 | 1, 0, 0, 1, 1, 0, 0]
             └─ Od Roditelja 1 ─┘  └─ Od Roditelja 2 ─┘

Dijete 2:   [0, 1, 0, 0, 1, 1, 0 | 0, 1, 1, 0, 0, 1, 1]
             └─ Od Roditelja 2 ─┘  └─ Od Roditelja 1 ─┘
```

**Vjerovatnoća ukrštanja (Pc):**
- Tipično: **Pc = 0.6 - 0.9**
- Ako `random() > Pc` → ne vrši se ukrštanje, djeca = roditelji

### 6.2 Mutacija (Mutation)

**Svrha:** Održavanje **diverziteta** u populaciji, izbjegavanje lokalnih optimuma.

#### Binarna mutacija (Bit-Flip Mutation)

**Algoritam:**
```python
Za svaki bit u hromozomu:
    Ako random() < Pm:
        Flipuj bit (0→1, 1→0)
```

**Primjer:** Pm=0.01
```
Prije:  [1, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0, 1, 1]
               ↓              ↓           ↓
Poslije: [1, 0, 0, 1, 0, 0, 0, 0, 1, 1, 1, 0, 1, 1]
         (flipovali smo bite na pozicijama 2, 6 i 10)
```

**Vjerovatnoća mutacije (Pm):**
- Tipično: **Pm = 0.001 - 0.05** (veoma niska!)
- **Pravilo palca:** Pm = 1/d (gdje je d dužina hromozoma)
- Razlog: Mutacija je **rijedak** događaj u prirodi

---

## <a name="implementacija"></a>7. KOMPLETNA IMPLEMENTACIJA

### 7.1 Arhitektura sistema

```
┌──────────────────────────────────────────────────────────┐
│                   GENETIČKI ALGORITAM                    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────────┐                                     │
│  │ ApstraktnaInd. │ (ABC)                               │
│  └───────┬────────┘                                     │
│          │                                               │
│          ↓                                               │
│  ┌────────────────┐                                     │
│  │ MojaIndividua  │                                     │
│  ├────────────────┤                                     │
│  │ • Hromozom     │                                     │
│  │ • Fitness      │                                     │
│  │ • Evaluiraj()  │ ← Rastrigin funkcija               │
│  └────────────────┘                                     │
│                                                          │
│  ┌──────────────────────────────────────────────┐      │
│  │              POPULACIJA                       │      │
│  ├──────────────────────────────────────────────┤      │
│  │ ATRIBUTI:                                     │      │
│  │  • Lista individua                            │      │
│  │  • Parametri GA (Pc, Pm, maxgen, elite)      │      │
│  ├──────────────────────────────────────────────┤      │
│  │ METODE:                                       │      │
│  │  1. OpUkrstanjaTacka()                        │      │
│  │  2. OpBinMutacija()                           │      │
│  │  3. SelekcijaRTocak()                         │      │
│  │  4. SelekcijaRang()                           │      │
│  │  5. NovaGeneracija()                          │      │
│  │  6. GenerisiGeneracije() ← GLAVNI LOOP        │      │
│  └──────────────────────────────────────────────┘      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 7.2 Glavni algoritam (GenerisiGeneracije)

```python
def GenerisiGeneracije(self, metoda_selekcije='rulet'):
    # Početna evaluacija
    for individua in populacija:
        individua.Evaluiraj()

    # Glavna petlja
    for generacija in range(maxgen):

        # 1. Nova generacija
        nova_populacija = []

        # 2. Elitizam - prenesi najbolje
        if elitizam > 0:
            sortiraj_po_fitnesu(populacija)
            for i in range(elitizam):
                nova_populacija.add(kopija(populacija[i]))

        # 3. Popuni ostatak populacije
        while len(nova_populacija) < N:

            # a) SELEKCIJA
            if metoda == 'rulet':
                roditelj1 = SelekcijaRTocak()
                roditelj2 = SelekcijaRTocak()
            else:
                roditelj1 = SelekcijaRang()
                roditelj2 = SelekcijaRang()

            # b) UKRŠTANJE
            (dijete1, dijete2) = OpUkrstanjaTacka(roditelj1, roditelj2)

            # c) MUTACIJA
            dijete1 = OpBinMutacija(dijete1)
            dijete2 = OpBinMutacija(dijete2)

            # d) EVALUACIJA
            dijete1.Evaluiraj()
            dijete2.Evaluiraj()

            # e) DODAJ U NOVU POPULACIJU
            nova_populacija.add(dijete1, dijete2)

        # 4. ZAMJENA POPULACIJE
        populacija = nova_populacija

        # 5. STATISTIKA
        track_best_fitness()
        track_avg_fitness()

    # 6. VRATI NAJBOLJU INDIVIDUU
    return max(populacija, key=lambda x: x.fitness)
```

---

## <a name="testiranje"></a>8. TESTIRANJE I ANALIZA

### 8.1 Rastrigin funkcija

**Formula:**
$$f(x_1, x_2) = 80 - \sum_{i=1}^{2} (x_i^2 - 10\cos(2\pi x_i))$$

**Karakteristike:**
- **Visoko multimodalna** - mnogo lokalnih optimuma
- **Globalni maksimum:** f(0, 0) = 100
- **Domen:** [-5.12, 5.12] × [-5.12, 5.12]

**Zašto je teška:**
```
      ▲
  100 │    ╱╲
      │   ╱  ╲   ╱╲
   80 │  ╱    ╲ ╱  ╲   ← Mnogo lokalnih maksimuma
      │ ╱      ╲    ╲
      │╱             ╲
    ──┴────────────────────► x
```

### 8.2 Parametri GA

**Preporučene vrijednosti:**

| Parametar | Vrijednost | Objašnjenje |
|-----------|------------|-------------|
| Veličina populacije | 30-100 | Balans između performansi i brzine |
| Vjerovatnoća ukrštanja | 0.6-0.9 | Visoka (ukrštanje je glavni operator) |
| Vjerovatnoća mutacije | 0.001-0.05 | Niska (rijedak događaj) |
| Broj generacija | 50-200 | Zavisi od težine problema |
| Elitizam | 1-2 | Čuva najbolje rješenje |
| Selekcijski pritisak | 1.5 | Za rang selekciju |

### 8.3 Konvergencija algoritma

**Grafik konvergencije:**
```
Fitness
  100 │                     ╱─────────────
      │                   ╱
   90 │                 ╱
      │               ╱
   80 │             ╱    Najbolji fitness
      │           ╱
   70 │         ╱──────── Prosječni fitness
      │       ╱
      └───────────────────────────────────► Generacija
      0      20      40      60      80     100
```

**Faze:**
1. **Eksploracija (0-30):** Brzo poboljšanje, populacija istražuje prostor
2. **Eksploatacija (30-70):** Sporije poboljšanje, fokus na dobre regije
3. **Konvergencija (70-100):** Minimalno poboljšanje, populacija konvergirala

---

## <a name="savjeti"></a>9. SAVJETI I TRIKOVI

### 9.1 Kako odabrati parametre?

**1. Veličina populacije**
- **Previše mala** → Premalo diverziteta, lokalni optimum
- **Previše velika** → Sporo, skupa računski
- **Pravilo:** 30-100 za većinu problema

**2. Vjerovatnoća ukrštanja**
- **Visoka (0.8-0.9):** Brža konvergencija, više eksploracije
- **Niska (0.5-0.6):** Sporija konvergencija, čuva dobre rješenja
- **Preporuka:** Počni sa 0.8

**3. Vjerovatnoća mutacije**
- **Visoka:** Previše nasumična pretraga (gubimo dobre rješenje)
- **Niska:** Mogućnost zaglavljivanja u lokalnom optimumu
- **Pravilo palca:** Pm = 1/d ili 0.01

**4. Elitizam**
- **0:** Možemo izgubiti najbolje rješenje
- **1-2:** Osigurava da najbolje rješenje ne bude izgubljeno
- **Više od 2:** Može uzrokovati preranu konvergenciju

### 9.2 Kako izbjeći uobičajene greške?

**❌ Greška 1: Prezagađena populacija**
```python
# LOŠE: Previše dobar započetka → svi slični → bez diverziteta
populacija = [najbolja_individua_copy for _ in range(N)]

# DOBRO: Slučajna početna populacija
populacija = [random_individua() for _ in range(N)]
```

**❌ Greška 2: Preuranjena konvergencija**
```python
# LOŠE: Svi brzo postanu isti
Pm = 0.0001  # Previše niska mutacija

# DOBRO: Održavaj diverzitet
Pm = 0.01    # Razumna mutacija
```

**❌ Greška 3: Predugo čekanje**
```python
# LOŠE: Ne prati napredak
for gen in range(1000):  # Previše generacija?
    evolve()

# DOBRO: Rani stopping
for gen in range(maxgen):
    evolve()
    if best_fitness > 99.9:  # Dovoljno dobro
        break
```

### 9.3 Ruletska vs Rang selekcija?

| Aspekt | Ruletska | Rang |
|--------|----------|------|
| **Performanse** | Brža | Malo sporija (sortiranje) |
| **Robusnost** | Osjetljiva na fitnes skalu | Robusna |
| **Preuranjena konvergencija** | Česta | Rijetka |
| **Preporuka** | Za jednostavne probleme | Za složene probleme |

**Pravilo palca:** Ako nisi siguran, **koristi rang selekciju sa SP=1.5**.

### 9.4 Debugging savjeti

**1. Prati fitness kroz generacije**
```python
best_fitness_history = []
for gen in range(maxgen):
    ...
    best_fitness_history.append(max(fitness_values))

# Ako se fitness NE poboljšava → problem!
plt.plot(best_fitness_history)
```

**2. Provjeri kodiranje/dekodiranje**
```python
# Test: enkoduj → dekoduj → uporedi
x_original = (2.5, -3.75)
hromozom = encode(x_original)
x_decoded = decode(hromozom)

assert abs(x_decoded[0] - x_original[0]) < 0.01
assert abs(x_decoded[1] - x_original[1]) < 0.01
```

**3. Testiraj operatore pojedinačno**
```python
# Test ukrštanja
roditelj1 = [1,1,1,1,1,1,1]
roditelj2 = [0,0,0,0,0,0,0]
(dijete1, dijete2) = crossover(roditelj1, roditelj2, t=3)

# Očekivano:
# dijete1 = [1,1,1,0,0,0,0]
# dijete2 = [0,0,0,1,1,1,1]
```

---

## ZAKLJUČAK

Genetički algoritam je **moćan** alat za globalnu optimizaciju, ali **nije magija**. Ključ uspjeha je:

1. **Razumijevanje problema** - Dobro kodiranje rješenja
2. **Pažljivo podešavanje parametara** - Balans između eksploracije i eksploatacije
3. **Praćenje konvergencije** - Rani stopping, vizualizacija
4. **Eksperimentisanje** - Nema "jednog pravog" skupa parametara

**Sretno sa optimizacijom!** 🚀

---

## DODATNI RESURSI

### Knjige
1. Goldberg, D. E. (1989). *"Genetic Algorithms in Search, Optimization and Machine Learning"*
2. Mitchell, M. (1998). *"An Introduction to Genetic Algorithms"*

### Online materijali
- [GA Tutorial - MIT](https://web.mit.edu/6.141/)
- [Genetic Algorithm Visualizer](https://towardsdatascience.com/ga-visualizer)

### Lab fajlovi
- **Lab6.ipynb** - Osnovne klase i operatori (ukrštanje, mutacija)
- **Lab7.ipynb** - Kompletan GA (selekcija, generacije, testiranje)

---

**Autor:** Implementacija za kurs "Optimizacija resursa"
**Datum:** 2024
**Verzija:** 1.0
