0. The goals of phase 2b is to perform benchmarking/scalability tests of sample three-tier lakehouse solution.

1. In main.tf, change machine_type at:

```
module "dataproc" {
  depends_on   = [module.vpc]
  source       = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.36/modules/dataproc"
  project_name = var.project_name
  region       = var.region
  subnet       = module.vpc.subnets[local.notebook_subnet_id].id
  machine_type = "e2-standard-2"
}
```

and subsititute "e2-standard-2" with "e2-standard-4".
ODP 1,2:

Początkowym Regionem naszego projektu byl region = europe-west1, na którym niemożliwe było zwiększenie zasobów, jak opisane w punkcie nr.[2]. Uniemożliwiało to postawienie architektury z zadanymi parametrami i poprawne wykonanie zadania..
  ![image](https://github.com/user-attachments/assets/b1dfd4b8-dc99-4f8d-abc2-2305e997fac8)

Rozwiązaniem okazało się być zmiana regionu na europe-central2, na której bez problemu zwiększono zasoby do 36 CPU's
  <img width="1221" alt="image" src="https://github.com/user-attachments/assets/3e141541-4fd2-41bb-ac32-da1b7c4fad7b" />

Przed rozpoczeciem pracy ustawiono odpowiedni kernel pyspark zgodnie z punktem: [8](https://github.com/bdg-tbd/tbd-workshop-1/tree/v1.0.32#project-setup) 

2. If needed request to increase cpu quotas (e.g. to 30 CPUs): 
https://console.cloud.google.com/apis/api/compute.googleapis.com/quotas?project=tbd-2023z-9918

3. Using tbd-tpc-di notebook perform dbt run with different number of executors, i.e., 1, 2, and 5, by changing:
```
 "spark.executor.instances": "2"
```

in profiles.yml.

4. In the notebook, collect console output from dbt run, then parse it and retrieve total execution time and execution times of processing each model. Save the results from each number of executors.

Poniżej przedstawiono zebrane w tabeli wyniki czasowe wykonania dla każdego z modeli, dla każdego z executorów. Pojedyńcze czasy wykonania dla tabel zostały przedstawione w sekundach, natomaist suma wszystkich tabel została przedstawiona w minutach 
   ![image](https://github.com/user-attachments/assets/464052b0-58d0-4fda-a356-5176d51559e0)

5. Analyze the performance and scalability of execution times of each model. Visualize and discucss the final results.

Wizualne przedstawienie wyników poszczególnych czasów wykonań dla executorów:
   ![image](https://github.com/user-attachments/assets/53fa1fc2-3ce2-4b4f-8eb7-14e9fc986f40)

Wizualne przedstawienie sumarycznych czasów wykonan dla poszczególnych executorów
   ![image](https://github.com/user-attachments/assets/d405aa65-c453-42cf-a818-a5f3c7a48e51)


Wnioski:

Analizując wykres, można zauważyć istotny wzrost efektywności przy niewielkim zwiększeniu liczby executorów, szczególnie w przypadku przejścia z 1 do 2. Odnotowano niemal trzykrotną poprawę czasu wykonania na korzyść większej liczby executorów. Taki wynik był przewidywalny, ponieważ zwiększenie liczby executorów pozwala na bardziej efektywne rozproszenie obciążenia, co znacząco wpływa na skrócenie czasu obliczeń. Jednakże dalsze zwiększanie liczby executorów do 3 i 4 prowadzi, wbrew oczekiwaniom, do wzrostu czasu wykonania.

Patrząc na wykresy czasu wykonania dla poszczególnych modeli, można wyróżnić dwa interesujące przypadki. W pierwszym, czas wykonania zmniejsza się wraz ze wzrostem liczby executorów, co jest zgodne z przewidywaniami. W drugim przypadku czas wykonania osiąga minimum przy liczbie executorów równej 2, a następnie rośnie wraz z dalszym wzrostem ich liczby.

Pierwszy przypadek, w którym czas wykonania maleje przy zwiększaniu liczby executorów, jest w pełni zrozumiały. Większa liczba executorów pozwala na efektywniejsze równoległe przetwarzanie danych, dzięki czemu zadania są realizowane szybciej. Ograniczeniem w takich sytuacjach może być jednak zbyt mała wielkość modelu, przy której użycie zbyt dużej liczby executorów staje się nieoptymalne, ze względu na zwiększony czas komunikacji między nimi.

Drugi przypadek, w którym czas wykonania rośnie przy liczbie executorów większej niż 2, jest bardziej złożony. Można zaobserwować to zarówno dla mniejszych modeli (o czasie wykonania kilku lub kilkunastu sekund), jak i dla bardziej złożonych modeli, których czas wykonania przekracza 600 sekund. Dla małych modeli wzrost czasu może wynikać z faktu, że koszt komunikacji pomiędzy executorami przewyższa korzyści z równoległego przetwarzania. W przypadku modeli bardziej złożonych przyczyną może być charakterystyka danych lub nadmierne obciążenie executorów wynikające z ograniczeń liczby rdzeni. Istnieje także możliwość, że konfiguracja Sparka była nieoptymalna, co prowadziło do niewykorzystania potencjału większej liczby executorów.

Podsumowując, dla analizowanego zestawu modeli najlepszym wyborem okazała się liczba executorów równa 2. Taka konfiguracja zapewniła niemal trzykrotnie lepszy czas wykonania w porównaniu do pojedynczego executora, co było widoczne we wszystkich badanych przypadkach. Analiza pokazała również, że zwiększenie liczby executorów nie zawsze przekłada się na lepszą wydajność. Optymalna liczba executorów powinna być dostosowana do specyfiki przetwarzanych modeli oraz dostępnych zasobów obliczeniowych.




   
