# gioco-digitale

Istruzioni per Claude: costruire il **prototipo digitale per il playtest** di un mio gioco da tavolo (motore con test, interfaccia da gioco da tavolo, AI a più livelli, simulatore, regolamento sincronizzato con l'app).

**Come si usa:** apri una sessione con questo repository, con il **repository nuovo e vuoto** del gioco e con i repository di riferimento, allega il file con le regole del gioco, poi scrivi:

> *«Segui le istruzioni nel repo gioco-digitale. Il gioco è [NOME], il file con le regole è allegato.»*

Dopo il digitale, per il kit stampabile: repository `stampa-gioco`.

Riferimenti già fatti (da leggere prima di scrivere codice):
- **Fifo-Fafo** — `olivieridomenicopodcast/Fifo-Fafo-playtest-`, branch `ccr-ef847750-u6ouub`: l'architettura di base.
- **Cromozapd** — `olivieridomenicopodcast/cromozapd-`, branch `ccr-5b3c90aa-gh0360`: la versione più evoluta (3 giocatori, regole di colore, punteggio a podio, AI tarata, kit stampabile, `docs/DA_RICORDARE.md`).

---

## ISTRUZIONI (da seguire alla lettera)

Voglio costruire un **prototipo digitale per il playtest** del mio gioco **[NOME DEL GIOCO]**, in un repo nuovo e vuoto. In allegato c'è **[NOME-FILE.md]** con tutto quello che so finora: regole, carte, dubbi aperti. Parla e scrivi **sempre in italiano**.

### Riferimento: parti da un lavoro già fatto
Aggiungi in sola lettura, clona e studia **prima di scrivere codice** i due repo di riferimento qui sopra: voglio la stessa filosofia e la stessa qualità, adattate a questo gioco. Usa Fifo-Fafo per l'architettura di base e Cromozapd per le parti più recenti (simulatore, AI, punteggio, stampabili). Guarda in particolare README.md, js/engine.js, js/ai.js, js/sim.js, js/ui/*, tests/, tools/, docs/REGOLAMENTO.md e docs/DA_RICORDARE.md. Riusa architettura e idee, **non copiare le regole** di quei giochi.

### Come lavoriamo (importante)
1. **Prima di tutto leggi l'MD e dimmi cosa trovi**: lacune, contraddizioni e punti ambigui delle regole. Fammi poche domande mirate e **aspetta le mie risposte**: non inventare regole. Ogni interpretazione va scritta nel regolamento.
2. **Lavora a piccoli passi** e a ogni passo **verifica davvero**: test automatici sulle regole e prova nel browser (Chromium e Playwright sono già installati: gioca partite complete con un bot, fai screenshot su desktop **e su telefono**, a ogni cambio di interfaccia). Dimmi con onestà cosa hai verificato e cosa no.
3. Sviluppa e fai push solo sul branch che ti viene indicato. **Niente PR** se non te le chiedo.
4. Quando ti incollo un log o ti segnalo che qualcosa "non ha senso": **prima analizza e dimmi cosa vedi di strano, poi proponi, poi aspetta la mia decisione** prima di cambiare le regole. Se cambi una regola, aggiorna regolamento, test, simulatore **e ritara l'AI** (vedi sotto).
5. **Giocabile al tavolo vero.** Il gioco deve funzionare anche con carte e segnalini reali. Ogni regola deve avere poco carico mentale, controlli semplici al momento di scoprire le carte, **niente punti "fluttuanti" da ricordare**, conteggi con sole somme e confronti. Se una regola funziona bene nell'app ma al tavolo sarebbe pesante o facile da sbagliare, **segnalamelo**.
6. **Misura prima di proporre una regola.** Una regola nuova o cambiata si prova in simulazione prima di adottarla, e mi porti i numeri con l'intervallo di confidenza: durata della partita (media **e distribuzione**), vantaggio di posto o di chi inizia, quanto vince l'AI forte contro le deboli (la **abilità deve contare**), e tutto ciò che è specifico del gioco (es. quanto spesso si "sfora", quanto spesso succede un certo evento). All'inizio fissiamo insieme gli **obiettivi di bilanciamento** (per esempio "nessun posto sopra il 36% di vittorie") e li uso come riferimento.
7. **Le regole sono mie.** Segna nel regolamento ogni punto come **[chiarito]** (deciso con me) o **[interpretazione]** (scelta tua, da confermare al playtest). Se una mia idea ti sembra problematica, **prima misurala**, poi dimmelo con i numeri: non scartarla né cambiarla da solo.
8. **Memoria del progetto.** Tieni due file: `docs/DA_RICORDARE.md` (misure, note di bilanciamento, cose da riguardare al playtest) e `docs/DA_IMPLEMENTARE.md` (idee decise ma non ancora fatte, con lo stato). Le carte o le regole eliminate restano nel codice come varianti a zero copie / parametri spenti, così si possono riprovare.

### Architettura
- **Nessuna build**: HTML/CSS/JS vanilla con script classici, che funzionano aprendo index.html e anche in Node. **PWA installabile e offline** (a ogni modifica ai file in cache, incrementa la versione del service worker).
- **Motore di gioco separato dall'interfaccia**: la partita è un generatore che chiede "decisioni"; stato JSON clonabile; **RNG con seed**; ogni partita è riproducibile e rigiocabile dal seed o dalla cronologia delle risposte (serve per salvataggio/ripresa e per "Rivedi").
- **Informazione nascosta** (mani, mazzo, carte coperte): l'AI non deve mai "sbirciare". Per le decisioni difficili usa il motore come modello in avanti con simulazioni su mani possibili, senza leggere i dadi o le carte future vere.
- **Parametri delle regole in un unico oggetto**, modificabili dall'interfaccia (per fare esperimenti).
- **Test con node:test**: un test per ogni regola importante, test di determinismo e replay, test "fuzz" con invarianti (ad esempio nessuna carta si perde o duplica).
- **Regolamento unico** in docs/REGOLAMENTO.md (con le parti nuove o chiarite segnate e una tabella delle differenze rispetto all'originale), mostrato anche nell'app alla voce Regole e generato dallo stesso file. Test che controlla che il testo mostrato sia sincronizzato e che i valori citati (quantità di carte, parametri) coincidano col codice.

### Modalità di gioco
Adatta al numero di giocatori del gioco (se non è 2, dimmi come le proponi):
1. **Contro l'AI** (facile, media, difficile) con suggerimento dell'AI facoltativo e anteprima delle conseguenze.
2. **Più giocatori umani sullo stesso dispositivo** ("passa il telefono"): schermata di passaggio prima di ogni scelta privata, così gli altri non vedono la mano.
3. **AI contro AI**, da guardare: velocità regolabile.
4. **Simulazione veloce** dentro l'app (vedi sotto).

Salvataggio automatico con ripresa; log (cronaca) completo, esportabile; possibilità di aggiungere **note di playtest** al log.

### Interfaccia: i miei requisiti
- **Look da gioco da tavolo**: tavolo, carte vere (ben disegnate, con il testo leggibile), zone di gioco chiare (mano, mazzo, scarti, tavolo). **Sprite e grafica generati da te in SVG**, senza risorse esterne; esportali anche come file. Disegnali già pensando che dovranno poter essere **stampati** (carte con rapporto 100×140, uguale a quello di una carta da poker).
- **Tutto leggibile**: testo grande (base almeno 17px), icone grandi, buon contrasto, responsive per telefono. Le carte si possono ingrandire con un tocco.
- **Legenda** sempre disponibile con tutte le icone e i simboli (a sinistra della plancia su schermi larghi).
- **Messaggi grandi sopra al tavolo per ogni evento** (anche le mosse dell'AI e gli effetti delle carte), che **faccio avanzare io** con un bottone, un clic sul messaggio o Spazio/Invio. La modalità predefinita è manuale, con un tasto "salta fino alla mia mossa". I cambi di fase, round o turno chiedono **sempre** il clic. Ogni effetto deve spiegare **cosa fa e perché**.
- **A colpo d'occhio, senza cliccare**: cosa sta per succedere (carta o effetto attivo e prossimo), le risorse e i contatori di tutti, di chi è il turno. Gli stati speciali vanno descritti sempre, anche quando non sono attivi.
- **Le azioni non disponibili non spariscono**: restano visibili e grigie con il motivo. Prima di una mossa costosa o rischiosa, **avviso evidente e conferma**.
- Cronaca di tutti gli eventi sempre consultabile.

### Il simulatore (dentro l'app e da riga di comando)
- N partite AI vs AI con due profili A e B, **posti alternati**, vittorie con **intervallo di confidenza**, vantaggio di chi gioca per primo, andamento dei punteggi nel tempo, distribuzione degli scarti, **durata delle partite (media e distribuzione)**.
- **Quanto rende ogni carta o effetto**, medie degli eventi per partita, elenco partite con log e "Rivedi" dal seed.
- **Esperimento sulle regole**: cambia un parametro e confronta i risultati.
- **Analisi di ogni carta/obiettivo "forzata"** per tarare i valori, anche per quelle che l'AI non sceglie mai da sola.
- **Prove "estreme"**: strategie sbagliate di proposito (sempre il massimo, solo egoista…) devono perdere. Se non perdono, la regola ha un buco.
- Esporta in Markdown, CSV e JSON. Prima di adottare un cambiamento all'AI, misuralo con un torneo e un intervallo di confidenza.

### Qualità dell'AI
Tre livelli ben distinti (facile < media < difficile, misurati col simulatore). Dopo averli fatti, **leggi tu stesso i log di partite difficile contro difficile** e correggi le scelte senza senso (azioni sprecate, risorse buttate, mosse che si annullano). **Ogni volta che cambia una regola di punteggio o di struttura**, ricontrolla che l'AI stimi il valore delle mosse nel modo giusto e rimisura i tre livelli. Se vedo io qualcosa di strano te lo segnalo.

### Primi passi
1. Clona i repo di riferimento e leggi il mio MD.
2. Dimmi le ambiguità e le domande sulle regole, e proponi una **struttura del progetto, gli obiettivi di bilanciamento e un piano a tappe** adatti al mio gioco (adattando quelli di riferimento).
3. Aspetta il mio ok, poi parti dal motore e dai test, e solo dopo interfaccia, AI e simulatore.

### Alla fine di ogni tappa
Riassumi con onestà: cosa è fatto, cosa hai verificato (test, partite nel browser, telefono, simulazioni con i numeri) e cosa **non** hai verificato, cosa ti sembra dubbio nelle regole. Quando il digitale è stabile e voglio giocare con carte vere, seguirà il kit stampabile: istruzioni nel repo `stampa-gioco`.
