# UAAS — Kutatási és fejlesztési roadmap

**Projekt:** Uncertainty-Aware Adaptive Screening  
**Verzió:** 0.1  
**Dátum:** 2026-10-10  
**Státusz:** előzetes ütemterv

> Ez a roadmap sorrendet, kimeneteket és elfogadási feltételeket rögzít; nem vállal naptári határidőket. Az ütemezést a JEV állapota, az adatforrások hozzáférhetősége és a rendelkezésre álló erőforrások ismeretében kell konkretizálni.

## Cél

A UAAS célja annak megvizsgálása, hogy a becsült bizonytalanság és a hibaköltség alapján vezérelt útválasztás csökkentheti-e a szisztematikus szakirodalmi áttekintések cím- és absztraktszűrésének teljes költségét egy előre meghatározott recall-korlát megtartása mellett.

A részletes kérdéseket, hipotéziseket, adatkezelést és kísérleti tervet a [`docs/research_protocol.md`](docs/research_protocol.md) rögzíti.

## Fázisok és elfogadási feltételek

| Fázis | Feladatok | Kimenet / elfogadási feltétel |
|---|---|---|
| **F0 — Specifikáció** | JEV technikai specifikációja; célváltozó, döntési költségek és adatlicenc-szempontok tisztázása. | Jóváhagyott projektdefiníció és mérési terv. |
| **F1 — Adatvalidáció** | 2–3 benchmark vizsgálata; tényleges letöltés, séma, címkék és licenc ellenőrzése; feltáró adatelemzés. | Reprodukálható adatbetöltő, adatlap és dokumentált adatfelosztás. |
| **F2 — Baseline-ok** | TF-IDF-os klasszikus klasszifikátor és statikus LLM; standard metrikák és költségnaplózás. | Baseline-jelentés és futtatható pipeline. |
| **F3 — JEV és kalibráció** | JEV-integráció; kalibrációs eljárás; validációs küszöbök meghatározása. | Mérhető és dokumentált kockázati pontszám. |
| **F4 — Adaptív útválasztás** | Gyors út, mélyebb LLM-elemzés, tartózkodás és emberi ellenőrzés. | A UAAS első működő verziója. |
| **F5 — Összehasonlító teszt** | Előre rögzített tesztfuttatás, review-szintű elemzés és érzékenységvizsgálat. | Eredménytáblák és korlátelemzés. |
| **F6 — Prototípus és dokumentáció** | Egyszerű UI/CLI, auditnapló, futtatási útmutató, model card és adatlap. | Bemutatható, megismételhető demonstrátor. |
| **F7 — Publikálható csomag** | Kód- és adatlicenc-ellenőrzés, preregisztráció/protokoll és jelentés. | Reprodukálható kutatási csomag és publikációs terv. |

## Fő kapuk a következő szakasz előtt

### F0 → F1: specifikáció

- [ ] A JEV pontos definíciója, bemenete és kimenete dokumentált.
- [ ] Tisztázott, hogy a JEV pontszáma kalibrált valószínűség vagy csak rangsorolási érték.
- [ ] A téves kizárás, téves beválasztás, LLM-hívás és emberi felülvizsgálat költsége rögzített.
- [ ] Az elsődleges recall-korlát a végső tesztadatok megtekintése előtt kiválasztott.

### F1 → F2: adatvalidáció

- [ ] Legalább két lehetséges benchmark hozzáférése, licencfeltétele és sémája ellenőrzött.
- [ ] Rögzített adatpillanatkép, forrás, letöltési dátum és verzió elmentve.
- [ ] Deduplikáció ellenőrizve DOI/rekordazonosító és normalizált cím alapján.
- [ ] Tanító-, validációs és teszthalmaz teljes review-k szerint különválasztva.
- [ ] Hiányzó absztraktok és címkézési eltérések dokumentálva.

### F2 → F3: baseline-ok

- [ ] M0 klasszikus klasszifikátor reprodukálható futtatása elkészült.
- [ ] M1 statikus LLM-baseline azonos tesztrekordokon fut.
- [ ] A prompt-, modell-, paraméter- és költségadatok naplózva.
- [ ] A baseline-eredmények és fő hibamódok dokumentálva.

### F3 → F4: JEV és kalibráció

- [ ] A JEV interfésze tesztekkel lefedett.
- [ ] A kockázati pontszám értelmezése dokumentált.
- [ ] A kalibrációt kizárólag a tanító/validációs folyamatban állítják be.
- [ ] A küszöbök nem a végső teszthalmazon kerülnek kiválasztásra.

### F4 → F5: adaptív útválasztás

- [ ] A gyors, mélyebb és emberi útvonal döntési szabálya dokumentált.
- [ ] A rendszer támogatja a tartózkodást és a manuális felülvizsgálatot.
- [ ] A döntések, evidenciarészletek, verziók, idő- és költségadatok auditálhatók.

### F5 → F6/F7: értékelés és csomagolás

- [ ] Az elsődleges végpont és a recall-korlát teljesülését előre rögzített szabály szerint értékelték.
- [ ] Review-szintű bizonytalanság és témakörök közötti általánosíthatóság elemzése elkészült.
- [ ] A negatív eredmények és a korlátok is dokumentáltak.
- [ ] A licenc és a megosztható adatok ellenőrzöttek.

## Javasolt repository-struktúra

```text
project_uaas/
├── README.md
├── ROADMAP.md
├── docs/
│   ├── research_protocol.md
│   ├── data_card.md
│   ├── model_card.md
│   └── experiment_log.md
├── configs/
│   ├── benchmark.yaml
│   └── routing_policy.yaml
├── data/
│   ├── raw/          # csak akkor, ha a licenc engedi
│   ├── interim/
│   └── processed/
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_baselines.ipynb
│   ├── 03_calibration.ipynb
│   └── 04_adaptive_routing.ipynb
├── src/uaas/
│   ├── ingestion/
│   ├── preprocessing/
│   ├── classifiers/
│   ├── calibration/
│   ├── routing/
│   ├── evaluation/
│   └── logging/
├── tests/
├── results/
├── pyproject.toml
└── requirements.lock
```

A fenti könyvtárstruktúra célállapot, nem azt jelenti, hogy a komponensek már léteznek. A `data/raw/` csak olyan adatokat tartalmazzon, amelyek tárolását és terjesztését a licenc engedi; ellenkező esetben a repó a letöltési és előfeldolgozási utasításokat tárolja.

## Fő kockázatok és kontrollok

| Kockázat | Kontroll |
|---|---|
| Adatszivárgás review-k között | Review-szintű split és deduplikáció. |
| Túlbecsült modellbizonyosság | Kalibráció, risk–coverage elemzés, tartózkodási út. |
| Recall-romlás a költségcsökkentésért | Előre rögzített recall-korlát; fals negatívak külön riportja. |
| Témakörfüggő eredmények | Több tématerület, review-szintű eredmények. |
| Nem reprodukálható futások | Modell- és promptverzió, paraméterek, időbélyegek, futásazonosítók rögzítése. |
| Licenc- vagy adatvédelmi probléma | Forrás- és modelllicencek ellenőrzése; tiltott újraközlés kerülése. |
| Pontatlan JEV-definíció | Formális interfész és algoritmusspecifikáció F0-ban. |

## Következő munkamenet — priorizált teendők

1. **JEV-specifikáció beszerzése:** pontos modellnév/verzió, bemenet, kimenet, tanítási mód, pontszámértelmezés.
2. **Benchmark kiválasztása:** két elsődleges adatforrás tényleges letöltésének és licencének ellenőrzése.
3. **Elsődleges metrika rögzítése:** minimális recall és költségdefiníció meghatározása.
4. **Baseline implementálása:** M0 és M1 létrehozása az adaptív útválasztás előtt.
5. **Kísérleti konfiguráció befagyasztása:** tesztfelosztás, promptok, modellverziók, küszöbök és elemzési terv előzetes rögzítése.

## Tudományos pozicionálás

A UAAS újdonságát nem szabad pusztán az adaptív szűrés vagy az LLM használata alapján állítani. A cél annak empirikus vizsgálata, hogy a JEV-specifikus, kalibrált döntési szabály a releváns baseline-okhoz képest jobb költség–minőség kompromisszumot biztosít-e, átlátható hibakezeléssel és emberi felülvizsgálattal. Ezt a kísérletek előtt nyitott kérdésként kell kezelni.
