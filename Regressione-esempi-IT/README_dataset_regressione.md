# Dataset sintetici per gli esempi di regressione

Quattro dataset CSV (UTF-8, separatore virgola, nomi colonna senza accenti) generati con
`genera_dataset_regressione.py` (seme fisso 42, quindi riproducibili e modificabili nei parametri).
Per ogni dataset riportiamo le colonne, il target e la "verità" generativa: conoscerla in anticipo
ci permette di guidare la discussione sui coefficienti stimati e sui residui.

---

## 1. `ticket_risoluzione.csv` — tempo di risoluzione di un ticket (1.500 righe)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_ticket` | id | identificativo (da escludere dal modello) |
| `priorita` | categorica ordinale | Bassa / Media / Alta / Critica |
| `componente` | categorica | Backend, Frontend, Database, API, Mobile, Infrastruttura, Documentazione |
| `tipo` | categorica | Bug / Richiesta / Domanda |
| `lunghezza_descrizione` | numerica | caratteri della descrizione |
| `commenti_24h` | numerica | commenti ricevuti nelle prime 24 ore |
| `carico_assegnatario` | numerica | ticket aperti in carico all'assegnatario |
| `esperienza_assegnatario_mesi` | numerica | mesi di esperienza dell'assegnatario |
| `allegati` | binaria | presenza di allegati |
| **`ore_risoluzione`** | **target** | ore tra apertura e chiusura |

**Verità generativa** (effetti moltiplicativi, cioè additivi sul logaritmo delle ore):
ticket "medio" 60 ore; priorità Critica ×0,2, Alta ×0,35, Media ×0,6, Bassa ×1;
Infrastruttura ×1,5 e Database ×1,4, Documentazione ×0,5; Richiesta ×1,5, Domanda ×0,4;
descrizioni lunghe aumentano leggermente il tempo, commenti ed esperienza lo riducono,
il carico dell'assegnatario lo aumenta. Rumore lognormale (σ = 0,45).
Circa il 2% dei ticket è "dimenticato": tempi moltiplicati per 4–10 (outlier).

**Spunti didattici:** target asimmetrico → trasformazione logaritmica; codifica delle categoriche
(one-hot vs ordinale per la priorità); gli outlier "dimenticati" emergono dai residui.
Con una regressione lineare sul log si ottiene R² ≈ 0,70.

---

## 2. `build_ci.csv` — durata di una build/pipeline CI (2.000 righe)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_build` | id | numero progressivo della build |
| `righe_modificate` | numerica | righe cambiate nel commit/PR |
| `file_modificati` | numerica | file toccati (correlato alle righe) |
| `moduli_toccati` | numerica | moduli del monorepo interessati |
| `numero_test` | numerica | test eseguiti (cresce con i moduli) |
| `cache_hit` | binaria | 1 se la cache delle dipendenze è stata riutilizzata |
| `ora_giorno` | numerica | ora di avvio (0–23) |
| `giorno_settimana` | categorica | Lun … Dom |
| `runner` | categorica | small / medium / large |
| `trigger` | categorica | push / pull_request / schedule |
| **`durata_minuti`** | **target** | durata della pipeline |

**Verità generativa:** durata = (compilazione + test + 1,5 min di overhead) × fattore runner × congestione,
con compilazione = 1 + 0,0015·righe + 0,6·moduli (×1,8 se cache miss), test = 0,0035·numero_test,
runner small ×1,4 / medium ×1 / large ×0,7, congestione ×1,25 nelle ore di punta (10–11 e 15–16
nei giorni feriali). Rumore moltiplicativo σ = 0,15. **`trigger` non ha alcun effetto**: è una
variabile-esca per mostrare che non tutte le feature contano.

**Spunti didattici:** interazione cache × righe; effetto non lineare dell'ora (utile per feature
engineering: `ora_di_punta` binaria); collinearità tra righe, file, moduli e test.
R² ≈ 0,80 con un modello lineare.

---

## 3. `costo_cloud.csv` — costo cloud mensile per servizio (288 righe = 6 servizi × 48 mesi)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `mese` | data (YYYY-MM) | da 2022-09 a 2026-08 |
| `indice_mese` | numerica | 1…48, pronto come feature di trend |
| `servizio` | categorica | api-core, auth, storage, analytics, notifiche, frontend-web |
| `utenti_attivi` | numerica | utenti attivi mensili del servizio |
| `richieste_api_milioni` | numerica | richieste servite (milioni) |
| `dati_gb` | numerica | dati archiviati (GB, in crescita) |
| `deploy` | numerica | deploy effettuati nel mese |
| **`costo_euro`** | **target** | costo mensile del servizio |

**Verità generativa:** costo = base fissa per servizio × indice prezzi + 9 €/milione di richieste
+ 0,045 €/GB + 25 €/deploy, rumore 5%. Gli utenti totali crescono da 20k a 60k con stagionalità
(picco novembre, minimo agosto); le richieste derivano dagli utenti (collinearità voluta).
L'indice prezzi ha due gradini (+6% da gennaio 2024, +4% da luglio 2025) e circa il 2,5% dei mesi
contiene un'anomalia (costo ×1,5–2,5: job impazzito, incidente, test di carico dimenticato).

**Spunti didattici:** feature temporali (mese dell'anno, trend); stagionalità; collinearità
utenti/richieste; i gradini di prezzo e le anomalie si scoprono dai residui nel tempo.
R² ≈ 0,80 con un modello lineare (sale molto una volta rimossi gli outlier).

---

## 4. `story_point_ore.csv` — story point → ore effettive (800 righe)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_story` | id | identificativo della user story |
| `team` | categorica | Alpha / Beta / Gamma / Delta |
| `sprint` | numerica | numero dello sprint (1–24) |
| `story_point` | numerica | stima in punti (1, 2, 3, 5, 8, 13) |
| `tipo` | categorica | Feature / Bug / Debito tecnico / Spike |
| `seniority_sviluppatore` | categorica | Junior / Mid / Senior |
| `criteri_accettazione` | numerica | numero di criteri di accettazione |
| `dipendenze_esterne` | binaria | 1 se la story dipende da un altro team o fornitore |
| **`ore_effettive`** | **target** | ore realmente consumate |

**Verità generativa:** ore = ore_per_punto(team) × story_point^1,2 × seniority × tipo
× (1 + 0,30·dipendenze) × (1 + 0,03·(criteri − criteri attesi)), con ore_per_punto
Gamma 3,2 / Alpha 4,0 / Delta 4,8 / Beta 5,5; Junior ×1,35, Senior ×0,8; Bug ×1,15, Spike ×1,3.
Il rumore lognormale cresce con la dimensione (σ = 0,18 + 0,02·punti). **`sprint` non ha effetto.**

**Spunti didattici:** l'esponente 1,2 riproduce l'errore sistematico delle stime umane (le story
grandi sono sottostimate: 13 punti costano in media ~24 volte una story da 1 punto, non 13);
eteroschedasticità evidente nel grafico ore vs punti; effetto team (i punti di Beta "valgono"
quasi il doppio di quelli di Gamma); confronto lineare vs log-log. R² ≈ 0,80 sul log delle ore.
