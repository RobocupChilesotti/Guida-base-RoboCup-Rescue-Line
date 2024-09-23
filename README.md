# Guida-base-RoboCup-Rescue-Line
Breve guida per lo sviluppo di un robot CV-based per RoboCup Rescue Line
## Indice
* [Contatti](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#contatti)
* [Repo](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#repo)
* [Seguilinea](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#seguilinea)
  * [Concetti base](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#concetti-base)
    * [Maschere](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#maschere)
    * [Kernel](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#kernel)
    * [Acquisizione Immagini](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#acquisizione-immagini)
    * [Comunicazione Seriale](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#comunicazione-seriale)
    * [Altri elementi mancanti in example 2](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#altri-elementi-mancanti-in-example_2)
* [Hardware](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#Hardware) 
## Contatti
### Email
giovannimanzardo05@gmail.com
giovipegoraro@gmail.com

### Telegram
@Big_Gi0
@giovipeg

### Cell.
351 949 4991
366 727 0499

## Repo

Questa guida è basata sulle seguenti repo:

https://github.com/gpego/arduino-serial-com

https://github.com/gpego/Python-serial-com

https://github.com/B1gGi0/RescueLine_New

https://github.com/B1gGi0/Rescue-line

https://github.com/gpego/lineFollowing

https://github.com/gpego/tflite

https://github.com/gpego/robot1

https://github.com/gpego/newCode

https://github.com/lucaSartore/Robocup-Rescue-Line-simulation

https://github.com/lucaSartore/Dataset-for-robocup-rescue-line

## Seguilinea
Un'ottima spiegazione dell'algoritmo per il seguilinea si può trovare [qui](https://github.com/lucaSartore/Robocup-Rescue-Line-simulation/blob/main/line_follower_program.pdf).

La modalità più immediata per approcciarsi allo sviluppo del sw in Python è quella di basarsi su questa [repo](https://github.com/lucaSartore/Robocup-Rescue-Line-simulation), in particolare riteniamo che l'implementazione migliore si trovi nell'[example_2](https://github.com/lucaSartore/Robocup-Rescue-Line-simulation/blob/main/examples/example_2.py). Quest'ultimo fornisce già un'ottima base di partenza, le potenziali migliorie verranno listate in seguito.

La repo in questione contiene un simulatore che è consigliato dal momento che consente di velocizzare lo sviluppo, potendo testare il sw anche mentre l'hw non è ancora pronto.

Il sistema funziona come segue:

IN sensori -> processing dati -> valori velocità motori -> controllo motori

Dovrebbe essere familiare :).

Chiaramente il simulatore ignora il primo e l'ultimo passaggio, che perciò non sono presenti in example_2. Per comodità ora ci concentreremo sugli aspetti sw legati al Raspberry, trascurando momentaneamente l'Arduino (se presente).

### Concetti base
#### Maschere
Una 'maschera' è un'area (nel nostro caso è generalmente bianca, quindi composta da 'uni' della libreria `numpy` (`np.ones`), su sfondo nero, cioè 'zeri' della stessa libreria (`np.zeros`)) all'interno di un'immagine binaria, che quindi contiene esclusivamente pixel bianchi (1) o neri (0). Sulle maschere si possono compiere, tra le altre cose, tutte le operazioni binarie (`np.bitwise_or()`, `np.bitwise_and()`, ecc.). Le aree possono poi essere dilatate, erose, ecc. Queste ultime spesso richiedono l'uso di un kernel.

#### Kernel
Prendendo come esempio la dilatazione di un'area (le operazioni di dilatazione, erosione, ecc. si applicano all'intera immagine binaria e sono riferite poi alle aree bianche) i kernel possono essere immaginati come la misura dell'offset applicato ad una determinata area. Un kernel "verticale" dilaterà l'area verso l'alto e il basso, ma non a destra e sinistra, un kernel "orizzontale" farà l'opposto. Nella maggior parte dei casi i kernel hanno sia componente orizzontale che verticale.

### Acquisizione immagini
Questa procedura varia a seconda del Raspberry che usate (tecnicamente cambia a seconda del OS, ma questo è poi legato al modello di Raspberry). Assumendo di usare un Pi5, che è consigliato, [questa](https://github.com/B1gGi0/RescueLine_New/blob/main/line/acquire_stream_pi.py) è la procedura di acquisizione completa (con filtraggio e quantaltro) di un frame.

Come si può vedere una volta acquisita l'immagine, questa viene poi processata in maniera tale da diventare binaria (bianca e nera) con [questa funzione](https://github.com/B1gGi0/RescueLine_New/blob/main/line/acquire_stream_pi.py#L43).

[Qui](https://github.com/gpego/robot1/blob/main/prestito/RescueLineMondialiiV3/cattura.py) Sartore usava il multithreading per l'acquisizione video. Delegandola ad un thread separato liberava risorse per il processamento. Purtroppo non siamo riusciti ad adattare quel metodo al Pi5, non sappiamo se possa o no tornare utile. A meno che non si abbia familiarità con questa pratica suggeriamo di evitarla, almeno all'inizio, essendo il Pi5 più performante potrebbe non essere necessaria (come nel nostro caso). Ad ogni modo, se si rendesse necessaria maggiore potenza dedicata al processing delle immagini, questa potrebbe essere un'area su cui agire.

### Comunicazione seriale
Premessa: la miglior comunicazione seriale è nessuna connessione seriale.

Scherzi a parte, se si trovasse una qualsiasi via alternativa questa sarebbe con tutta probabilità preferibile al sistema attualmente in uso.

La comunicazione seriale tra Raspberry e Arduino è implementata in questi file ([fw Arduino](https://github.com/B1gGi0/RescueLine_New/tree/main/Arduino/main-0.0) e [sw Raspberry](https://github.com/B1gGi0/RescueLine_New/blob/main/Raspberry/Serial_communication/Serial_main.py)).

N.B.: il file relativo al Raspberry indicato sopra non è quello utilizzato nel sw finale, ma il principio di funzionamento è il medesimo perciò è la maniera più semplice di capire il funzionamento, dal momento che la versione finale contiene più elenti non immediatamente necessari che rischierebbero di creare confusione nel codice.

#### Vantaggi
Delegando le operazioni di IO all'Arduino si liberano risorse che vengono quindi allocate al processamento delle immagini. Quest'ultimo risulta dunque molto più veloce (FPS più alti). Queste considerazioni derivano da prove svolte esclusivamente sul Pi4.

#### Problematiche
La problematica principale della comunicazione seriale sta nel fatto che spesso il buffer si intasa e questo crea false letture e tutti gli imprevisti collegati.

#### Possibili miglioramenti
Al De Pretto sembrano adottare un sistema molto più affidabile rispetto al nostro, parlare con loro sarebbe dunque estremamente consigliato.

Rispetto al Pi4 il Pi5 non solo ha una CPU più performante, ma anche un nuovo chip (l'RP1) per la gestione del GPIO. Non sappiamo se, coombinati, questi due aspetti possano permettere di utilizzare direttamente il GPIO del Raspberry bypassando l'Arduino e tutte le problematiche associate alla comunicazione seriale. Avendo un po' di tempo potrebbe essere interessante fare una prova in questo senso.

### Altri elementi mancanti in example_2
#### Riconoscimento colori
Come detto in precendenza example_2 è un ottimo punto di partenza, la difficoltà reale è integrarlo con gli elementi mancanti, al di là degli aspetti hardware la mancanza principale riguarda gli incroci, che non sono implementati, questi sono trattati qui sotto. Per poter riconoscere gli incroci, però, è essenziale poter distinguere i colori sulla mappa, in particolare il verde legato ai quadrati degli incroci.

I dati sui quadrati vengono estrapolati ad esempio [qui](https://github.com/B1gGi0/RescueLine_New/blob/main/line/main.py#L42). Il funzionamento è il seguente `maschera_area_linea_senza_area_quadrati, maschera_area_quadrati = isolate_squares(img_a_colori, maschera_area_linea)`. A sua volta `isolate_squares()` funziona grazie ad `isolate_green()`, che si può trovare [qui](https://github.com/B1gGi0/RescueLine_New/blob/main/line/colors_menagement.py#L15). Quest'ultima ha bisogno di essere calibrata poichè il colore risente fortemente sia delle diverse condizioni di illuinazione, sia, più banalmente, del colore stesso dei quadrati, che varia anche di parecchio da una gara all'altra (es. a Vicenza era verde chiaro, a Verbania era verde scuro). Tra le mattonelle esiste la "mattonella dei verdi", cioè quella che ha diverse tonalità di verde per la calibrazione, è molto utile.

La calibrazione si effettua rispetto ai parametri dello spazio colore HSV, nel caso in esempio a riga [7](https://github.com/B1gGi0/RescueLine_New/blob/main/line/colors_menagement.py#L7) e [8](https://github.com/B1gGi0/RescueLine_New/blob/main/line/colors_menagement.py#L8), l'immagine qui sotto aiuta a capire il funzionamento dell'HSV e la ragione per cui viene usato (è più semplice per rappresentare range di tonalità).

![HSV_color_solid_cylinder_saturation_gray](https://github.com/user-attachments/assets/52e5aaa9-8195-4441-ba84-8ff9bb6f4a2f)

Per effettuare materialmente la calibrazione esiste [questo tool](https://github.com/B1gGi0/RescueLine_New/blob/main/line/ColorPickerScript.py) molto comodo. Un esempio del suo utilizzo si può trovare [qui](https://youtu.be/6WcMva128WI?feature=shared&t=422).

N.B.: nel nostro software l'area colorata era isolata quando era di colore bianco su sfondo nero. Non è una regola e si può benissimo scegliere di fare il contrario in future versioni se per qualsiasi motivo dovesse essere più comodo.

#### Incroci
Nella guida menzionata sopra è presente un [video](https://www.youtube.com/watch?v=Njx3aAMHUKc) che spiega l'algoritmo per risolvere il problema degli incroci, quell'algoritmo è stato implementato [qui](https://github.com/B1gGi0/RescueLine_New/blob/main/line/turn_utils2.py#L45).

N.B.: la funzione `crossing_logic()` va calibrata. La calibrazione è da effettuare nei kernel, definiti tra [linea 23](https://github.com/B1gGi0/RescueLine_New/blob/main/line/turn_utils2.py#L23) e [linea 32](https://github.com/B1gGi0/RescueLine_New/blob/main/line/turn_utils2.py#L23). Per comprendere al meglio il processo di calibrazione è opportuno vedere per prima cosa il video di cui sopra.

La calibrazione è effettuata in relazione allo spessore della linea nell'immagine: a seconda di divesri parametri (risoluzione dell'immagine, altezza della telecamera, % di frame tagliata, ecc.) la dimenisione media della linea (in pixel) varia grandemente. I kernel vanno tarati in maniera tale da ricoprire la giusta porzione di linea per permettere le diverse operazioni di intersezione tra le diverse aree.

## Hardware
Per quanto riguarda l'aspetto fisico del vostro robot le cose da tenere in considerazione sono molte, da tenere sempre bene a mente nella progettazione è la [legge di Murphy](https://it.wikipedia.org/wiki/Legge_di_Murphy). Per questo dovete essere in grado di cambiare qualsiasi pezzo del robot, anche durante le gare, in meno di 30 minuti. Questo vincola due aspetti fondamentali per i quali lottare:
* Avere un buon disegno che permetta uno smontaggio semplice di qualsiasi pezzo
* Avere **pezzi di ricambio** (lottate con le istituzioni)
### Disegno
I file sono disponibili su [WeTransfer](aggiungere_link) fino al inserire_data. Sono file SolidWorks ma dovrebbero poter essere aperti con Fusion, ad ogni modo, in caso di necessità li possiamo convertire.

Se si volesse reverse-engineerizzare il nostro robot, anche avendo i file CAD sarebbe consigliabile smontarlo per capire come funziona. Smontarlo è complicato, in fase di progettazione, se possibile, sarebbe opportuno creare un robot che sia relativamente FACILE da smontare ed i cui componenti siano FACILMENTE accessibili per la manutenzione.

Per smonralo si procede così:
- si svitano i bulloni sul coperchio (c'è anche una vitina sul foro superiore del servo della telecamere, la si svita dal lato)
- ora il robo di può aprire  e si hanno due metà. Si disconnettono i cavi che vanno dalla millefori alla parte superiore
- ora è possibile smontare i diversi componeti
- a questo punto di possono staccare le staffe dei motori
- per staccare i motori dalle staffe bisogna allentare i bulloni M4 paralleli al terreno che si trovano subito sopra i motori

Il disegno è specifico per i nostri componenti, la modifica è complicata a causa delle relazioni tra i vari componenti. Tuttavia ci sono alcuni aspetti che si potrebbero riutilizzare. In particolare, a parità di motori e batteria potrebbe essere interessante riutilizzare il pianale e le relative staffe per i motori, in questo caso però emerge una criticità. La composizione riesce ad essere molto compatta in questa maniera, a scapito della praticità: per smontare un motore è necessario smontare entrambe le staffe che lo sostengono. Questo è possibile solamente svitando i bulloncini dalla parte superiore del pianale, che nel nostro caso era coperto di componenti a loro volta avvitati. In sostanza era difficilissimo smontare ed eventualmente sostituire un motore. Ovviando a questo problema, tuttavia, la base sarebbe valida dal momento che un robot compatto presenta numerosi vantaggi dal momento che permette dtolleranze maggiori a livello di scostamento del robot dalla linea a livello sw.

Se si decidesse di usare una base di questo tipo un altro elemento valido sarebbe il posizionamento dei servo frontali. Nel nostro caso la cover di plastica della parte inferiore è stata sostituita con una stampata in 3D che si interfacciasse con il resto della struttura.

Per quanto riguarda il coperchio: non è necessario inclinarlo inoltre l'altezza può essere aumentata. È stato tenuto così basso per MANIE DI PERFEZIONISMO CHE È OPPORTUNO NOM FARSI VENIRE. Più in generale è necessario tenere sempre in considerazione spazio extra per i cablaggi, che sono parecchio voluminosi. Il miglior modo di ottenere spazio è verso l'alto, ovviamente facendo attenzione a non spostare troppo il baricentro (soprattutto per le altalene) e a non ecedere l'altezza max. indicata nel regolamento (meglio tenere una soglia di sicurezza e farlo un po' di cm più basso).

I cassoni funzionano bene.

La pinza è carina, anzi, esteticamente è molto bella MA QUELLO NON DEVE ESSERE L'OBBIETTIVO. È comoda perchè ha un'apertura ampia, quindi di nuovo richiede meno precisione, il problema è che, quando la pallina è di fianco ad una parete (e succede più spesso di quanto si immagini) la pinza sbatte contro il muro e di conseguenza non riesce a prendere la pallina.

### Circuiti
La nostra soluzione è stata la creazione di una scheda millefori che facesse da shield all'arduino, così da riuscire a sostituire l'arduino velocemente in caso di rottura. La soluzione migliore sarebbe avere due pcb (una montata e una di scorta). Tuttavia non si deve perdere troppo tempo nel disegno della scheda stampata o nel cablaggio (specialmente se nel team si è in pochi).
