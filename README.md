# **Nienadzorowana Detekcja Anomalii oraz Analiza Sprawiedliwości** 

# **Algorytmicznej w Wykrywaniu Oszustw Bankowych** 

#### **STRESZCZENIE** 

W praktyce sektora finansowego identyfikacja wniosków o otwarcie konta stanowiących oszustwo 

charakteryzuje się opóźnioną w czasie, kosztowną oraz obarczoną błędem selekcji próby etykietyzacją klas. Klasyczne modele uczenia nadzorowanego dziedziczą obciążenia historycznych decyzji oraz wykazują ograniczoną elastyczność w detekcji nowych wzorców przestępczych. W niniejszej pracy przeprowadzono systematyczne badanie porównawcze sześciu algorytmów nienadzorowanej detekcji anomalii oraz autorskiego modelu zespołowego ( _ensemble_ ) na benchmarkowym zbiorze danych _Bank Account Fraud (BAF)_ NeurIPS 2022 (N = 1 000 000). Analiza obejmuje zagadnienia   efektywności   predykcyjnej,   podatności   modeli   na   dryf   koncepcyjny,   ewaluację sprawiedliwości algorytmicznej ( _fairness_ ) w kontekście obciążenia wiekowego ( _age bias_ ) oraz ocenę zgodności z wymogami regulacyjnymi (RODO, EU AI Act). 

## **1. Wprowadzenie i Cel Badania** 

Procedury weryfikacji tożsamości oraz oceny ryzyka przy otwieraniu rachunków bankowych stanowią fundamentalny element ochrony instytucji finansowych przed stratami wywołanymi przestępczością tożsamościową. Stosowanie klasycznych algorytmów uczenia nadzorowanego w systemach produkcyjnych ograniczone jest trzema kluczowymi zjawiskami: 

**Opóźnienie etykiety klasowej (Label Delay): Potwierdzenie faktu popełnienia oszustwa następuje zazwyczaj ze znacznym opóźnieniem czasowym (od kilku tygodni do kilku miesięcy).** 

- **Sample Selection   Bias: Etykiety wskazujące na fraud są znane** 

**wyłącznie przez wnioski zaakceptowane przez dotychczasowe systemy decyzyjne.** 

- **Concept Drift: Przestępcy stale modyfikują wzorce działań w czasie.** 

**Cel niniejszej pracy stanowi ewaluacja użyteczności wybranych modeli nienadzorowanych** 

**jako wstępnego kroku w procesie filtrowania podejrzanych transakcji w danych bankowych.** 

## **2. Metodologia i Przygotowanie Danych** 

### **2.1. Zbiór Danych BAF (NeurIPS 2022)** 

**Ewaluację przeprowadzono na oficjalnym zbiorze danych** **_Bank Account Fraud (BAF)_ zaprezentowanym podczas konferencji NeurIPS 2022. Zbiór obejmuje 1 000 000 wniosków o otwarcie konta bankowego. Odsetek klasowo zrównoważonych oszustw (zmienna `fraud_bool`) wynosi ~1,1%.** 

### **2.2. Temporalny Podział Czasowy (Out-of-Sample Split)** 

1 

**W celu zapobiegania zjawisku wycieku informacji z przyszłości (data leakage) oraz odzwierciedlenia realnych** 

**warunków wdrożenia produkcyjnego, dane podzielono według kryterium czasowego (`month`):** 

- **Zbiór Treningowy (Train):** Miesiące 0 – 5 (dane historyczne, N = 600 000 obserwacji).** 

- **Zbiór Testowy (Test OOS):** Miesiące 6 – 7 (przyszłe obserwacje, N = 400 000 obserwacji).** 

**Wykluczono całkowicie zmienną `device_fraud_count`, stanowiącą agregację wyników z wcześniejszych procedur sprawdzających (zjawisko** **_near-label leakage_ ).** 

### **2.3. Transformacje Preprocessingowe (Receptura A2)** 

**Zaprojektowano i przetestowano dedykowany schemat transformacji zmiennych wejściowych:** 

**1. Kodowanie Braków Danych (MNAR): Brak danych w zmiennej `prev_address_months_count` występował w 92,2% wniosków oszukańczych wobec 71,3% w populacji ogólnej. Zamiast imputacji wartością środkową, wygenerowano binarne flagi obecności braku (`na_flag_*`), przekształcając brak danych w silny sygnał predykcyjny.** 

**2. Frequency   Encoding:  Zmienne   kategoryczne   o   wysokiej   kardynalności** 

- **(`device_os`, `housing_status`) zakodowano częstością ich występowania na zbiorze treningowym.** 

**3. Zmienne   Relacyjne: Utworzono   cechy   złożone   mierzące   proporcje   ryzyka   do   dochodu (`risk_to_income`), stabilności zamieszkania do wieku (`address_stability`) oraz wnioskowanego limitu do dochodu (`loan_to_income`).** 

**4. Selekcja Wymiarowości (Receptura A2): Rezygnacja z nieliniowych przekształceń Yeo-Johnsona na rzecz wyselekcjonowanego podzbioru 8 zmiennych cechujących się najwyższą stabilnością macierzy kowariancji.** 

## **3. Ewaluowane Metody Nienadzorowane** 

**W badaniu poddano analizie 6 zróżnicowanych algorytmów oraz model hybrydowy:** 

**1. Odległość Mahalanobisa: Parametryczna metoda mierząca odległość punktu od wielowymiarowego środka ciężkości rozkładu z uwzględnieniem macierzy kowariancji cech (z regularyzacją Ridge).** 

**2. Isolation Forest: Zespół drzew losowych, w którym miarą anomalii jest średnia długość ścieżki potrzebna do odizolowania danej obserwacji.** 

**3. Local   Outlier   Factor   (LOF): Nieparametryczny   algorytm   gęstościowy   wyznaczający   lokalną   względną rzadkość otoczenia punktu w oparciu o** **_k_ -najbliższych sąsiadów.** 

**4. Głęboki  Autoenkoder  (H2O  Deep  Learning): Architektura  sieci  neuronowej  redukująca  i  rekonstruująca sygnał; miarą anomalii jest błąd średniokwadratowy rekonstrukcji (MSE).** 

**5. K-Means Clustering: Algorytm grupowania (dla  k=3), gdzie miarą anomalii jest odległość euklidesowa od najbliższego centroidu.** 

**6. One-Class   SVM   (OCSVM): Jądrowa   metoda   wyznaczania   optymalnej   hiperpowierzchni   ograniczającej rozkład populacji wzorcowej z jądrem RBF (** **_ν=0,011_ ).** 

**7. Model   Zespołowy   (Ensemble   Pipeline): Ważona   agregacja   znormalizowanych   rang ocen generowanych przez Isolation Forest, Autoenkoder oraz OCSVM.** 

## **4. Wyniki Empiryczne i Dyskusja** 

2 

**Tabela 1. Efektywność predykcyjna badanych modeli nienadzorowanych na zbiorze testowym Out-of-Sample (Miesiące 6–7)** 

|**Model / Algorytm**|**AUC-ROC**|**Recall @ 1%**|**Recall @ 5%**|**Recall @ 10%**|**Lift @ 1%**|**Lift @ 10%**|
|---|---|---|---|---|---|---|
|**Mahalanobis (Receptura A2)***|**0,653**|**0,019**|**0,097**|**0,171**|**1,9×**|**1,71×**|
|**Autoenkoder Głęboki (H2O)**|**0,630**|**0,025**|**0,089**|**0,166**|**2,5×**|**1,66×**|
|**Isolation Forest**|**0,613**|**0,022**|**0,097**|**0,173**|**2,2×**|**1,73×**|
|**One-Class SVM (RBF)**|**0,583**|**0,015**|**0,098**|**0,173**|**1,5×**|**1,73×**|
|**LOF (bez cechy wieku)***|**0,581**|**0,027**|**0,106**|**0,181**|**2,7×**|**1,81×**|
|**K-Means (****_k=3_)**|**0,543**|**0,007**|**0,093**|**0,162**|**0,7×**|**1,62×**|
|**Model Zespołowy (Ensemble)**|**0,625**|**0,028**|**0,099**|**0,183**|**2,8×**|**1,83×**|



### **Kluczowe Wnioski z Ewaluacji Empirycznej:** 

- **Modele nienadzorowane osiągają wartości AUC-ROC w granicach 0,54–0,65,** 

- **co czyni je nieodpowiednimi jako samodzielne systemy decyzyjne (w porównaniu do modeli nadzorowanych osiągających AUC 0,82–0,88). Zapewniają one jednak wysoki współczynnik podbicia Lift@1% w przedziale 1,9× – 2,8×, co pozycjonuje je jako doskonałe narzędzia wstępnej filtracji i priorytetyzacji obciążenia analityków.** 

- **Ograniczenie liczby cech do 8 najistotniejszych wymiarów zwiększyło** 

- **wskaźnik AUC Odległości Mahalanobisa z 0,564 do 0,653 (+8,9 p.p.), przesuwając tę klasyczną metodę na pozycję lidera pojedynczych estymatorów.** 

- **Model   zespołowy   zoptymalizował   stabilność   wskaźników   detekcji   w najwyższym decylu ryzyka, osiągając najwyższy współczynnik Recall@1% (2,8%) oraz Lift@1% (2,8×).** 

## **5. Analiza Zaawansowana** 

### **5.1. Sprawiedliwość Algorytmiczna i Dyskryminacja Wiekowa** 

**Przeprowadzona analiza odsetka fałszywych alarmów (FPR) w poszczególnych podgrupach wiekowych wykazała występowanie znacznego age bias:** 

- **Wnioskodawcy z grupy <30 lat generowali najwięcej bezwzględnych oszustw (n=1045), przy czym modele dystansowe uzyskiwały dla nich niski wskaźnik detekcji (Recall@5% = 6,9%–8,0%).** 

- **Wnioskodawcy z grupy 60+ przy niewielkim udziale w przestępczości (n=68) uzyskiwali drastycznie zawyżony współczynnik wykrywalności (Recall@5% = 19,1%–21,4%).** 

**W przypadku algorytmu LOF obecność zmiennej `customer_age` powodowała, że dla klientów w wieku 80+ wskaźnik fałszywych alarmów wynosił FPR = 1,00 (100%). Metody dystansowe traktują wiek podeszły jako rzadkość statystyczną, a nie rzeczywisty wskaźnik ryzyka. Usunięcie zmiennej wieku z przestrzeni wejściowej LOF pozwoliło na wyeliminowanie skrzywienia demograficznego bez utraty zdolności predykcyjnej.** 

### **5.2. Analiza Dryfu Koncepcyjnego (Concept Drift)** 

3 

**W celu oceny stabilności rozkładu zmiennych w czasie skonstruowano klasyfikator przeciwniczy (** **_Adversarial Discriminator_ bazujący na modelu XGBoost) trenowany na zadaniu rozróżniania próbek pochodzących z miesięcy 0–5 od miesięcy 6–7.** 

**Uzyskany wynik AUC-ROC równy 0,9986 potwierdził obecność niemal całkowitego przesunięcia rozkładu danych w czasie. Analiza ważności zmiennych wykazała, że głównym nośnikiem dryfu były skumulowane zmienne dynamiczne z grupy `velocity_*` (np. `velocity_4w`). Usunięcie cech typu** **_velocity_ zwiększyło stabilność czasową algorytmu Isolation Forest, podnosząc wskaźnik AUC na zbiorze testowym o +0,0016.** 

### **5.3. Profilowanie Archetypów Oszustw (Segmentacja K-Means)** 

**Zastosowanie algorytmu klastrowania K-Means na próbie potwierdzonych oszustw pozwoliło na wyodrębnienie 3 głównych profilów operacyjnych:** 

- **Archetyp   I   (59,0%   przestępstw): Klienci   w   średnim   wieku   o   umiarkowanym   wskaźniku   ryzyka   i ugruntowanej historii. Wzorzec odpowiadający kradzieży tożsamości (** **_Identity Theft_ ).** 

- **Archetyp   II   (33,4%   przestępstw): Młodzi   wnioskodawcy   o   niskim   dochodzie   i   braku   historii.   Wzorzec charakterystyczny dla słupów finansowych oraz zautomatyzowanych botów (** **_Money Mules / Botnets_ ).** 

- **Archetyp III (7,6% przestępstw): Wnioski o wysokim deklarowanym dochodzie i niskim wskaźniku ryzyka. Wzorzec właściwy dla oszustw syntetycznych oraz wyłudzeń limitów (** **_Bust-out Fraud_ ).** 

## **6. Aspekty Regulacyjne (RODO i EU AI Act)** 

**Zastosowanie nienadzorowanych modeli detekcji anomalii w sektorze bankowym podlega ścisłym regulacjom prawnym:** 

**1. Art.  22  RODO  (GDPR): Zakazuje  wydawania  w  pełni  zautomatyzowanych  decyzji  wywołujących  skutki prawne lub w podobny sposób istotnie wpływających na osobę. Z uwagi na umiarkowany poziom precyzji, modele nienadzorowane muszą działać wyłącznie w trybie nadzorowanym (** **_Human-in-the-loop_ ), wspierając priorytetyzację zadań dla analityków.** 

**2. Akt w sprawie AI (EU AI Act 2024): Systemy klasyfikacji ryzyka oraz detekcji oszustw finansowych zaliczane są do kategorii systemów wysokiego ryzyka (** **_High-Risk AI Systems_ ). Wymagają one ciągłego audytu pod kątem braku dyskryminacji pośredniej (np. wiekowej), monitorowania dryfu danych oraz zapewnienia wytycznych interpretowalności.** 

## **7. Podsumowanie** 

**Przeprowadzone badania wskazują, że metody nienadzorowanego uczenia maszynowego stanowią skuteczne wsparcie operacyjne w wykrywaniu oszustw bankowych, generując do 2,8-krotnego podbicia wykrywalności (Lift@1%) w stosunku do losowej inspekcji. Kluczowym czynnikiem sukcesu wdrożeniowego okazuje się odpowiednia inżynieria cech (kodowanie braków danych MNAR) oraz dbałość o sprawiedliwość algorytmiczną i czasową stabilność modeli w warunkach zmieniających się wzorców przestępczości.** 

4 

