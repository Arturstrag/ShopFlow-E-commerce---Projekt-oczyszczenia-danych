# ShopFlow-E-commerce-Projekt-oczyszczania-danych

## 🎯 Cel projektu

Celem projektu było dokładne oczyszczenie danych sprzedażowych sklepu internetowego **ShopFlow**.  

## Opis projektu
Dane pochodzą z symulowanego sklepu internetowego **ShopFlow**, działającego na rynku polskim w branży e-commerce, obejmującej m.in.:

- modę,
- elektronikę,
- dom i wnętrza,
- urodę.

Zbiór obejmuje około **85 000 rekordów** w 5 powiązanych tabelach i obejmują **24 miesiące historii sprzedaży**:

- klienci – ok. 5 000 rekordów,
- produkty – ok. 2 000 rekordów,
- zamówienia – ok. 20 000 rekordów,
- pozycje zamówień – ok. 49 000 rekordów,
- stany magazynowe – 2 000 rekordów.

## 🧹 Czyszczenie i przygotowanie danych

Przed rozpoczęciem właściwej analizy przeprowadzono kontrolę jakości danych we wszystkich tabelach.

Zidentyfikowano m.in.:
- braki danych,
- duplikaty,
- niespójne formaty,
- błędne wartości,
- literówki,
- wartości odstające,
- problemy z integralnością danych.

Problemy te zostały przeanalizowane i naprawione przy użyciu SQL oraz PostgreSQL.

---

### 👤 Tabela `customers`

W danych klientów zidentyfikowano następujące problemy: 

#### 1. Niespójne formaty numerów telefonów

Numery telefonów występowały w wielu różnych formatach, np. z prefiksem `+48`, bez prefiksu, ze spacjami lub myślnikami.

![Różne formaty telefonów](image/surowe_dane/telefony_rozne_formaty_customers.png)

Numery zostały ustandaryzowane do jednego formatu:

`+48XXXXXXXXX`

---

#### 2. Niepoprawne kody pocztowe

Część kodów pocztowych nie była zgodna z polskim formatem `XX-XXX`.

![Niepoprawne kody pocztowe](images/kody_pocztowe_customers.png)

Rekordy, których nie można było jednoznacznie poprawić, zostały oznaczone do dalszej weryfikacji.

---

#### 3. Niespójne adresy e-mail

W adresach e-mail występowały:
- różnice w wielkości liter,
- zbędne spacje,
- brak jednolitego formatu.

![Niespójne adresy e-mail](images/Emaile_customers.png)

Adresy zostały ujednolicone przy użyciu funkcji `LOWER()` oraz `TRIM()`.

---

#### 4. Literówki i niespójne nazwy miast

W kolumnie `city` występowały różne warianty tej samej miejscowości, np. literówki oraz problemy z wielkością liter.

![Niespójne nazwy miast](images/miasta_customers.png)

Utworzono tabelę mapującą, która pozwoliła przypisać błędne warianty do poprawnych nazw miast.

---

#### 5. Duplikaty klientów

Wykryto rekordy klientów posiadających ten sam znormalizowany adres e-mail.

![Duplikaty klientów](images/duplikaty_klientów_customers.png)

Przed usunięciem duplikatów historia zamówień została przypisana do właściwego rekordu klienta.

---

#### 6. Dodatkowe duplikaty z sufiksem `_dup`

Część duplikatów posiadała zmodyfikowany adres e-mail, np.:

`jan.kowalski@gmail.com`  
`jan.kowalski_dup@gmail.com`

![Duplikaty z sufiksem dup](images/dodatkowe_duplikaty_customers.png)

Takie rekordy wymagały osobnej reguły identyfikacji i scalania.

---

### 📦 Tabela `products`

W tabeli produktów problemy dotyczyły przede wszystkim cen oraz niespójnych kategorii produktowych.

#### 1. Produkty z ceną równą 0

W katalogu znaleziono produkty, których `unit_price` wynosiło `0`.

![Produkty z ceną 0](images/cena_0_products.png)

Rekordy nie zostały automatycznie usunięte. Zostały oznaczone do weryfikacji, ponieważ bez dodatkowych informacji nie można było jednoznacznie stwierdzić, czy cena była błędem.

---

#### 2. Niespójne kategorie produktów

Nazwy kategorii występowały w różnych wariantach, m.in.:
- różna wielkość liter,
- polskie i angielskie nazwy,
- dodatkowe spacje,
- różne warianty tej samej kategorii.

![Niespójne kategorie produktów](images/kategorie_products.png)

Kategorie zostały zmapowane do jednolitego zestawu wartości.

---

#### 3. Wartości odstające cen produktów

W danych występowały produkty o cenach znacznie wyższych niż typowe ceny w danej kategorii.

![Produkty z nietypową ceną](images/produkty_z_dziwną_ceną.png)

Potencjalne wartości odstające zostały wykryte metodą IQR.

Nie zostały automatycznie usunięte, ponieważ wysoka cena nie musi oznaczać błędu danych i w rzeczywistym projekcie wymagałaby dodatkowej weryfikacji biznesowej.

---

### 🧾 Tabela `orders`

W tabeli zamówień zidentyfikowano problemy związane z duplikacją rekordów oraz formatem dat.

#### 1. Duplikaty zamówień

Wykryto klientów posiadających więcej niż jedno zamówienie o tej samej dacie.

![Duplikaty zamówień](images/duplikaty_zamówień_orders.png)

Duplikaty zostały zidentyfikowane przy użyciu `ROW_NUMBER()`.

Przed usunięciem rekordów sprawdzono również powiązania z tabelą `order_items`, aby zachować integralność danych.

---

#### 2. Niespójne formaty dat

Daty zamówień występowały w kilku formatach, np.:

- `08.09.2024`
- `26/12/2024`
- `2024-12-26`

![Różne formaty dat](images/rozne_formaty_dat_orders.png)

Wszystkie daty zostały ujednolicone do formatu ISO:

`YYYY-MM-DD`

Następnie kolumna została przekonwertowana do typu `DATE`.

---

### 🛒 Tabela `order_items`

#### 1. Ujemne wartości `quantity`

W tabeli pozycji zamówień występowały rekordy z ujemną liczbą produktów.

![Ujemne quantity](images/ujemne_quantity_order_items.png)

Ujemne wartości zostały zinterpretowane jako zwroty zapisane w niewłaściwym miejscu.

Zamiast zmieniać je na wartości dodatnie, utworzono osobną tabelę `returns`, do której przeniesiono informacje o zwrotach.

---

### 🏭 Tabela `inventory`

W danych magazynowych zidentyfikowano problemy związane ze stanami magazynowymi oraz formatem lokalizacji.

#### 1. Ujemne stany magazynowe

W kolumnie `stock_quantity` występowały wartości poniżej zera.

![Ujemne stany magazynowe](images/ujemne_stock_quantity_inventory.png)

Ujemne wartości zostały zastąpione wartością `0`, ponieważ fizyczny stan magazynowy nie może być ujemny.

---

#### 2. Zbędne spacje w lokalizacji magazynowej

W kolumnie `warehouse_location` występowały dodatkowe spacje.

![Spacje w lokalizacji magazynowej](images/spacje_warehouse_inventory.png)

Dane zostały oczyszczone przy użyciu funkcji `TRIM()`.

---

## ✅ Efekt czyszczenia danych

Po zakończeniu procesu dane zostały przygotowane do dalszej analizy:

- ujednolicono formaty danych,
- usunięto lub scalono duplikaty,
- poprawiono niespójne wartości tekstowe,
- oddzielono zwroty od sprzedaży,
- ujednolicono daty,
- zidentyfikowano wartości odstające,
- zachowano integralność pomiędzy powiązanymi tabelami.

Dopiero po zakończeniu tego etapu rozpoczęto właściwą analizę biznesową.
