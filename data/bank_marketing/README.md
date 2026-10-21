# Bank Marketing duomenys (1.3 užduotis)

- **Šaltinis:** [UCI Machine Learning Repository – Bank Marketing](https://archive.ics.uci.edu/dataset/222/bank+marketing) (tas pats rinkinys yra ir [Kaggle](https://www.kaggle.com/datasets/henriqueyamahata/bank-marketing)).
- **Autoriai:** S. Moro, P. Rita, P. Cortez (2014). *A Data-Driven Approach to Predict the Success of Bank Telemarketing.* Decision Support Systems.
- **Licencija:** CC BY 4.0.
- **Failas:** `bank-full.csv` – 45 211 eilučių, skirtukas `;`, nepakeistas originalas.

| Stulpelis | Aprašymas |
|---|---|
| `age`, `job`, `marital`, `education` | Kliento amžius, profesija, šeiminė padėtis, išsilavinimas |
| `default`, `balance`, `housing`, `loan` | Ar yra pradelstų kreditų, vidutinis sąskaitos likutis (€), būsto ir vartojimo paskola |
| `contact`, `day`, `month` | Paskutinio kontakto būdas ir data |
| `duration` | Paskutinio skambučio trukmė (s). **Nenaudojama** – žinoma tik po skambučio (nutekėjimas) |
| `campaign`, `pdays`, `previous`, `poutcome` | Kontaktų skaičius šioje kampanijoje, dienos nuo ankstesnės kampanijos (−1 – nekontaktuotas), ankstesnių kontaktų skaičius, ankstesnės kampanijos rezultatas |
| `y` | **Tikslas:** ar klientas sutiko su terminuotu indėliu (*yes*/*no*), 11,7 % „yes“ |
