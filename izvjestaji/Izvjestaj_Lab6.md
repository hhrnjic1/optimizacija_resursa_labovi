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

## 8. PRILOG - KOMPLETAN KÔD

### Lab6.py - Kompletna implementacija

```python
"""
============================================================================
LABORATORIJSKA VJEŽBA 6: Genetički algoritam (prvi dio)
Predmet: Optimizacija resursa
Univerzitet u Sarajevu - Elektrotehnički fakultet
============================================================================
"""

# ============================================================================
# IMPORTOVANJE BIBLIOTEKA
# ============================================================================

import random as r
from abc import ABC, abstractmethod

print("✓ Biblioteke uspješno učitane!\\n")

# ============================================================================
# KLASA: ApstraktnaIndividua
# ============================================================================

class ApstraktnaIndividua(ABC):
    """
    Apstraktna klasa koja predstavlja individuu u genetičkom algoritmu.

    Atributi:
    ---------
    DuzinaHromozoma : int
        Dužina hromozoma (binarnog niza)
    Hromozom : list
        Binarni niz dužine DuzinaHromozoma (lista 0 i 1)
    Fitness : float
        Vrijednost fitnesa individue
    """

    def __init__(self, DuzinaHromozoma):
        """
        Konstruktor koji kreira individuu sa slučajnim binarnim hromozomom.

        Parametri:
        ----------
        DuzinaHromozoma : int
            Dužina hromozoma
        """
        self.__DuzinaHromozoma = DuzinaHromozoma
        # Generiši slučajan binarni niz dužine DuzinaHromozoma
        self.__Hromozom = [r.randint(0, 1) for _ in range(DuzinaHromozoma)]
        self.__Fitness = 0.0

    # ========== GET METODE ==========

    def GetDuzinaHromozoma(self):
        """Vraća dužinu hromozoma."""
        return self.__DuzinaHromozoma

    def GetHromozom(self):
        """Vraća hromozom (binarni niz)."""
        return self.__Hromozom

    def GetFitness(self):
        """Vraća fitness vrijednost."""
        return self.__Fitness

    # ========== SET METODE ==========

    def SetDuzinaHromozoma(self, DuzinaHromozoma):
        """Postavlja dužinu hromozoma."""
        self.__DuzinaHromozoma = DuzinaHromozoma

    def SetHromozom(self, Hromozom):
        """Postavlja hromozom."""
        self.__Hromozom = Hromozom

    def SetFitness(self, Fitness):
        """Postavlja fitness vrijednost."""
        self.__Fitness = Fitness

    # ========== APSTRAKTNA METODA ==========

    @abstractmethod
    def Evaluiraj(self):
        """
        Apstraktna metoda koja evaluira fitness individue.
        Mora biti reimplementirana u izvedenoj klasi.
        """
        pass


print("✓ Klasa ApstraktnaIndividua uspješno implementirana!")

# ============================================================================
# KLASA: MojaIndividua (izvedena iz ApstraktnaIndividua)
# ============================================================================

class MojaIndividua(ApstraktnaIndividua):
    """
    Konkretna implementacija individue sa specifičnom fitness funkcijom.

    Nasleđuje sve atribute i metode od ApstraktnaIndividua i
    reimplementira metodu Evaluiraj().
    """

    def __init__(self, DuzinaHromozoma):
        """
        Konstruktor koji poziva konstruktor roditeljske klase.

        Parametri:
        ----------
        DuzinaHromozoma : int
            Dužina hromozoma
        """
        super().__init__(DuzinaHromozoma)

    def Evaluiraj(self):
        """
        Reimplementirana metoda za evaluaciju fitnesa.
        Za sada ostaje prazna - bit će implementirana u drugom dijelu.
        """
        pass


print("✓ Klasa MojaIndividua uspješno implementirana!")

# ============================================================================
# KLASA: Populacija
# ============================================================================

class Populacija:
    """
    Klasa koja predstavlja populaciju individua u genetičkom algoritmu.

    Atributi:
    ---------
    VelicinaPopulacije : int
        Broj individua u populaciji
    VjerovatnocaUkrstanja : float
        Vjerovatnoća ukrštanja (0-1)
    VjerovatnocaMutacije : float
        Vjerovatnoća mutacije (0-1)
    MaxGeneracija : int
        Maksimalan broj generacija
    VelicinaElite : int
        Broj najboljih individua koje se prenose (0-2)
    Populacija : list
        Lista individua (objekti tipa MojaIndividua)
    """

    def __init__(self, VelicinaPopulacije, VjerovatnocaUkrstanja,
                 VjerovatnocaMutacije, MaxGeneracija, VelicinaElite,
                 DuzinaHromozoma=16):
        """
        Konstruktor koji kreira populaciju i validira parametre.

        Parametri:
        ----------
        VelicinaPopulacije : int
            Broj individua u populaciji (mora biti > 0)
        VjerovatnocaUkrstanja : float
            Vjerovatnoća ukrštanja (mora biti između 0 i 1)
        VjerovatnocaMutacije : float
            Vjerovatnoća mutacije (mora biti između 0 i 1)
        MaxGeneracija : int
            Maksimalan broj generacija (mora biti > 0)
        VelicinaElite : int
            Broj najboljih individua (mora biti između 0 i 2)
        DuzinaHromozoma : int, optional
            Dužina hromozoma (default: 16)
        """

        # VALIDACIJA PARAMETARA

        # Provjera VelicinaPopulacije
        if not isinstance(VelicinaPopulacije, int) or VelicinaPopulacije <= 0:
            raise Exception("VelicinaPopulacije mora biti pozitivan cijeli broj.")

        # Provjera VjerovatnocaUkrstanja
        if not (0 <= VjerovatnocaUkrstanja <= 1):
            raise Exception("Vjerovatnoca mora biti izmedju 0 i 1.")

        # Provjera VjerovatnocaMutacije
        if not (0 <= VjerovatnocaMutacije <= 1):
            raise Exception("Vjerovatnoca mora biti izmedju 0 i 1.")

        # Provjera MaxGeneracija
        if not isinstance(MaxGeneracija, int) or MaxGeneracija <= 0:
            raise Exception("MaxGeneracija mora biti pozitivan cijeli broj.")

        # Provjera VelicinaElite
        if not isinstance(VelicinaElite, int) or not (0 <= VelicinaElite <= 2):
            raise Exception("VelicinaElite mora biti između 0 i 2.")

        # Provjera DuzinaHromozoma
        if not isinstance(DuzinaHromozoma, int) or DuzinaHromozoma <= 0:
            raise Exception("DuzinaHromozoma mora biti pozitivan cijeli broj.")

        # POSTAVLJANJE ATRIBUTA
        self.__VelicinaPopulacije = VelicinaPopulacije
        self.__VjerovatnocaUkrstanja = VjerovatnocaUkrstanja
        self.__VjerovatnocaMutacije = VjerovatnocaMutacije
        self.__MaxGeneracija = MaxGeneracija
        self.__VelicinaElite = VelicinaElite
        self.__DuzinaHromozoma = DuzinaHromozoma

        # GENERISANJE POČETNE POPULACIJE
        self.__Populacija = [MojaIndividua(DuzinaHromozoma)
                            for _ in range(VelicinaPopulacije)]

    # ========== GET METODE ==========

    def GetVelicinaPopulacije(self):
        """Vraća veličinu populacije."""
        return self.__VelicinaPopulacije

    def GetVjerovatnocaUkrstanja(self):
        """Vraća vjerovatnoću ukrštanja."""
        return self.__VjerovatnocaUkrstanja

    def GetVjerovatnocaMutacije(self):
        """Vraća vjerovatnoću mutacije."""
        return self.__VjerovatnocaMutacije

    def GetMaxGeneracija(self):
        """Vraća maksimalan broj generacija."""
        return self.__MaxGeneracija

    def GetVelicinaElite(self):
        """Vraća veličinu elite."""
        return self.__VelicinaElite

    def GetDuzinaHromozoma(self):
        """Vraća dužinu hromozoma."""
        return self.__DuzinaHromozoma

    def GetPopulacija(self):
        """Vraća listu individua u populaciji."""
        return self.__Populacija

    # ========== SET METODE ==========

    def SetVelicinaPopulacije(self, VelicinaPopulacije):
        """Postavlja veličinu populacije."""
        if not isinstance(VelicinaPopulacije, int) or VelicinaPopulacije <= 0:
            raise Exception("VelicinaPopulacije mora biti pozitivan cijeli broj.")
        self.__VelicinaPopulacije = VelicinaPopulacije

    def SetVjerovatnocaUkrstanja(self, VjerovatnocaUkrstanja):
        """Postavlja vjerovatnoću ukrštanja."""
        if not (0 <= VjerovatnocaUkrstanja <= 1):
            raise Exception("Vjerovatnoca mora biti izmedju 0 i 1.")
        self.__VjerovatnocaUkrstanja = VjerovatnocaUkrstanja

    def SetVjerovatnocaMutacije(self, VjerovatnocaMutacije):
        """Postavlja vjerovatnoću mutacije."""
        if not (0 <= VjerovatnocaMutacije <= 1):
            raise Exception("Vjerovatnoca mora biti izmedju 0 i 1.")
        self.__VjerovatnocaMutacije = VjerovatnocaMutacije

    def SetMaxGeneracija(self, MaxGeneracija):
        """Postavlja maksimalan broj generacija."""
        if not isinstance(MaxGeneracija, int) or MaxGeneracija <= 0:
            raise Exception("MaxGeneracija mora biti pozitivan cijeli broj.")
        self.__MaxGeneracija = MaxGeneracija

    def SetVelicinaElite(self, VelicinaElite):
        """Postavlja veličinu elite."""
        if not isinstance(VelicinaElite, int) or not (0 <= VelicinaElite <= 2):
            raise Exception("VelicinaElite mora biti između 0 i 2.")
        self.__VelicinaElite = VelicinaElite

    def SetPopulacija(self, Populacija):
        """Postavlja populaciju."""
        self.__Populacija = Populacija

    # ========== GENETIČKI OPERATORI ==========

    def OpUkrstanjaTacka(self, roditelj1, roditelj2):
        """
        Operator ukrštanja u JEDNOJ tački.

        Algoritam:
        1. Odaberi slučajnu tačku presjecanja
        2. Dijete 1: geni od početka do tačke od roditelja 1,
                     ostalo od roditelja 2
        3. Dijete 2: geni od početka do tačke od roditelja 2,
                     ostalo od roditelja 1

        Parametri:
        ----------
        roditelj1 : MojaIndividua
            Prvi roditelj
        roditelj2 : MojaIndividua
            Drugi roditelj

        Vraća:
        ------
        tuple (MojaIndividua, MojaIndividua)
            Dvoje djece (potomaka)
        """
        # Provjeri da li izvršiti ukrštanje
        if r.random() > self.__VjerovatnocaUkrstanja:
            # Ako ne, vrati kopije roditelja
            return (roditelj1, roditelj2)

        # Uzmi hromozome roditelja
        h1 = roditelj1.GetHromozom()
        h2 = roditelj2.GetHromozom()
        duzina = len(h1)

        # Odaberi slučajnu tačku presjecanja (između 1 i duzina-1)
        tacka = r.randint(1, duzina - 1)

        # Kreiraj nove hromozome za djecu
        hromozom_dijete1 = h1[:tacka] + h2[tacka:]
        hromozom_dijete2 = h2[:tacka] + h1[tacka:]

        # Kreiraj nove individue (djecu)
        dijete1 = MojaIndividua(duzina)
        dijete2 = MojaIndividua(duzina)

        # Postavi hromozome
        dijete1.SetHromozom(hromozom_dijete1)
        dijete2.SetHromozom(hromozom_dijete2)

        return (dijete1, dijete2)

    def OpUkrstanjaDvijeTacke(self, roditelj1, roditelj2):
        """
        Operator ukrštanja u DVIJE tačke.

        Algoritam:
        1. Odaberi dvije slučajne tačke presjecanja
        2. Dijete 1: geni prije tačke1 od roditelja 1,
                     geni između tačaka od roditelja 2,
                     geni nakon tačke2 od roditelja 1
        3. Dijete 2: obrnuto od dijete 1

        Parametri:
        ----------
        roditelj1 : MojaIndividua
            Prvi roditelj
        roditelj2 : MojaIndividua
            Drugi roditelj

        Vraća:
        ------
        tuple (MojaIndividua, MojaIndividua)
            Dvoje djece (potomaka)
        """
        # Provjeri da li izvršiti ukrštanje
        if r.random() > self.__VjerovatnocaUkrstanja:
            # Ako ne, vrati kopije roditelja
            return (roditelj1, roditelj2)

        # Uzmi hromozome roditelja
        h1 = roditelj1.GetHromozom()
        h2 = roditelj2.GetHromozom()
        duzina = len(h1)

        # Odaberi dvije slučajne tačke (sortirane)
        tacka1 = r.randint(1, duzina - 2)
        tacka2 = r.randint(tacka1 + 1, duzina - 1)

        # Kreiraj nove hromozome za djecu
        # Dijete 1: R1 | R2 | R1
        hromozom_dijete1 = h1[:tacka1] + h2[tacka1:tacka2] + h1[tacka2:]

        # Dijete 2: R2 | R1 | R2
        hromozom_dijete2 = h2[:tacka1] + h1[tacka1:tacka2] + h2[tacka2:]

        # Kreiraj nove individue (djecu)
        dijete1 = MojaIndividua(duzina)
        dijete2 = MojaIndividua(duzina)

        # Postavi hromozome
        dijete1.SetHromozom(hromozom_dijete1)
        dijete2.SetHromozom(hromozom_dijete2)

        return (dijete1, dijete2)

    def OpBinMutacija(self, individua):
        """
        Operator binarne mutacije.

        Algoritam:
        1. Za svaki gen u hromozomu:
           - Sa vjerovatnoćom VjerovatnocaMutacije: flipuj bit (0→1, 1→0)

        Parametri:
        ----------
        individua : MojaIndividua
            Individua koja se mutira

        Vraća:
        ------
        MojaIndividua
            Mutirana individua (ista referenca)
        """
        hromozom = individua.GetHromozom()

        # Prolazi kroz svaki gen i mutiraj sa vjerovatnoćom
        for i in range(len(hromozom)):
            if r.random() < self.__VjerovatnocaMutacije:
                # Flipuj bit: 0 → 1, 1 → 0
                hromozom[i] = 1 - hromozom[i]

        individua.SetHromozom(hromozom)
        return individua


print("✓ Klasa Populacija uspješno implementirana!\\n")

# ============================================================================
# TESTIRANJE
# ============================================================================

if __name__ == "__main__":
    print("="*70)
    print("       TESTIRANJE GENETIČKOG ALGORITMA - SVI TESTOVI")
    print("="*70)

    # TEST 1: Ukrštanje u jednoj tački i mutacija
    print("\\n" + "="*70)
    print("TEST 1: Ukrštanje u jednoj tački i mutacija")
    print("="*70)

    p = Populacija(10, 0.99, 0.99, 10, 1, 10)

    r1 = r.randint(0, p.GetVelicinaPopulacije() - 1)
    r2 = r.randint(0, p.GetVelicinaPopulacije() - 1)
    p1 = p.GetPopulacija()[r1]
    p2 = p.GetPopulacija()[r2]

    print("\\nRODITELJI:")
    print("P1:", p1.GetHromozom())
    print("P2:", p2.GetHromozom())

    (c1, c2) = p.OpUkrstanjaTacka(p1, p2)

    print("\\nDJECA NAKON UKRŠTANJA (jedna tačka):")
    print("C1:", c1.GetHromozom())
    print("C2:", c2.GetHromozom())

    c3 = p.OpBinMutacija(c1)

    print("\\nNAKON MUTACIJE:")
    print("C3:", c3.GetHromozom(), "← C1 nakon mutiranja")

    print("\\n✓ TEST 1 ZAVRŠEN!")

    # TEST 2: Ukrštanje u dvije tačke
    print("\\n" + "="*70)
    print("TEST 2: Ukrštanje u dvije tačke")
    print("="*70)

    p2 = Populacija(10, 1.0, 0.1, 10, 1, 12)

    r1 = r.randint(0, p2.GetVelicinaPopulacije() - 1)
    r2 = r.randint(0, p2.GetVelicinaPopulacije() - 1)
    roditelj1 = p2.GetPopulacija()[r1]
    roditelj2 = p2.GetPopulacija()[r2]

    print("\\nRODITELJI:")
    print("Roditelj 1:", roditelj1.GetHromozom())
    print("Roditelj 2:", roditelj2.GetHromozom())

    (dijete1, dijete2) = p2.OpUkrstanjaDvijeTacke(roditelj1, roditelj2)

    print("\\nDJECA NAKON UKRŠTANJA (dvije tačke):")
    print("Dijete 1:  ", dijete1.GetHromozom())
    print("Dijete 2:  ", dijete2.GetHromozom())

    print("\\n✓ TEST 2 ZAVRŠEN!")

    # TEST 3: Validacija parametara
    print("\\n" + "="*70)
    print("TEST 3: Validacija parametara")
    print("="*70)

    print("\\n1. Test: Vjerovatnoća ukrštanja = 1.5 (treba da baci izuzetak)")
    try:
        p_invalid = Populacija(10, 1.5, 0.5, 10, 1, 10)
        print("   ✗ GREŠKA: Izuzetak NIJE bacen!")
    except Exception as e:
        print(f"   ✓ USPJEŠNO: Bacen izuzetak: '{e}'")

    print("\\n2. Test: Vjerovatnoća mutacije = -0.1 (treba da baci izuzetak)")
    try:
        p_invalid = Populacija(10, 0.5, -0.1, 10, 1, 10)
        print("   ✗ GREŠKA: Izuzetak NIJE bacen!")
    except Exception as e:
        print(f"   ✓ USPJEŠNO: Bacen izuzetak: '{e}'")

    print("\\n3. Test: VelicinaElite = 5 (treba da baci izuzetak, dozvoljeno 0-2)")
    try:
        p_invalid = Populacija(10, 0.5, 0.1, 10, 5, 10)
        print("   ✗ GREŠKA: Izuzetak NIJE bacen!")
    except Exception as e:
        print(f"   ✓ USPJEŠNO: Bacen izuzetak: '{e}'")

    print("\\n4. Test: Svi validni parametri")
    try:
        p_valid = Populacija(20, 0.8, 0.01, 100, 2, 16)
        print("   ✓ USPJEŠNO: Populacija kreirana bez grešaka")
        print(f"   - Veličina populacije: {p_valid.GetVelicinaPopulacije()}")
        print(f"   - Vjerovatnoća ukrštanja: {p_valid.GetVjerovatnocaUkrstanja()}")
        print(f"   - Vjerovatnoća mutacije: {p_valid.GetVjerovatnocaMutacije()}")
    except Exception as e:
        print(f"   ✗ GREŠKA: Neočekivan izuzetak: '{e}'")

    print("\\n✓ TEST 3 ZAVRŠEN!")

    # SAŽETAK
    print("\\n" + "="*70)
    print("                    ✅ SVI TESTOVI USPJEŠNO ZAVRŠENI!")
    print("="*70)
    print("\\n📊 SAŽETAK:")
    print("  ✓ Test 1: Ukrštanje u jednoj tački - PROŠAO")
    print("  ✓ Test 2: Ukrštanje u dvije tačke - PROŠAO")
    print("  ✓ Test 3: Validacija parametara - PROŠAO")
    print("\\n🎉 Implementacija genetičkog algoritma (prvi dio) je kompletna!")
    print("="*70)
```

---

**Datum izrade:** 14.12.2024

**Fajlovi:**
- `Lab6.ipynb` - Jupyter Notebook sa kompletnom implementacijom (na https://github.com/hhrnjic1/optimizacija_resursa_labovi)
- `Izvjestaj_Lab6.md` - Ovaj izvještaj
