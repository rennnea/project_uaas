# UAAS — Kutatási protokoll

**Projekt:** Uncertainty-Aware Adaptive Screening (UAAS)  
**Verzió:** 0.1  
**Dátum:** 2026-10-10  
**Státusz:** előzetes protokoll; a hipotézisek empirikus ellenőrzésre várnak  
**Kutatási terület:** cím- és absztraktszűrés szisztematikus szakirodalmi áttekintésekben

> **Fontos státuszjelölés:** ez a dokumentum kutatási terv, nem eredménybeszámoló. A UAAS hatékonysága még nincs igazolva. A JEV pontos technikai specifikációját az implementáció előtt rögzíteni kell.

## 1. Összefoglaló és indoklás

A UAAS egy olyan szakirodalmi szűrési munkafolyamat prototípusát vizsgálja, amely a becsült bizonytalanság és a hibák költsége alapján választ az olcsó, gyors klasszifikáció, a részletesebb nagy nyelvi modell (LLM) általi elemzés és az emberi felülvizsgálat között.

A központi kérdés nem pusztán az, hogy az LLM képes-e relevanciacímkét rendelni egy rekordhoz, hanem az, hogy a JEV-alapú adaptív útválasztás csökkenti-e a teljes feldolgozási költséget egy előre rögzített minőségi korlát betartása mellett. A puszta accuracy nem elegendő értékelési mutató: a releváns tanulmány téves kizárása (false negative) súlyosabb lehet, mint egy irreleváns rekord téves beválasztása.

Az adaptív szűrés, az aktív tanulás és az LLM-alapú cím- és absztraktszűrés már létező kutatási területek. Az UAAS lehetséges hozzájárulása ezért nem önmagában az LLM vagy a RAG használata, hanem a kalibrált bizonytalanságra épülő döntési politika, a költségérzékeny útválasztás és a releváns baseline-okkal végzett reprodukálható összehasonlítás.

## 2. Kutatási kérdések

**RQ1 — Elsődleges:** Javítja-e a kalibrált bizonytalanságon alapuló, JEV-vel vezérelt adaptív útválasztás a szakirodalmi szűrés költség–minőség kompromisszumát a statikus LLM-alapú szűréshez képest?

**RQ2:** Előre jelzi-e a JEV által becsült kockázat a klasszifikációs hibákat?

**RQ3:** Csökkenthető-e a manuális felülvizsgálat aránya egy előre rögzített minimális recall mellett?

**RQ4:** Mennyire változik a teljesítmény eltérő kutatási témák és adathalmazok között?

**RQ5:** Fennmarad-e a megtakarítás, ha a teljes költségbe az LLM-hívások, futtatási idők és emberi ellenőrzés is beleszámít?

## 3. Hipotézisek

A hipotézisek tesztelendők, nem előre igazolt állítások.

| ID | Hipotézis | Elsődleges operacionalizálás |
|---|---|---|
| H1 | Az adaptív útválasztás csökkenti a feldolgozási költséget a statikus LLM-megoldáshoz képest. | Költség rekordra vetítve, azonos recall-korlát mellett. |
| H2 | A JEV kockázati pontszáma informatív a klasszifikációs hibák előrejelzésében. | Kalibrációs mutatók és a hibák elkülöníthetősége. |
| H3 | Az adaptív rendszer csökkenti a manuálisan ellenőrzendő rekordok arányát a megadott minőségi korlát mellett. | Recall, emberi felülvizsgálati arány és téves kizárások száma. |
| H4 | A teljesítmény adatforrásonként és témakörönként eltér. | Témakörönkénti eredmények és áttekintés-szintű variabilitás. |

A minimálisan elfogadható recall-küszöböt a végső tesztkészlet eredményeinek megtekintése előtt, a feladat kockázata és alkalmazási célja alapján kell rögzíteni. A küszöb utólagos kiválasztása torzíthatja az eredmények értelmezését.

## 4. Tervezett rendszerarchitektúra

1. **Bemeneti réteg:** cím, absztrakt, DOI vagy rekordazonosító, review-azonosító, kutatási protokoll, beválasztási és kizárási kritériumok.
2. **JEV-réteg:** relevanciaklasszifikáció, bizonytalansági vagy kockázati jelzés, illetve a következő feldolgozási út javaslata.
3. **Gyors útvonal:** olcsó klasszifikátor vagy kis modell, ha a döntés validált szabályok alapján automatikusan kezelhető.
4. **Mélyebb útvonal:** strukturált prompttal működő LLM-elemzés, a beválasztási kritériumokhoz kötött indoklással; szükség esetén további metaadatokkal vagy teljes szöveggel.
5. **Biztonságos kimenet:** automatikus döntés, emberi felülvizsgálatra továbbítás vagy tartózkodás. A releváns tanulmányok potenciális téves kizárását külön nyomon kell követni.
6. **Naplózás:** modell- és promptverzió, döntés, pontszám, evidenciarész, idő, költség és emberi korrekció.

### 4.1. A JEV specifikációjának követelményei

Az implementáció megkezdése előtt dokumentálni kell, hogy a JEV pontosan milyen modell vagy komponens, milyen bemenetet fogad, hogyan állít elő pontszámot, és a pontszám kalibrált valószínűségként vagy csupán rangsorolásra használható értékként értelmezhető-e. A nyelvi modell által önmagáról közölt „bizonyosság” önmagában nem tekinthető kalibrált valószínűségnek.

## 5. Adatok és adatforrások

### 5.1. Minimális rekordmezők

| Mező | Szerep | Kötelező? |
|---|---|---|
| `record_id` / DOI | Deduplikáció és visszakereshetőség. | Erősen ajánlott |
| `title` | Elsődleges szöveges bemenet. | Igen, ha elérhető |
| `abstract` | A cím- és absztraktszűrés fő bemenete. | Igen, lehetőleg |
| `review_id` / `topic` | A tanító-, validációs és tesztadatok review-szintű elkülönítéséhez. | Igen |
| `eligibility_criteria` | A kutatási kérdés és beválasztási feltételek. | Igen |
| `human_label` | Referencia-címke, lehetőleg emberi konszenzusból vagy a review publikált címkéiből. | Igen |
| `full_text` / URL | Második fázisú részletes ellenőrzés. | Opcionális az első prototípusban |
| `screening_rationale` | Az indoklás ellenőrzését segítheti, ha elérhető. | Opcionális |

### 5.2. Vizsgálandó benchmarkok

| Forrás | Tervezett használat | Ellenőrizendő korlátok |
|---|---|---|
| SYNERGY / aktívtanulásos szűréshez használt review-adatok | Baseline-ok és szimulációs kísérletek. | Hozzáférés, letöltési út, verzió, címkék és licenc. |
| SESR-Eval | Általánosíthatóság tesztelése szoftverfejlesztési szisztematikus áttekintéseken. A kapcsolódó közlemény 34 528 címkézett rekordot ír le 24 áttekintésből. | A licencet ellenőrizni kell; a szoftverfejlesztési domén eredményei nem általánosíthatók automatikusan más területekre. |
| Chan et al. (2025) által leírt egészségügyi/medicinális review-adatkészletek | Nagyobb, több tématerületet lefedő tesztelés, ha hozzáférhető. A közlemény több mint 8 600 review-t és 540 000 absztraktot ír le 51 területről. | Letöltés, licenc, címkézett rekordok és adatséma. |
| CLEF eHealth 2019 Technology-Assisted Review (TAR) | Kiegészítő benchmark vagy összehasonlítás. | A csomagok, címkék és felhasználási feltételek hozzáférhetősége. |
| Kaggle-adatkészletek | Felderítő prototípus és pipeline-teszt. | Eredet, duplikáció, címkék minősége, reprezentativitás és licenc. |

Elsődleges választásként olyan publikált review-adatokat kell keresni, amelyek címkéi dokumentáltak, az adat eredete követhető, és a felhasználási feltételek egyértelműek. A Kaggle platformon való megjelenés önmagában nem bizonyítja a címkék minőségét vagy tudományos reprezentativitását. A végleges adatforrást csak tényleges letöltés, sémaellenőrzés és licencvizsgálat után szabad rögzíteni.

### 5.3. Adatminőség és etika

- Dokumentálni kell a címkék eredetét, a felhasználási feltételeket és az esetleges emberi konszenzusfolyamatot.
- A tanító-, validációs és teszthalmazt **teljes review-k szerint** kell szétválasztani; ugyanazon review rekordjai nem kerülhetnek különböző halmazokba.
- DOI/rekordazonosító és normalizált cím alapján is ellenőrizni kell a duplikátumokat.
- Külön kell riportálni a hiányzó absztraktokat, a más nyelvű rekordokat és az eltérő címkézési sémákat.
- Az emberi felülvizsgálatot és az eltérő ítéletek rendezését auditálhatóan kell rögzíteni.
- A teljes szöveg csak akkor kerüljön az első prototípusba, ha a cím- és absztraktszűrés eredményei indokolják, és a licenc ezt lehetővé teszi.

## 6. Adatgyűjtési és integrációs stratégia

A connectorok és MCP-integrációk elsősorban adatfelderítésre, metaadat-lekérésre és munkafolyamat-integrációra szolgáljanak. A benchmark címkéit nem szabad kizárólag egy élő keresési API aktuális eredményeiből rekonstruálni, mert a rekordkészlet változhat, és a referencia-standard nem feltétlenül lesz ismert.

Javasolt első lépés egy rögzített, licencelt benchmark letöltése fájlból vagy hivatalos API-ból. Meg kell őrizni a nyers választ, a lekérés dátumát, az API-verziót és paramétereket, valamint a normalizált rekordokat. Az OpenAlex vagy a Crossref később metaadat-kiegészítésre használható. Az MCP nem helyettesíti a hibakezelést, kvótakezelést, licencellenőrzést vagy a reprodukálható adatpillanatkép rögzítését.

Lehetséges integrációk:

| Kapcsolódás | Lehetséges szerep |
|---|---|
| Crossref REST API | DOI, cím, szerzők és folyóirat-metaadatok; DOI-normalizálás. |
| OpenAlex API | Művek, szerzők, témakörök és kapcsolódó metaadatok. |
| PubMed / NCBI E-utilities | Orvosbiológiai rekordok keresése és metaadatai, amennyiben a benchmark biomedicinára fókuszál. |
| Semantic Scholar API | Kiegészítő publikációs és hivatkozási metaadatok, az elérhetőség és kvóták függvényében. |
| Kaggle API / nyílt fájlletöltés | Verziózott kísérleti adatok beszerzése, ha a hozzáférés és licenc ezt engedi. |
| MCP-szerver vagy saját adatadapter | Integrációs réteg; a kutatási logikát külön Python-modulokban kell tartani. |

**Validációs megjegyzés:** a jelen terv összeállításakor nem történt végponttól végpontig tartó letöltési teszt a felsorolt API-kkal. Az aktuális API-séma, hozzáférés, kvóták és licenc külön ellenőrzendő.

## 7. Kísérleti terv

Az első prototípusban az M0–M3 rendszerek összehasonlítása elegendő lehet. Az aktívtanulási baseline a következő szakaszban adható hozzá. Minden rendszer azonos végső tesztrekordokon, azonos beválasztási kritériumokkal fusson. Rögzíteni kell a promptokat, modellverziókat, hőmérsékletet, döntési küszöböket és ismétléseket.

### 7.1. Összehasonlítandó modellek

| Modell | Konfiguráció | Összehasonlítási cél |
|---|---|---|
| M0 | TF-IDF + logisztikus regresszió vagy Naive Bayes | Olcsó klasszikus baseline. |
| M1 | Egyetlen statikus LLM rögzített prompttal | Generatív baseline. |
| M2 | JEV + LLM, adaptív útválasztás nélkül | A JEV besorolási hozzájárulásának elkülönítése. |
| M3 | JEV + adaptív útválasztás + LLM + tartózkodás/emberi felülvizsgálat | A teljes javasolt UAAS-rendszer. |
| M4 (kiegészítő) | ASReview vagy más dokumentált aktívtanulási baseline | Összevetés létező szűrési módszerrel. |

### 7.2. Mérőszámok

| Dimenzió | Mutatók | Megjegyzés |
|---|---|---|
| Klasszifikáció | Recall, precision, F1, MCC, konfúziós mátrix | A releváns rekordok téves kizárását külön riportolni kell. |
| Kalibráció | Brier-score, ECE, kalibrációs görbe | Csak értelmezhető valószínűségi kimenet esetén. |
| Költség | API-költség, tokenek, futási idő, modellhívások | Rekordonként és teljes review-nként. |
| Felülvizsgálat | Manuális továbbítási arány, emberi munkaidő | Lehetőség szerint valós vagy szimulált munkaidővel. |
| Kockázat–lefedettség | Risk–coverage görbe, tartózkodási arány | Az automatikus döntések hibakockázata. |
| Reprodukálhatóság | Futásazonosító, verziók, konfiguráció, random seedek | Az eredmények visszaállíthatóságához. |

### 7.3. Végpontok

**Elsődleges végpont:** a teljes feldolgozási költség csökkenése egy előre rögzített minimális recall mellett. A költség definíciójának rögzítenie kell, hogy az API- és futtatási költségen felül a humán felülvizsgálatot is tartalmazza-e.

**Másodlagos végpontok:** manuális ellenőrzési arány, tévesen kizárt releváns rekordok száma, kalibráció, futási idő és témakörök közötti teljesítménykülönbség.

### 7.4. Statisztikai elemzés

- A fő szétválasztási egység a teljes review, ne az egyedi rekord legyen.
- Az eredményeket review-nként és összesítve is közölni kell.
- A modellkülönbségekhez konfidenciaintervallumot kell becsülni review-szintű újramintavételezéssel vagy a szerkezethez illeszkedő hierarchikus modellel.
- A végső tesztadatokat nem szabad hiperparaméter-hangolásra vagy küszöbválasztásra használni.
- A szükséges review-k és rekordok számát a benchmark kiválasztása és az előzetes variabilitás felmérése után kell meghatározni.

## 8. Kockázatok és kontrollok

| Kockázat | Kontroll |
|---|---|
| Adatszivárgás review-k között | Review-szintű szétválasztás és duplikátumvizsgálat. |
| Túlbecsült modellbizonyosság | Kalibráció, risk–coverage elemzés és tartózkodási út. |
| A recall romlása költségcsökkentésért | Előre rögzített recall-korlát és fals negatívak külön riportja. |
| Témakörfüggő működés | Több tématerület és review-szintű eredmények. |
| Nem reprodukálható API-k vagy modellek | Verziók, időbélyegek, promptok és paraméterek naplózása; lehetőség szerint nyílt baseline. |
| Licenc- vagy adatvédelmi probléma | Adatforrás- és modelllicenc ellenőrzése; nem engedélyezett újraközlés elkerülése. |
| A JEV meghatározása nem elég pontos | Formális interfész és algoritmusspecifikáció a fejlesztés előtt. |

## 9. Újdonság és értékelési szempont

Az adaptív szűrés, az aktív tanulás, a többmodellű együttműködés és a költség–recall kompromisszum már létező kutatási irányok. Az UAAS nem állíthat újdonságot pusztán az adaptivitás vagy az LLM használata alapján.

A lehetséges hozzájárulás akkor támasztható alá, ha a JEV-specifikus, kalibrált döntési szabály releváns baseline-okhoz képest kedvezőbb költség–minőség görbét ad, miközben a hibák és az emberi felülvizsgálat elszámolása átlátható. Ez jelenleg kutatási hipotézis, nem bizonyított eredmény.

## 10. Hivatkozások és ellenőrzendő források

Az alábbi tételek a tervezési dokumentum előzetes hivatkozáslistájából származnak. A végleges protokoll elfogadása előtt bibliográfiai adataikat, a hozzáférést és az adott állításokhoz való relevanciájukat ellenőrizni kell.

1. Ferdinands, G. et al. (2023). *Performance of active learning models for screening prioritization in systematic reviews: a simulation study into the Average Time to Discover relevant records.* Systematic Reviews, 12. https://consensus.app/papers/performance-of-active-learning-models-for-screening-ferdinands-schram/14734585139c562fa18282d5ed243824/
2. Teijema, J. et al. (2025). *Large-scale simulation study of active learning models for systematic reviews.* International Journal of Data Science and Analytics, 20, 5435–5456. https://consensus.app/papers/largescale-simulation-study-of-active-learning-models-for-teijema-bruin/152242f20aa45fb88f6031e7e3ac40b3/
3. van de Schoot, R. et al. (2021; online 2020). *An open source machine learning framework for efficient and transparent systematic reviews.* Nature Machine Intelligence, 3, 125–133. https://consensus.app/papers/details/caf1d3175727540e96de1c11d601f3b3/
4. Jiang, X. & Palade, V. (2025). *Weakly Supervised Active Learning for Abstract Screening Leveraging LLM-Based Pseudo-Labeling.* https://consensus.app/papers/weakly-supervised-active-learning-for-abstract-screening-jiang-palade/f28630f984c45de0875b7b9fcbb88edd/
5. Akinseloyin, O., Jiang, X. & Palade, V. (2026). *Large language model-based multiagent collaboration for abstract screening toward automated systematic reviews.* Biology Methods & Protocols, 11. https://consensus.app/papers/large-language-modelbased-multiagent-collaboration-for-akinseloyin-jiang/84668b125d595a59bfff8066c43cfdf6/
6. *SESR-Eval: Dataset for Evaluating LLMs in the Title-Abstract Screening of Systematic Reviews* (2025). arXiv:2507.19027. https://arxiv.org/abs/2507.19027
7. ASReview projekt — nyílt aktívtanulásos szoftver szisztematikus áttekintésekhez. https://asreview.nl/

A DataCamp-anyagok hasznosak lehetnek fejlesztési tanulóforrásként, de nem tekintendők tudományos bizonyítéknak az UAAS hipotéziseihez.

## 11. A következő munkamenet döntései

1. A JEV pontos specifikációjának beszerzése és leírása.
2. Két elsődleges adatforrás tényleges hozzáférhetőségének, licencének és adatsémájának ellenőrzése.
3. Az elsődleges minőségi korlát (minimális recall) és a hibák költségének előzetes definiálása.
4. Statikus baseline-ok létrehozása az adaptív rendszer megépítése előtt.
5. A kísérleti konfigurációk és a végső tesztprotokoll rögzítése.
