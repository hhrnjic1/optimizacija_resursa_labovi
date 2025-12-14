# IZVJEŠTAJ SA LABORATORIJSKE VJEŽBE 6

## Genetički algoritam (prvi dio)

---

**Predmet:** Optimizacija resursa
**Laboratorijska vježba:** 6
**Akademska godina:** 2024/2025

**Univerzitet u Sarajevu**
**Elektrotehnički fakultet**

---

## 1. UVOD

### 1.1 Cilj vježbe

Cilj ove laboratorijske vježbe je implementacija osnovnih komponenti genetičkog algoritma (GA) - evolutivnog optimizacionog algoritma inspirisanog prirodnom selekcijom i genetikom. Ovo je prvi dio implementacije gdje se fokusiramo na:

- Kreiranje strukture individua i populacije
- Implementaciju osnovnih genetičkih operatora (ukrštanje i mutacija)
- Testiranje funkcionalnosti implementiranih komponenti

### 1.2 Teorijska osnova

**Genetički algoritam** je metaheuristički algoritam koji imitira proces prirodne evolucije. Osnovna ideja je da se problem predstavi kao skup rješenja (individua), gdje svaka individua ima određenu "fitness" vrijednost koja mjeri kvalitet tog rješenja.

#### Osnovni koncepti:

1. **Individua (hromozom)** - Jedno moguće rješenje problema, predstavljeno binarnim nizom
2. **Populacija** - Skup individua (mogućih rješenja)
3. **Fitness funkcija** - Mjera kvaliteta rješenja
4. **Ukrštanje (Crossover)** - Genetički operator koji kombinuje dva roditelja za kreiranje potomaka
5. **Mutacija** - Genetički operator koji nasumično mijenja gene u hromozomu
6. **Selekcija** - Proces odabira najboljih individua za reprodukciju

#### Tok genetičkog algoritma:

```
1. Generiši početnu populaciju (random)
2. Evaluiraj fitness svake individue
3. PONAVLJAJ dok se ne ispuni uslov zaustavljanja:
   a) Selektuj roditelje na osnovu fitnesa
   b) Primijeni ukrštanje → kreiraj potomke
   c) Primijeni mutaciju na potomke
   d) Evaluiraj fitness novih individua
   e) Formiranje nove generacije (zamjena populacije)
4. VRATI najbolju individuu
```

---

## 2. ZADATAK 1: Implementacija klasa

### 2.1 Klasa ApstraktnaIndividua

Implementirana je apstraktna bazna klasa koja predstavlja generičku individuu u genetičkom algoritmu.

#### Atributi:
- `DuzinaHromozoma` (int) - Dužina binarnog niza
- `Hromozom` (list) - Binarni niz (lista 0 i 1)
- `Fitness` (float) - Vrijednost fitnesa individue

#### Metode:
- **Get/Set metode** - Za sve atribute (GetDuzinaHromozoma, GetHromozom, GetFitness, SetDuzinaHromozoma, SetHromozom, SetFitness)
- **Konstruktor** - Prima dužinu hromozoma i generiše slučajan binarni niz
- **Evaluiraj()** - Apstraktna metoda koja mora biti reimplementirana u izvedenim klasama

#### Implementacione odluke:
- Korištena je `ABC` (Abstract Base Class) iz `abc` modula
- Atributi su privatni (prefix `__`) za enkapsulaciju
- Hromozom se generiše korištenjem `random.randint(0, 1)`

### 2.2 Klasa MojaIndividua

Konkretna implementacija individue izvedena iz `ApstraktnaIndividua`.

#### Karakteristike:
- Nasleđuje sve atribute i metode od roditeljske klase
- Reimplementira metodu `Evaluiraj()` (za sada prazna - bit će implementirana u drugom dijelu vježbe)
- Koristi `super().__init__()` za poziv konstruktora roditeljske klase

### 2.3 Klasa Populacija

Najkompleksnija klasa koja upravlja populacijom individua i implementira genetičke operatore.

#### Atributi:
- `VelicinaPopulacije` (int) - Broj individua u populaciji
- `VjerovatnocaUkrstanja` (float) - Vjerovatnoća ukrštanja, interval [0, 1]
- `VjerovatnocaMutacije` (float) - Vjerovatnoća mutacije, interval [0, 1]
- `MaxGeneracija` (int) - Maksimalan broj generacija
- `VelicinaElite` (int) - Broj najboljih individua koje se prenose, interval [0, 2]
- `DuzinaHromozoma` (int) - Dužina hromozoma (podrazumijevana vrijednost: 16)
- `Populacija` (list) - Lista objekata tipa MojaIndividua

#### Get/Set metode:
Implementirane su sve Get i Set metode za navedene atribute sa validacijom parametara.

#### Validacija parametara:

Konstruktor klase `Populacija` provjerava ispravnost svih ulaznih parametara:

1. **VelicinaPopulacije** - Mora biti pozitivan cijeli broj
2. **VjerovatnocaUkrstanja** - Mora biti u intervalu [0, 1]
3. **VjerovatnocaMutacije** - Mora biti u intervalu [0, 1]
4. **MaxGeneracija** - Mora biti pozitivan cijeli broj
5. **VelicinaElite** - Mora biti u intervalu [0, 2]
6. **DuzinaHromozoma** - Mora biti pozitivan cijeli broj

U slučaju neispravnih parametara, baca se izuzetak tipa `Exception` sa odgovarajućom porukom.

---

## 3. GENETIČKI OPERATORI

### 3.1 Operator ukrštanja u jednoj tački (OpUkrstanjaTacka)

#### Algoritam:
```
1. Provjeri vjerovatnoću ukrštanja
   - Ako random() > VjerovatnocaUkrstanja → vrati roditelje
2. Odaberi slučajnu tačku presjecanja (između 1 i duzina-1)
3. Kreiraj dijete1:
   - Geni [0:tacka] od roditelja1
   - Geni [tacka:kraj] od roditelja2
4. Kreiraj dijete2:
   - Geni [0:tacka] od roditelja2
   - Geni [tacka:kraj] od roditelja1
5. Vrati (dijete1, dijete2)
```

#### Primjer:
```
Roditelj 1: [1, 0, 0, 1, 1, 1, 1, 1, 0, 0]
Roditelj 2: [0, 1, 0, 0, 1, 1, 0, 1, 1, 1]
Tačka = 6

Dijete 1:   [1, 0, 0, 1, 1, 1 | 0, 1, 1, 1]
              ←  Roditelj 1  → | ← Roditelj 2 →

Dijete 2:   [0, 1, 0, 0, 1, 1 | 1, 1, 0, 0]
              ←  Roditelj 2  → | ← Roditelj 1 →
```

### 3.2 Operator ukrštanja u dvije tačke (OpUkrstanjaDvijeTacke)

#### Algoritam:
```
1. Provjeri vjerovatnoću ukrštanja
2. Odaberi dvije slučajne sortirane tačke: tacka1 < tacka2
3. Kreiraj dijete1:
   - Geni [0:tacka1] od roditelja1
   - Geni [tacka1:tacka2] od roditelja2
   - Geni [tacka2:kraj] od roditelja1
4. Kreiraj dijete2: obrnuto
5. Vrati (dijete1, dijete2)
```

#### Primjer:
```
Roditelj 1: [1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 1]
Roditelj 2: [0, 0, 1, 1, 0, 0, 1, 0, 1, 1, 0, 0]
Tačka1 = 3, Tačka2 = 8

Dijete 1:   [1, 1, 0 | 1, 0, 0, 1, 0 | 0, 1, 1]
              ← R1 →  |  ←  R2   →   | ← R1 →

Dijete 2:   [0, 0, 1 | 0, 1, 1, 0, 1 | 1, 0, 0]
              ← R2 →  |  ←  R1   →   | ← R2 →
```

### 3.3 Operator binarne mutacije (OpBinMutacija)

#### Algoritam:
```
1. Za svaki gen u hromozomu:
   a) Generiši random broj r ∈ [0, 1)
   b) Ako r < VjerovatnocaMutacije:
      - Flipuj bit: 0 → 1, 1 → 0
2. Vrati mutiranu individuu
```

#### Karakteristike:
- Svaki bit u hromozomu ima nezavisnu vjerovatnoću da bude mutiran
- Mutacija se izvršava "in-place" (mijenja se originalna individua)
- Operacija flip bita: `gen = 1 - gen`

#### Primjer:
```
Prije mutacije:     [1, 0, 0, 1, 1, 1, 0, 1, 1, 1]
                           ↓     ↓  ↓        ↓
Nakon mutacije:     [1, 1, 0, 0, 0, 0, 0, 0, 1, 1]
                        (mutirani geni označeni strelicom)
```

---

## 4. TESTIRANJE

### 4.1 Test 1: Ukrštanje u jednoj tački i mutacija

**Parametri:**
- Veličina populacije: 10
- Vjerovatnoća ukrštanja: 0.99
- Vjerovatnoća mutacije: 0.99 (visoka radi demonstracije)
- Dužina hromozoma: 10

**Rezultat:**
- Uspješno kreirana populacija sa 10 individua
- Ukrštanje u jednoj tački proizvodi dva potomka
- Mutacija uspješno flipuje bitove sa visokom vjerovatnoćom

### 4.2 Test 2: Ukrštanje u dvije tačke

**Parametri:**
- Veličina populacije: 10
- Vjerovatnoća ukrštanja: 1.0 (uvijek se izvršava)
- Vjerovatnoća mutacije: 0.1
- Dužina hromozoma: 12

**Rezultat:**
- Ukrštanje u dvije tačke pravilno kombinuje gene roditelja
- Djeca zadržavaju dijelove oba roditelja u očekivanom rasporedu

### 4.3 Test 3: Validacija parametara

**Testirani scenariji:**

1. **Nevažeća vjerovatnoća ukrštanja (1.5)**
   - ✓ Sistem pravilno baca izuzetak: "Vjerovatnoca mora biti izmedju 0 i 1."

2. **Nevažeća vjerovatnoća mutacije (-0.1)**
   - ✓ Sistem pravilno baca izuzetak: "Vjerovatnoca mora biti izmedju 0 i 1."

3. **Nevažeća veličina elite (5)**
   - ✓ Sistem pravilno baca izuzetak: "VelicinaElite mora biti između 0 i 2."

4. **Validni parametri**
   - ✓ Populacija uspješno kreirana bez grešaka

---

## 5. ANALIZA REZULTATA

### 5.1 Ukrštanje u jednoj tački

Operator ukrštanja u jednoj tački pokazuje sljedeće karakteristike:
- **Prednost**: Jednostavan za implementaciju i razumijevanje
- **Prednost**: Zadržava kontinuirane segmente gena od roditelja
- **Nedostatak**: Ograničena eksploracija prostora rješenja (samo jedna tačka presjecanja)

### 5.2 Ukrštanje u dvije tačke

Operator ukrštanja u dvije tačke:
- **Prednost**: Veća fleksibilnost u kombinovanju gena
- **Prednost**: Omogućava zadržavanje "building blocks" (blokova gena) iz sredine hromozoma
- **Napomena**: Kompleksniji od ukrštanja u jednoj tački, ali pruža bolju eksploraciju

### 5.3 Binarna mutacija

Operator mutacije:
- **Uloga**: Održavanje diverziteta u populaciji
- **Uloga**: Prevencija prerane konvergencije
- **Uloga**: Omogućavanje eksploracije novih dijelova prostora rješenja
- **Parametar**: Vjerovatnoća mutacije obično je niska (0.001 - 0.05)
  - U našem testu je postavljena na 0.99 radi demonstracije efekta

---

## 6. ZAKLJUČAK

### 6.1 Ostvareni rezultati

U ovoj laboratorijskoj vježbi uspješno smo implementirali:

1. **Tri klase:**
   - `ApstraktnaIndividua` - Apstraktna bazna klasa
   - `MojaIndividua` - Konkretna implementacija
   - `Populacija` - Klasa za upravljanje populacijom

2. **Tri genetička operatora:**
   - Ukrštanje u jednoj tački
   - Ukrštanje u dvije tačke
   - Binarna mutacija

3. **Validaciju parametara:**
   - Provjera svih ulaznih parametara
   - Bacanje odgovarajućih izuzetaka

### 6.2 Testirane funkcionalnosti

- ✓ Generisanje slučajne početne populacije
- ✓ Ukrštanje roditelja u jednoj tački
- ✓ Ukrštanje roditelja u dvije tačke
- ✓ Binarna mutacija individua
- ✓ Validacija parametara konstruktora

### 6.3 Budući rad

Ovo je prvi dio implementacije genetičkog algoritma. U drugom dijelu će biti implementirani:

- **Fitness funkcija** - Evaluacija kvaliteta rješenja
- **Selekcija** - Odabir roditelja (rulet selekcija, turnirska selekcija, itd.)
- **Elitizam** - Prenos najboljih individua u narednu generaciju
- **Zamjena populacije** - Formiranje nove generacije
- **Main loop** - Kompletan genetički algoritam sa svim komponentama
- **Optimizacioni problem** - Primjena GA na konkretan problem

### 6.4 Naučene lekcije

1. **Object-Oriented Programming**: Uspješna primjena OOP principa (nasleđivanje, enkapsulacija, apstrakcija)
2. **Validacija ulaza**: Važnost provjere parametara i odgovarajuće obrade grešaka
3. **Genetički algoritmi**: Razumijevanje osnovnih koncepata i operatora
4. **Python sintaksa**: Korištenje ABC modula, list comprehension, private atributa

---

## 7. LITERATURA I REFERENCE

1. Predavanja iz predmeta "Optimizacija resursa"
2. Laboratorijska vježba 6 - Uputstvo
3. Mitchell, M. (1998). "An Introduction to Genetic Algorithms". MIT Press.
4. Goldberg, D. E. (1989). "Genetic Algorithms in Search, Optimization and Machine Learning". Addison-Wesley.
5. Python dokumentacija: https://docs.python.org/3/library/abc.html
6. Python dokumentacija: https://docs.python.org/3/library/random.html

---

## 8. PRILOG - KÔD

Kompletan kôd implementacije nalazi se u fajlu: **Lab6.ipynb**

### Struktura implementacije:

```
Lab6.ipynb
├── Uvod i teorijska osnova (Markdown)
├── Import biblioteka (random, abc)
├── Klasa ApstraktnaIndividua
├── Klasa MojaIndividua
├── Klasa Populacija
│   ├── Konstruktor sa validacijom
│   ├── Get/Set metode
│   ├── OpUkrstanjaTacka()
│   ├── OpUkrstanjaDvijeTacke()
│   └── OpBinMutacija()
├── Test 1: Ukrštanje i mutacija
├── Test 2: Ukrštanje u dvije tačke
├── Test 3: Validacija parametara
└── Zaključak
```

---

**Datum izrade:** 14.12.2024
**Fajlovi:**
- `Lab6.ipynb` - Jupyter Notebook sa kompletnom implementacijom
- `Izvjestaj_Lab6.md` - Ovaj izvještaj (Markdown format)

---

**Napomena:** Ovaj izvještaj može se konvertovati u MS Word format korištenjem Pandoc alata ili direktnim otvaranjem u Wordu.
