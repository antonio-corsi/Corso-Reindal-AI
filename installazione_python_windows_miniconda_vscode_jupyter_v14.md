# Installare Python su Windows 10/11 con Miniconda, Conda, Windows Terminal e Visual Studio Code

Questa guida descrive una configurazione semplice e ordinata per utilizzare **Python su Windows 10/11** durante il corso, usando **Prompt dei comandi (cmd)** all'interno di **Windows Terminal**.

L'idea è:

```text
Miniconda
   ↓
Conda
   ↓
Ambiente dedicato al corso
   ↓
Windows Terminalexport
   ↓
Prompt dei comandi (cmd)
   ↓
Visual Studio Code
   ↓
Python + estensioni Python
```

In questo modo Python e i pacchetti utilizzati nel corso rimangono isolati dal resto del sistema.

---

## Indice

1. [Installare Visual Studio Code](#1-installare-visual-studio-code)
2. [Installare Miniconda](#2-installare-miniconda)
3. [Rendere Conda disponibile nel Prompt dei comandi di Windows Terminal](#3-rendere-conda-disponibile-nel-prompt-dei-comandi-di-windows-terminal)
4. [Creare un ambiente Conda dedicato al corso](#4-creare-un-ambiente-conda-dedicato-al-corso)
5. [Attivare l'ambiente del corso](#5-attivare-lambiente-del-corso)
6. [Isolare l'ambiente con `PYTHONNOUSERSITE`](#6-isolare-lambiente-con-pythonnousersite)
7. [Verificare Python](#7-verificare-python)
8. [Aggiornare `pip`](#8-aggiornare-pip)
9. [Installare i package con Conda](#9-installare-i-package-con-conda)
10. [Interprete Python e kernel Jupyter](#10-interprete-python-e-kernel-jupyter)
11. [Avviare Visual Studio Code dall'ambiente Conda](#11-avviare-visual-studio-code-dallambiente-conda)
12. [Installare le estensioni Python e Jupyter in VS Code](#12-installare-le-estensioni-python-e-jupyter-in-vs-code)
13. [Selezionare il kernel in VS Code](#13-selezionare-il-kernel-in-vs-code)
14. [Flusso di lavoro consigliato durante il corso](#14-flusso-di-lavoro-consigliato-durante-il-corso)
15. [Terminare la sessione](#15-terminare-la-sessione)
16. [Problemi comuni](#problemi-comuni)
17. [In caso di problemi con VS Code / Jupyter](#in-caso-di-problemi-con-vs-code--jupyter)
18. [Riepilogo dei comandi](#riepilogo-dei-comandi)
19. [Concetto fondamentale](#concetto-fondamentale)

---

## 1. Installare Visual Studio Code

Se Visual Studio Code non è già installato:

1. scaricare **Visual Studio Code** dal sito ufficiale Microsoft;
2. eseguire il programma di installazione;
3. lasciare attive le opzioni standard;
4. verificare che il comando `code` sia disponibile da terminale.

Aprire **Windows Terminal** usando il profilo **Prompt dei comandi** e digitare:

```cmd
code --version
```

Se viene visualizzato il numero di versione di VS Code, il comando `code` è configurato correttamente.

---

## 2. Installare Miniconda

**Miniconda** è una distribuzione minimale che installa:

- Python;
- `conda`, il gestore degli ambienti e dei pacchetti;
- pochi altri componenti essenziali.

È più leggera di Anaconda e permette di installare soltanto ciò che serve.

Scaricare dal sito ufficiale la versione di **Miniconda per Windows 64 bit** ed eseguire il file `.exe`.

### Impostazioni consigliate

Durante l'installazione:

- scegliere **Just Me**, salvo esigenze particolari;
- lasciare la cartella di installazione proposta;
- **non è necessario aggiungere Miniconda al PATH di Windows**;
- completare l'installazione con le opzioni predefinite.

---

## 3. Rendere Conda disponibile nel Prompt dei comandi di Windows Terminal

Dopo l'installazione di Miniconda, può essere necessario inizializzare Conda una sola volta per **cmd.exe**.

Dal menu **Start** aprire:

```text
Miniconda Prompt
```

e digitare:

```cmd
conda init cmd.exe
```

Chiudere quindi Miniconda Prompt.

Chiudere anche eventuali finestre di Windows Terminal già aperte e riaprire **Windows Terminal** con il profilo:

```text
Prompt dei comandi
```

Verificare che Conda sia disponibile:

```cmd
conda --version
```

Ad esempio:

```text
conda 26.x.x
```

> `conda init cmd.exe` deve normalmente essere eseguito una sola volta.

---

## 4. Creare un ambiente Conda dedicato al corso

È buona pratica **non lavorare direttamente nell'ambiente `base`**.

Creiamo un ambiente separato per il corso:

```cmd
conda create -n corso-python python=3.12
```

Dove:

- `conda create` crea un nuovo ambiente;
- `-n corso-python` assegna il nome `corso-python`;
- `python=3.12` installa Python 3.12 nell'ambiente.

Quando Conda chiede:

```text
Proceed ([y]/n)?
```

digitare:

```text
y
```

e premere **Invio**.

Per vedere gli ambienti disponibili:

```cmd
conda env list
```

oppure:

```cmd
conda info --envs
```

---

## 5. Attivare l'ambiente del corso

Ogni volta che si inizia a lavorare al corso, aprire **Windows Terminal** con il profilo **Prompt dei comandi** e digitare:

```cmd
conda activate corso-python
```

Il prompt dovrebbe cambiare e mostrare il nome dell'ambiente:

```text
(corso-python) C:\Users\NomeUtente>
```

La presenza di:

```text
(corso-python)
```

indica che l'ambiente è attivo.

---

## 6. Isolare l'ambiente con `PYTHONNOUSERSITE`

In alcuni computer Python può trovare package installati in precedenza nella cartella personale dell'utente, anche quando è attivo un ambiente Conda.

Per evitare che Python utilizzi questi package esterni all'ambiente del corso, nel **Prompt dei comandi (cmd)** si può impostare:

```cmd
set PYTHONNOUSERSITE=1
```

Questa variabile d'ambiente dice a Python:

> non aggiungere al percorso di ricerca dei moduli la cartella *user site-packages* dell'utente.

In pratica, aiuta a fare in modo che Python utilizzi soltanto i package disponibili nell'ambiente Conda attivo e nelle sue dipendenze.

Schema semplificato:

```text
Senza PYTHONNOUSERSITE=1

Ambiente Conda
      │
      ├── package dell'ambiente
      │
      └── possibili package installati nella cartella utente
                         ↑
                  possibile interferenza
```

Con:

```cmd
set PYTHONNOUSERSITE=1
```

si ottiene:

```text
Ambiente Conda
      │
      └── package dell'ambiente
             ↑
      ambiente più isolato
```

Questo può essere utile soprattutto durante un corso, perché riduce il rischio che due partecipanti ottengano comportamenti diversi a causa di package Python installati in precedenza sul proprio computer.

## Quando eseguire il comando

Una possibile sequenza di avvio della sessione è:

```cmd
conda activate corso-python
set PYTHONNOUSERSITE=1
cd C:\CorsoPython
code .
```

In questo modo VS Code viene lanciato dal terminale con la variabile già impostata e la eredita.

## Attenzione: `set` vale solo per la sessione corrente

Con il Prompt dei comandi, il comando:

```cmd
set PYTHONNOUSERSITE=1
```

imposta la variabile soltanto nella finestra di terminale corrente e nei programmi avviati da essa.

Chiudendo quella finestra di Windows Terminal, l'impostazione viene persa.

Quindi, alla sessione successiva, occorre eventualmente eseguire di nuovo:

```cmd
set PYTHONNOUSERSITE=1
```

Questo comportamento è spesso preferibile durante un corso, perché non modifica permanentemente la configurazione di Windows.

## Verificare il valore

Per controllare che la variabile sia impostata:

```cmd
echo %PYTHONNOUSERSITE%
```

L'output dovrebbe essere:

```text
1
```

## Rimuovere l'impostazione nella sessione corrente

Per cancellare la variabile:

```cmd
set PYTHONNOUSERSITE=
```

Dopo questo comando:

```cmd
echo %PYTHONNOUSERSITE%
```

non dovrebbe più mostrare `1`.

## In sintesi

```cmd
set PYTHONNOUSERSITE=1
```

significa:

```text
PYTHON      → riguarda il comportamento di Python
NO          → esclude
USER SITE   → la cartella dei package installati a livello utente
=1          → attiva questa opzione
```

L'obiettivo è semplice:

> evitare che package Python installati fuori dall'ambiente Conda interferiscano con l'ambiente utilizzato durante il corso.

Per una sessione del corso, il flusso consigliato diventa quindi:

```cmd
conda activate corso-python
set PYTHONNOUSERSITE=1
cd C:\CorsoPython
code .
```

## 7. Verificare Python

Con l'ambiente attivo:

```cmd
python --version
```

Ad esempio:

```text
Python 3.12.x
```

Per verificare quale eseguibile Python viene utilizzato:

```cmd
where python
```

Il primo percorso mostrato dovrebbe fare riferimento alla cartella dell'ambiente Conda `corso-python`.

Per verificare **esattamente quale interprete Python sta eseguendo il comando**, si può usare anche:

```cmd
python -c "import sys; print(sys.executable)"
```

Il significato è:

- `python` avvia l'interprete Python attualmente attivo;
- `-c` significa **esegui direttamente il codice indicato tra virgolette**;
- `import sys` importa il modulo di sistema `sys`;
- `sys.executable` contiene il percorso completo dell'interprete Python in uso;
- `print(...)` visualizza quel percorso.

Esempio di risultato:

```text
C:\Users\NomeUtente\miniconda3\envs\corso-python\python.exe
```

Questo comando è molto utile per controllare che sia effettivamente in uso il Python dell'ambiente Conda del corso.

---

## 8. Aggiornare `pip`

Con l'ambiente attivo:

```cmd
python -m pip install --upgrade pip
```

Per verificare:

```cmd
pip --version
```

---

## 9. Installare i package con Conda

Con l'ambiente del corso attivo:

```cmd
conda activate corso-python
```

i package si installano con:

```cmd
conda install NOME_PACKAGE
```

Per esempio:

```cmd
conda install numpy
```

Per installare più package insieme:

```cmd
conda install numpy pandas matplotlib scikit-learn
```

### Specificare il canale

Un **canale Conda** è il repository dal quale Conda scarica i package.

Un canale molto usato in ambito scientifico e Data Science è:

```text
conda-forge
```

> **Raccomandazione.** **Usare `conda-forge` come unico canale ed escludere `defaults`.**
>
> Per il corso adotteremo quindi `conda-forge` come unica sorgente dei package, in modo da rendere l'ambiente più coerente e riproducibile ed evitare mescolanze tra package provenienti da canali diversi.
>
> La forma consigliata sarà:
>
> ```cmd
> conda install -c conda-forge --override-channels NOME_PACKAGE
> ```
>
> L'opzione `--override-channels` dice a Conda di ignorare i canali già configurati nel sistema, compreso `defaults`, e di usare solo quelli indicati con `-c`.

Per installare un package da `conda-forge` si usa l'opzione:

```cmd
-c conda-forge
```

Per esempio:

```cmd
conda install -c conda-forge pandas
```

oppure, per più package:

```cmd
conda install -c conda-forge numpy pandas matplotlib scikit-learn
```

`-c` è l'abbreviazione di:

```text
--channel
```

Quindi:

```cmd
conda install -c conda-forge pandas
```

e:

```cmd
conda install --channel conda-forge pandas
```

sono equivalenti.

Se si vuole usare **solo** il canale indicato, ignorando gli altri eventualmente configurati:

```cmd
conda install -c conda-forge --override-channels pandas
```

> **Nota:** `--override-channels` dice a Conda di ignorare i canali configurati nel sistema e di usare solo quelli specificati nel comando con `-c`.  
> Quindi:
>
> ```cmd
> conda install -c conda-forge --override-channels pandas
> ```
>
> significa: **installa `pandas` usando esclusivamente `conda-forge`**.

Per il corso useremo preferibilmente questa forma, perché rende esplicita la sorgente dei package:

```cmd
conda install -c conda-forge --override-channels NOME_PACKAGE
```

### Installare i package necessari al corso

Per esempio:

```cmd
conda install -c conda-forge --override-channels numpy pandas matplotlib scikit-learn
```

Per controllare i package installati:

```cmd
conda list
```


> 👉 **Strategia di installazione**
>
> Privilegiare sempre `conda install`; ricorrere a `python -m pip install` solo per i pacchetti non disponibili tramite **Conda**.
>
> Ci sono però **due importanti accortezze**.
>
> **1. Rispettare l'ordine temporale**
>
> Conviene installare **prima tutto il possibile con Conda** e, solo in un secondo momento, **ciò che resta con pip**.
>
> Il motivo è che Conda, quando risolve l'ambiente, non gestisce in modo completo i pacchetti già installati da pip. Se si installa qualcosa con pip e poi si esegue un successivo:
>
> ```cmd
> conda install ...
> ```
>
> Conda può modificare o sostituire file e dipendenze su cui il pacchetto installato con pip faceva affidamento, rendendo incoerente o non funzionante l'installazione precedente.
>
> Per questo motivo è consigliabile lasciare **pip "in coda"**:
>
> ```text
> 1. conda install ...
> 2. conda install ...
> 3. conda install ...
> 4. python -m pip install ...   ← solo ciò che manca
> ```
>
> **2. Non alternare Conda e pip sullo stesso pacchetto**
>
> Una volta che un pacchetto è stato installato con pip, gli aggiornamenti di **quel pacchetto** dovrebbero essere eseguiti con pip, non con Conda, e viceversa.
>
> Per esempio, se è stato installato con:
>
> ```cmd
> python -m pip install nome-pacchetto
> ```
>
> gli aggiornamenti successivi dovrebbero essere eseguiti con:
>
> ```cmd
> python -m pip install --upgrade nome-pacchetto
> ```
>
> e non con:
>
> ```cmd
> conda update nome-pacchetto
> ```
>
> Mescolare i due gestori sullo stesso package è una delle cause più comuni di ambienti Python incoerenti.
>
> **Forma consigliata per pip**
>
> In ogni caso, preferire sempre:
>
> ```cmd
> python -m pip install NOME_PACKAGE
> ```
>
> alla forma:
>
> ```cmd
> pip install NOME_PACKAGE
> ```
>
> Il significato è:
>
> - `python` richiama l'interprete Python attualmente attivo;
> - `-m` significa **esegui un modulo Python**;
> - `pip` è il modulo che gestisce l'installazione dei package;
> - `install NOME_PACKAGE` chiede a pip di installare il package indicato.
>
> In pratica:
>
> ```text
> python -m pip install pandas
> ```
>
> significa:
>
> > usa **questo Python** per eseguire **il suo pip** e installare `pandas`.
>
> Questa forma è preferibile a `pip install` perché riduce il rischio di usare accidentalmente un `pip` appartenente a un'altra installazione di Python.
>
> È quindi particolarmente utile quando sul PC sono presenti:
>
> - più versioni di Python;
> - più ambienti Conda;
> - Python installato anche fuori da Miniconda.
>
> Dopo aver attivato:
>
> ```cmd
> conda activate corso-python
> ```
>
> il comando:
>
> ```cmd
> python -m pip install NOME_PACKAGE
> ```
>
> installa il package usando il `pip` collegato al Python dell'ambiente `corso-python`.
>
> ### Comandi `pip` da evitare
>
> **⛔ Evitare sempre:**
>
> ```cmd
> pip install NOME_PACKAGE
> ```
>
> Il problema è semplice: **quale `pip` viene eseguito?**
>
> Se sul PC sono presenti più versioni di Python o più ambienti, il comando `pip` trovato nel `PATH` potrebbe non essere quello associato all'interprete Python dell'ambiente attivo.
>
> Per questo motivo usare invece:
>
> ```cmd
> python -m pip install NOME_PACKAGE
> ```
>
> così è l'interprete Python attivo a eseguire il proprio modulo `pip`.
>
> **⛔ Evitare quasi sempre:**
>
> ```cmd
> python -m pip install --user NOME_PACKAGE
> ```
>
> L'opzione:
>
> ```text
> --user
> ```
>
> installa il package nello **user-site**, cioè in una cartella personale dell'utente Windows, **non nell'ambiente Conda del corso**.
>
> Su Windows il percorso è tipicamente simile a:
>
> ```text
> %APPDATA%\Python\PythonXX\site-packages
> ```
>
> Quindi:
>
> ```text
> ambiente corso-python
>         │
>         ├── package dell'ambiente
>         │
>         └── NON contiene il package installato con --user
>
> user-site di Windows
>         │
>         └── package installato con --user
> ```
>
> Con un normale `venv`, `pip install --user` viene generalmente rifiutato perché lo user-site non è visibile nell'ambiente. Con un ambiente Conda, invece, può installare il package fuori dall'ambiente senza che l'errore sia evidente: si può quindi credere di avere installato il package nel proprio ambiente quando in realtà si trova nello user-site.
>
> In una guida che lavora sempre con ambienti Conda dedicati, `--user` è quindi **lo strumento sbagliato**, perché contraddice l'isolamento dell'ambiente.
>
> Il meccanismo dello *user-site* e il modo per neutralizzarne l'uso sono spiegati nella sezione dedicata a:
>
> ```cmd
> set PYTHONNOUSERSITE=1
> ```
>
> **Perché “quasi sempre” e non “sempre”?**
>
> `--user` ha alcuni usi legittimi fuori dal perimetro di questa guida. Il caso classico è l'uso del **Python di sistema senza permessi di amministratore**, quando si vuole installare un package soltanto per l'utente corrente.
>
> Un altro caso riguarda alcuni tool a riga di comando che si desidera rendere disponibili all'utente indipendentemente dall'ambiente; per questo scenario oggi è spesso preferibile usare strumenti dedicati come `pipx`.
>
> Nel nostro corso, che utilizza sempre un ambiente Conda dedicato, la regola pratica rimane:
>
> ```text
> NON usare pip install ...
> NON usare pip install --user ...
> usare, solo quando necessario:
> python -m pip install ...
> ```



> 📌 **Importante: non installare package dalle celle del notebook con `!pip` o `!conda`**
>
> Evitare di eseguire nelle celle del notebook comandi come:
>
> ```python
> !pip install nome_pacchetto
> ```
>
> oppure:
>
> ```python
> !conda install nome_pacchetto
> ```
>
> Il prefisso `!` fa eseguire il comando attraverso una **shell separata**. In quel contesto non è garantito che il `pip` o il `conda` individuato corrisponda esattamente all'ambiente Python associato al **kernel** del notebook.
>
> Il rischio è quindi di installare il package in un ambiente diverso da quello effettivamente usato dal notebook, creando incoerenze o problemi nel kernel.
>
> **Regola consigliata per il corso:** installare i package da **Windows Terminal**, dopo avere attivato l'ambiente Conda.
>
> ```cmd
> conda activate corso-python
> conda install -c conda-forge --override-channels nome_pacchetto
> ```
>
> Se il package non è disponibile tramite Conda:
>
> ```cmd
> conda activate corso-python
> python -m pip install nome_pacchetto
> ```
>
> **Meglio quindi installare sempre da terminale con l'ambiente Conda attivo.**


## 10. Interprete Python e kernel Jupyter

Poiché durante il corso lavoreremo esclusivamente con **notebook Jupyter (`.ipynb`)**, è importante distinguere due concetti: **interprete Python** e **kernel Jupyter**.

### L'interprete Python

L'**interprete** è il programma che esegue il linguaggio Python.

Nel nostro ambiente Conda:

```text
corso-python
```

l'interprete è il file `python.exe` installato dentro quell'ambiente.

In forma semplificata:

```text
Ambiente Conda: corso-python
        │
        └── python.exe
             ↑
        interprete Python
```

Quando creiamo l'ambiente con:

```cmd
conda create -n corso-python python=3.12
```

stiamo quindi creando un ambiente che contiene, tra le altre cose, uno specifico **interprete Python 3.12**.

L'interprete stabilisce:

> **quale Python stiamo utilizzando e quali package appartengono a quell'ambiente.**

---

### Il kernel Jupyter

Un notebook Jupyter non esegue direttamente le celle semplicemente "attraverso Python": utilizza un **kernel**.

Il **kernel** è un processo attivo che:

- usa un determinato interprete Python;
- riceve il codice delle celle dal notebook;
- lo esegue;
- restituisce i risultati a VS Code;
- mantiene in memoria lo stato della sessione.

Per esempio, se in una cella eseguiamo:

```python
x = 100
```

e successivamente, in un'altra cella:

```python
print(x)
```

otteniamo:

```text
100
```

perché il kernel è rimasto attivo e conserva `x` in memoria.

Se eseguiamo:

```text
Restart Kernel
```

la memoria del kernel viene azzerata e le variabili create in precedenza non esistono più.

La differenza essenziale è quindi:

```text
INTERPRETE = quale installazione di Python viene utilizzata

KERNEL     = la sessione attiva che usa quell'interprete
             per eseguire le celle del notebook
```

La relazione completa è:

```text
Ambiente Conda
corso-python
      │
      ├── Python 3.12
      │      │
      │      └── INTERPRETE
      │
      └── ipykernel
             │
             ▼
        KERNEL JUPYTER
             │
             ▼
       Notebook .ipynb
             │
             ▼
        celle eseguite
```

---

### Installare `ipykernel`

Perché l'ambiente Conda possa essere utilizzato come kernel di un notebook Jupyter, installiamo il package:

```text
ipykernel
```

Prima attivare l'ambiente:

```cmd
conda activate corso-python
```

quindi installare:

```cmd
conda install ipykernel
```

Se si vuole specificare esplicitamente il canale utilizzato nel corso:

```cmd
conda install -c conda-forge --override-channels ipykernel
```

`ipykernel` è il componente che collega l'interprete Python dell'ambiente alla modalità di esecuzione richiesta dai notebook Jupyter.

In termini semplici:

```text
python.exe
    │
    ▼
ipykernel
    │
    ▼
kernel utilizzabile da Jupyter / VS Code
```

> Normalmente non è necessario eseguire manualmente `python -m ipykernel install`: VS Code, con le estensioni **Python** e **Jupyter**, rileva gli ambienti Conda disponibili e permette di selezionarli direttamente.

---

## 11. Avviare Visual Studio Code dall'ambiente Conda

Questo è il flusso consigliato.

Prima attivare l'ambiente:

```cmd
conda activate corso-python
```

Poi lanciare Visual Studio Code:

```cmd
code
```

Oppure, se ci si trova già nella cartella del progetto:

```cmd
code .
```

Il punto `.` significa:

> apri in VS Code la cartella corrente.

Esempio:

```cmd
cd C:\CorsoPython
conda activate corso-python
code .
```

---

## 12. Installare le estensioni Python e Jupyter in VS Code

In Visual Studio Code aprire **Extensions** dalla barra laterale sinistra oppure usare:

```text
Ctrl + Shift + X
```

Installare le seguenti estensioni, tutte pubblicate da **Microsoft**.

### 1. Pylance

Fornisce supporto avanzato alla scrittura del codice Python:

- completamento automatico;
- analisi statica;
- suggerimenti sui tipi;
- navigazione nel codice.

### 2. Python

È l'estensione principale per l'integrazione tra VS Code e Python.

Serve, tra le altre cose, per:

- individuare gli interpreti Python;
- lavorare con gli ambienti Conda;
- integrare gli strumenti Python in VS Code.

### 3. Python Debugger

Aggiunge il supporto al debugging Python:

- breakpoint;
- esecuzione passo-passo;
- ispezione delle variabili;
- analisi dello stack delle chiamate.

### 4. Python Environments

Facilita la gestione degli ambienti Python e Conda direttamente da VS Code.

Nel nostro caso l'ambiente da utilizzare sarà:

```text
corso-python
```

### 5. Jupyter

Poiché durante il corso lavoreremo **solo con notebook `.ipynb`**, questa estensione è indispensabile.

Permette di:

- aprire e modificare notebook Jupyter;
- eseguire le celle;
- scegliere il kernel;
- visualizzare output, grafici e tabelle;
- gestire la sessione del notebook.

### Stampare un file Markdown aperto in VS Code

VS Code permette di visualizzare direttamente la **preview renderizzata** di un file Markdown `.md`.

Per aprirla:

```text
Ctrl + Shift + V
```

oppure:

```text
Ctrl + Shift + P
→ Markdown: Open Preview
```

Per la sola visualizzazione non serve installare alcuna estensione aggiuntiva.

Per **stampare** un file Markdown, invece, il flusso più pratico è:

```text
file .md
   ↓
Markdown Preview
   ↓
esportazione in PDF
   ↓
stampa del PDF
```

Se si desidera fare tutto da VS Code, è possibile installare un'estensione dedicata, ad esempio:

```text
Markdown PDF
```

che aggiunge comandi di esportazione, ad esempio:

```text
Markdown PDF: Export (pdf)
```

Una volta creato il PDF, lo si può aprire e stampare normalmente.

> **In sintesi**
>
> - per leggere il Markdown: `Ctrl + Shift + V`
> - per stamparlo: esportarlo prima in PDF e poi stampare il PDF

---

### Le estensioni da installare

```text
Pylance
Python
Python Debugger
Python Environments
Jupyter
```

Tutte devono avere come publisher:

```text
Microsoft
```

> L'estensione **Markdown PDF** è opzionale: serve soltanto se si desidera esportare e stampare comodamente i file Markdown direttamente da VS Code.

## 13. Selezionare il kernel in VS Code

Aprire il notebook `.ipynb`.

In alto a destra scegliere:

```text
Select Kernel
```

VS Code può mostrare una finestra con diverse sorgenti possibili del kernel:

```text
Python Environments...
Jupyter Kernel...
Existing Jupyter Server...
```

Nel nostro corso, con **Miniconda + ambiente Conda locale**, la scelta corretta è:

```text
Python Environments...
```

Questa voce mostra gli ambienti Python disponibili sul computer, compresi gli ambienti Conda.

Selezionare quindi l'ambiente del corso, ad esempio:

```text
Python 3.12 (corso-python)
```

Il flusso è:

```text
Select Kernel
     ↓
Python Environments...
     ↓
corso-python
     ↓
Python 3.12 dell'ambiente Conda
     ↓
ipykernel
     ↓
esecuzione delle celle del notebook
```

#### Significato delle tre opzioni

**Python Environments...**

Permette di scegliere uno degli ambienti Python installati sul computer:

- ambienti Conda;
- ambienti `venv`;
- altre installazioni Python locali.

È **l'opzione da utilizzare nel corso**.

---

**Jupyter Kernel...**

Permette di scegliere un **kernel Jupyter già registrato** nel sistema tramite una *kernelspec*.

Per esempio, un kernel può essere registrato manualmente con:

```cmd
python -m ipykernel install --user --name corso-python
```

Nel nostro approccio questo passaggio **non è normalmente necessario**, perché VS Code può usare direttamente l'ambiente Conda tramite:

```text
Python Environments...
```

---

**Existing Jupyter Server...**

Permette di collegare VS Code a un **Jupyter Server già esistente**, locale o remoto.

È utile, ad esempio, quando il notebook viene eseguito su:

- server aziendali;
- macchine remote;
- infrastrutture GPU;
- JupyterHub;
- server di laboratorio.

Nel nostro scenario locale su Windows **non è necessario**.

---

In sintesi:

```text
Per il corso:

Select Kernel
      ↓
Python Environments...
      ↓
corso-python
```

A quel punto il notebook utilizzerà:

```text
corso-python
      │
      ├── interprete Python 3.12
      ├── ipykernel
      └── package installati nell'ambiente
```

Tutte le celle del notebook saranno quindi eseguite dal **kernel associato all'ambiente `corso-python`**.

## 14. Flusso di lavoro consigliato durante il corso

Ogni volta che si inizia una sessione:

```text
1. Aprire Windows Terminal
          ↓
2. Selezionare Prompt dei comandi
          ↓
3. Attivare l'ambiente Conda
          ↓
4. Impostare PYTHONNOUSERSITE
          ↓
5. Spostarsi nella cartella del progetto
          ↓
6. Avviare VS Code
          ↓
7. Aprire il notebook e selezionare il kernel corso-python
```

Comandi:

```cmd
conda activate corso-python
set PYTHONNOUSERSITE=1
cd C:\CorsoPython
code .
```

Schema:

```text
Windows Terminal
      │
      ▼
Prompt dei comandi (cmd)
      │
      │  conda activate corso-python
      ▼
(corso-python)
      │
      │  cd C:\CorsoPython
      ▼
Cartella del progetto
      │
      │  code .
      ▼
Visual Studio Code
      │
      ▼
Notebook .ipynb
      │
      ▼
Kernel: corso-python
```

---

## 15. Terminare la sessione

Quando si è terminato di lavorare, è possibile disattivare l'ambiente con:

```cmd
conda deactivate
```

Il prefisso:

```text
(corso-python)
```

scomparirà dal prompt.

---

# Problemi comuni

## `conda` non viene riconosciuto

Se compare un messaggio simile a:

```text
'conda' non è riconosciuto come comando interno o esterno...
```

aprire **Miniconda Prompt** dal menu Start ed eseguire:

```cmd
conda init cmd.exe
```

Chiudere completamente Windows Terminal e riaprirlo.

---

## `code` non viene riconosciuto

Se compare:

```text
'code' non è riconosciuto come comando interno o esterno...
```

chiudere e riaprire Windows Terminal dopo l'installazione di VS Code.

Se il problema rimane, verificare l'installazione di Visual Studio Code e che il comando `code` sia disponibile nel PATH.

---

## VS Code utilizza il Python sbagliato

Premere:

```text
Ctrl + Shift + P
```

quindi:

```text
Python: Select Interpreter
```

e scegliere:

```text
corso-python
```

---

## Il terminale integrato di VS Code apre PowerShell invece di cmd

In VS Code è possibile selezionare il profilo del terminale desiderato.

Aprire il terminale integrato con:

```text
Ctrl + `
```

Dal menu del terminale scegliere:

```text
Select Default Profile
```

e quindi:

```text
Command Prompt
```

Aprire infine un nuovo terminale.

A questo punto il prompt dovrebbe avere una forma simile a:

```text
C:\CorsoPython>
```

e, dopo l'attivazione dell'ambiente:

```text
(corso-python) C:\CorsoPython>
```

---


# In caso di problemi con VS Code / Jupyter

Se VS Code non riconosce correttamente l'ambiente Conda, il kernel non compare oppure il notebook non viene eseguito come previsto, aprire la **Command Palette** con:

```text
Ctrl + Shift + P
```

e provare, nell'ordine, i seguenti comandi.

### 1. `Developer: Reload Window`

Ricarica la finestra di VS Code e riavvia l'interfaccia e le estensioni senza chiudere il progetto.

È il primo tentativo da fare quando VS Code sembra non aggiornare correttamente lo stato degli ambienti o delle estensioni.

---

### 2. `Python Environments: Refresh All Environment Managers`

Forza VS Code a rieseguire la ricerca degli ambienti Python disponibili.

È particolarmente utile quando:

- è stato appena creato un nuovo ambiente Conda;
- l'ambiente `corso-python` non compare;
- VS Code mostra un elenco di ambienti non aggiornato.

---

### 3. `Python: Select Interpreter`

Permette di scegliere esplicitamente l'interprete Python dell'ambiente Conda.

Nel nostro caso selezionare:

```text
corso-python
```

oppure una voce simile a:

```text
Python 3.12 (corso-python)
```

---

### 4. `Notebook: Select Notebook Kernel`

Permette di scegliere nuovamente il kernel utilizzato dal notebook.

Selezionare:

```text
Python Environments...
```

e quindi:

```text
corso-python
```

---

## Sequenza consigliata di troubleshooting

```text
Ctrl + Shift + P
        ↓
Developer: Reload Window
        ↓
Python Environments: Refresh All Environment Managers
        ↓
Python: Select Interpreter
        ↓
corso-python
        ↓
Notebook: Select Notebook Kernel
        ↓
Python Environments...
        ↓
corso-python
```

Se il problema persiste, verificare anche che nell'ambiente sia installato:

```cmd
conda install ipykernel
```

e che in VS Code siano installate e abilitate le estensioni Microsoft:

```text
Python
Pylance
Python Debugger
Python Environments
Jupyter
```


# Riepilogo dei comandi

```cmd
:: inizializzazione: normalmente una sola volta
conda init cmd.exe

:: creare l'ambiente del corso
conda create -n corso-python python=3.12

:: attivarlo
conda activate corso-python

:: isolare Python dai package installati a livello utente
set PYTHONNOUSERSITE=1

:: verificare Python
python --version
where python
python -c "import sys; print(sys.executable)"

:: installare i package del corso
conda install -c conda-forge --override-channels numpy pandas matplotlib scikit-learn

:: installare il kernel Jupyter
conda install ipykernel

:: entrare nella cartella del corso
cd C:\CorsoPython

:: avviare VS Code dalla cartella corrente
code .

:: al termine
conda deactivate
```

# Concetto fondamentale

Un **ambiente Conda** è un'installazione Python isolata.

Possiamo quindi avere sullo stesso computer ambienti diversi:

```text
base
│
├── corso-python      → Python 3.12 + librerie del corso
├── progetto-A        → altre librerie
└── progetto-B        → altra versione di Python
```

Questo riduce i conflitti tra versioni di Python e librerie e rende l'ambiente del corso più semplice da riprodurre e gestire.

---