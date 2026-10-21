# 1.1 užduotis – sistemos projektavimo promptas

Šis promptas pateiktas pokalbių DI modeliui. Gautos diagramos: [`docs/diagrams/1_diagramos_PlantUML.md`](../docs/diagrams/1_diagramos_PlantUML.md) ir [`docs/diagrams/2_diagramos_Mermaid.md`](../docs/diagrams/2_diagramos_Mermaid.md).

---

Esi patyręs programų sistemų architektas. Padėk suprojektuoti informacinę sistemą universiteto moduliui, kuriame reikia sukurti informacinę sistemą, naudojančią generatyvųjį dirbtinį intelektą.

**TEMA:** Generatyviuoju dirbtiniu intelektu grįsta adaptyvi sporto treniruočių planavimo informacinė sistema

**SISTEMOS APRAŠYMAS:**
Vartotojas užsiregistruoja ir nurodo savo tikslus (raumenų masės auginimas, ištvermė, svorio metimas ir pan.), fizinio pasirengimo lygį, turimą įrangą (namuose, sporto salėje, be įrangos) ir kiek laiko per savaitę gali skirti treniruotėms. Sistema, naudodama LLM, sugeneruoja struktūrizuotą treniruočių planą (savaitės, treniruotės, pratimai, serijos, pakartojimai, svoriai, poilsio laikas). Po kiekvienos treniruotės vartotojas įveda faktinius rezultatus ir subjektyvų sunkumo įvertinimą (RPE). Sistema analizuoja pažangą ir, naudodama LLM, koreguoja tolimesnį planą: didina ar mažina krūvį, keičia pratimus. Vartotojas gali užduoti klausimus natūralia kalba apie pratimus ar planą.

**TECHNINIAI REIKALAVIMAI:**
- LLM atsakymai turi būti struktūrizuoti (JSON pagal iš anksto apibrėžtą schemą) ir validuojami prieš įrašant į duomenų bazę.
- Pratimai parenkami iš sistemos pratimų katalogo (duomenų bazėje), o ne sugalvojami laisvai.
- Saugoma pilna planų versijų ir koregavimų istorija.
- Sistemos rolės: vartotojas (sportuojantis), administratorius (tvarko pratimų katalogą).

**1 ŽINGSNIS – PlantUML diagramos.**
Sukurk šias diagramas PlantUML sintakse, kiekvieną atskirame kodo bloke su trumpu paaiškinimu:
1. Panaudos atvejų diagrama (visos rolės ir pagrindiniai panaudos atvejai, įskaitant <<include>> ir <<extend>> ryšius, kur tinka).
2. Domeno modelio / klasių diagrama (esybės, atributai su tipais, ryšiai ir kardinalumai).
3. Duomenų bazės ER diagrama (lentelės, pirminiai ir išoriniai raktai).
4. Komponentų / architektūros diagrama (frontend, backend, LLM paslauga, duomenų bazė, JSON validavimo sluoksnis).
5. Sekų diagrama: treniruočių plano generavimas (nuo vartotojo užklausos iki išsaugoto plano, įskaitant LLM kvietimą, JSON validavimą ir pakartotinį bandymą, jei validavimas nepavyksta).
6. Sekų diagrama: plano koregavimas pagal įvestus treniruotės rezultatus.
7. Veiklos diagrama: visas vartotojo ciklas (registracija → tikslai → planas → treniruotė → rezultatai → koregavimas).
8. Būsenų diagrama: treniruočių plano gyvavimo ciklas (pvz., juodraštis, aktyvus, koreguojamas, baigtas, archyvuotas).
9. Diegimo diagrama.

Diagramos turi būti tarpusavyje nuoseklios: tie patys esybių, komponentų ir atributų pavadinimai visose diagramose. Pavadinimai kode – anglų kalba, paaiškinimai – lietuvių kalba. Patikrink, kad PlantUML kodas būtų sintaksiškai teisingas ir sugeneruotųsi be klaidų.

**2 ŽINGSNIS – konvertavimas į Mermaid.**
Kai baigsi PlantUML diagramas, perrašyk visas jas į Mermaid sintaksę (kiekvieną atskirame ```mermaid bloke). Naudok tinkamiausią Mermaid diagramos tipą: classDiagram, erDiagram, sequenceDiagram, stateDiagram-v2, flowchart. Mermaid neturi atskiro panaudos atvejų ir diegimo diagramų tipo, todėl jas atvaizduok per flowchart, kuo artimiau UML žymėjimui, ir trumpai nurodyk, kas prarandama lyginant su PlantUML versija. Patikrink, kad Mermaid kodas būtų suderinamas su naujausia Mermaid versija (pvz., GitHub ar mermaid.live).

Pradėk nuo 1 žingsnio. Prieš pradėdamas, jei sistemos apraše kas nors neaišku, išvardyk savo prielaidas.
