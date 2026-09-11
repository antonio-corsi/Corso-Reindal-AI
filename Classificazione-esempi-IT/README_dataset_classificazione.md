# Dataset sintetici per gli esempi di classificazione

Quattro dataset CSV (UTF-8, separatore virgola, nomi colonna senza accenti) generati con
`genera_dataset_classificazione.py` (un seme fisso per ciascun dataset, quindi riproducibili e
modificabili nei parametri). Per ogni dataset riportiamo le colonne, il target, il bilanciamento delle
classi e la "verità" generativa: conoscerla in anticipo ci permette di guidare la discussione sui
coefficienti, sulle regole e sugli errori dei modelli.

---

## 1. `churn_saas.csv` — churn dei clienti SaaS (2.000 righe, ~7,5% positivi)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_cliente` | id | da escludere dal modello |
| `piano` | categorica | Basic / Pro / Enterprise |
| `fatturazione` | categorica | mensile / annuale |
| `mesi_anzianita` | numerica | mesi dall'attivazione |
| `licenze` | numerica | licenze acquistate (dipende dal piano) |
| `utenti_attivi_30gg` | numerica | utenti che hanno usato il prodotto negli ultimi 30 giorni |
| `login_medi_settimana` | numerica | login medi per utente attivo |
| `feature_usate` | numerica | funzionalità usate su 12 disponibili |
| `integrazioni_attive` | numerica | integrazioni configurate (0–5) |
| `ticket_90gg` | numerica | ticket aperti negli ultimi 90 giorni |
| `tempo_risposta_medio_ore` | numerica | tempo medio di prima risposta del supporto |
| `ritardi_pagamento_12m` | numerica | fatture pagate in ritardo negli ultimi 12 mesi (0–4) |
| **`churn`** | **target** | 1 se l'account non ha rinnovato nei 90 giorni successivi |

**Verità generativa** (modello logistico, log-odds additivi): fatturazione mensile +1,2; tasso di
utilizzo (`utenti_attivi_30gg / licenze`) −3,5 per punto; feature usate −0,22 ciascuna;
integrazioni −0,5 ciascuna; ticket +0,2 ciascuno; tempo di risposta +0,04 per ora; ritardi di
pagamento +0,9 ciascuno; anzianità −0,03 per mese; login −0,2; piano Basic +0,5, Enterprise −0,8
(Pro riferimento). L'intercetta è calibrata per un tasso medio del 7%.

**Spunti didattici:** classi sbilanciate (accuracy ingenua 92,5%); feature engineering del tasso di
utilizzo; odds ratio e intervalli di confidenza con statsmodels (i valori veri sono e^β: ad esempio
e^0,9 ≈ 2,5 per ritardo di pagamento); calibrazione delle probabilità; soglia scelta in base ai costi
(teorica c_FP/(c_FP+c_FN)). Regressione logistica: ROC-AUC ≈ 0,82–0,83.

---

## 2. `test_flaky.csv` — test flaky vs test rotto (1.200 righe, ~46% flaky)

Una riga = un fallimento di test in CI da classificare.

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_fallimento` | id | da escludere dal modello |
| `tasso_fallimento_storico` | numerica | quota di fallimenti nello storico del test |
| `alternanze_ultimi_20` | numerica | passaggi pass↔fail negli ultimi 20 run |
| `fallimenti_consecutivi` | numerica | fallimenti di fila al momento attuale |
| `solo_su_alcuni_runner` | binaria | fallisce solo su certi runner |
| `durata_media_s` | numerica | durata media del test (secondi) — **irrilevante per costruzione** |
| `cv_durata` | numerica | coefficiente di variazione della durata |
| `usa_rete`, `usa_db`, `usa_sleep_timeout` | binarie | dipendenze del test |
| `altri_test_falliti_stesso_commit` | numerica | altri test caduti nello stesso commit |
| `modulo_modificato_nel_commit` | binaria | il commit ha toccato il modulo testato |
| **`flaky`** | **target** | 1 = instabile, 0 = bug reale |

**Verità generativa:** l'etichetta viene estratta prima (45% flaky) e le feature dopo, con
distribuzioni diverse per classe ma sovrapposte: i flaky hanno molte alternanze (Poisson 5 contro
1,8), pochi fallimenti consecutivi (1–3 contro Poisson 3,5+1), durata più variabile, più dipendenze da
rete/sleep/runner; i rotti hanno storico pulito, altri test caduti nello stesso commit (Poisson 2
contro 0,4) e il modulo modificato (75% contro 20%). Il 4% delle etichette è invertito (rumore).

**Spunti didattici:** classi bilanciate (qui l'accuracy ha senso); regole leggibili dell'albero;
regola di parsimonia sulla profondità; importanze e feature correlate (`modulo_modificato` risulta
inutilizzato perché ridondante con `fallimenti_consecutivi`); `durata_media_s` come feature-esca.
Albero di profondità 3: accuracy ≈ 0,89–0,91.

---

## 3. `triage_ticket.csv` — triage dei ticket (2.000 righe, 4 classi)

Classi: `Bug` 40%, `Domanda` 26%, `Richiesta` 24%, `Incidente` 10%.

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_ticket` | id | da escludere dal modello |
| `canale` | categorica | form / email / chat / telefono |
| `tipo_cliente` | categorica | Free / Business / Enterprise |
| `componente_indicato` | categorica | Backend, Frontend, API, Mobile, Fatturazione, Account, Nessuno |
| `priorita_dichiarata` | categorica | Bassa / Media / Alta / Urgente |
| `stack_trace` | binaria | presenza di uno stack trace |
| `allegati` | numerica | numero di allegati |
| `lunghezza_descrizione` | numerica | caratteri della descrizione |
| `parole_errore` | numerica | occorrenze di parole legate a errori (error, crash, exception...) |
| `parole_richiesta` | numerica | occorrenze di parole che esprimono una richiesta (vorrei, possibile, aggiungere...) |
| `punti_interrogativi` | numerica | numero di "?" nel testo |
| `utenti_segnalanti` | numerica | utenti che segnalano lo stesso problema entro un'ora |
| `ora_apertura` | numerica | ora di apertura (0–23) |
| **`categoria`** | **target** | Bug / Richiesta / Domanda / Incidente |

**Verità generativa:** etichetta estratta prima, feature dopo, con distribuzioni per classe: gli
incidenti arrivano per telefono/chat, a qualunque ora, con priorità alta e molti utenti segnalanti
(Poisson 6); i bug portano stack trace (55%), allegati e parole di errore (Poisson 3); le richieste
hanno parole di richiesta (Poisson 2,5) e spesso nessun componente; le domande sono brevi, con punti
interrogativi (Poisson 2,5), su fatturazione e account.

**Spunti didattici:** multiclasse con classe rara → medie macro; scaling e one-hot per il kNN;
matrice di confusione 4×4 (la coppia più confusa è Richiesta/Domanda); spiegare una previsione
mostrando i vicini. kNN: accuracy ≈ 0,89, F1 macro ≈ 0,90.

---

## 4. `defect_pr.csv` — defect prediction su pull request (4.000 righe, ~11–12% positivi)

| Colonna | Tipo | Descrizione |
|---|---|---|
| `id_pr` | id | da escludere dal modello |
| `tipo` | categorica | feature / fix / refactor |
| `righe_aggiunte`, `righe_rimosse` | numeriche | dimensione della modifica (code lunghe) |
| `file_modificati`, `moduli_toccati` | numeriche | ampiezza della modifica |
| `complessita_media` | numerica | complessità ciclomatica media delle funzioni toccate |
| `test_aggiunti` | binaria | la PR aggiunge o modifica test |
| `copertura_modulo` | numerica | copertura di test del modulo (%) |
| `esperienza_autore_mesi` | numerica | mesi di esperienza dell'autore |
| `commit_autore_modulo` | numerica | commit precedenti dell'autore sul modulo |
| `reviewer` | numerica | numero di reviewer (0–3) |
| `commenti_review` | numerica | commenti ricevuti in review |
| `ora_merge` | numerica | ora del merge (0–23) |
| `giorno_merge` | categorica | Lun … Ven |
| **`bug_entro_30gg`** | **target** | 1 se è servito un fix su quel codice entro 30 giorni |

**Verità generativa** (log-odds): effetti additivi deboli (dimensione +0,25 per unità di
log-righe, complessità +0,02, nessun reviewer +0,4, test −0,3, copertura −0,005/punto, esperienza
−0,005/mese) e **congiunzioni forti**: modifica > 300 righe **senza** test +2,8; copertura < 50%
**e** complessità > 10 +2,4; autore con meno di 12 mesi **su 3 o più moduli** +2,0; merge di notte
(20–7) o venerdì dalle 16 +1,2; review approfondita (≥ 2 reviewer e ≥ 3 commenti) −1,0. Intercetta
calibrata per un tasso medio del 12%.

**Spunti didattici:** il rischio nasce dalle interazioni (tabelle a doppia entrata nell'EDA);
confronto regressione logistica vs MLP (AUC ≈ 0,79 contro ≈ 0,81 in cross-validazione); la
regolarizzazione `alpha` decide tutto (con `alpha` piccolo la rete fa peggio del modello lineare);
soglia scelta sulla curva precision-recall; average precision come metrica per positivi rari.
