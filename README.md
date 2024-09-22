# Guida-base-RoboCup-Rescue-Line
Breve guida per lo sviluppo di un robot CV-based per RoboCup Rescue Line
#Indice
* [Repo](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#repo)
* [Seguilinea](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#seguilinea)
* [Concetti base](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#concetti-base)
  * [Maschere](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#maschere)
  * [Kernel](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#kernel)
  * [Acquisizione Immagini](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#acquisizione-immagini)
  * [Comunicazione Seriale](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#comunicazione-seriale)
  * [Altri elementi mancanti in example 2](https://github.com/NettunoRobotica/Guida-base-RoboCup-Rescue-Line/blob/main/README.md#altri-elementi-mancanti-in-example-2)

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
