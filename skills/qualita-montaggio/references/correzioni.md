# Lezioni imparate (montaggio)

Errori già fatti e regole che ne sono uscite. **Aggiungi in cima una riga per ogni correzione ricevuta**:
`data · problema · regola`. Qui sotto trovi un punto di partenza, raggruppato per tema.

## Sottotitoli
- **Nessun buco tra una frase e la successiva.** La fine di ogni evento è l'inizio del successivo: un fotogramma vuoto in mezzo fa sfarfallare i sottotitoli. Chiudi anche i buchi sotto gli 80 ms ai tagli tra segmenti. Verifica su TUTTI i fotogrammi (ritaglia la fascia dei sottotitoli con `crop`+`scale`+`format=gray`, misura l'altezza del box per fotogramma): guardare qualche frame non basta.
- **Non toccare il testo quando il problema è l'effetto.** Se l'utente dice che i sottotitoli "entrano male", il testo di solito è giusto: controlla tempi e animazione.
- **Un sottotitolo che resta a schermo meno di 0,5 s è illeggibile**: uniscilo al successivo (massimo 4 parole o ~24 caratteri).
- **Larghezza**: misura la larghezza reale del testo con lo stesso font (canvas di Chrome) e spezza a ≤ 820 px su 1080. Mai due eventi di testo sovrapposti nello stesso layer di libass: il secondo finisce fuori dal box.
- **Box dei sottotitoli**: se l'utente non vuole il fondo colorato, usa un grigio semitrasparente. In ASS disegna il box in un layer separato (stile Box, testo trasparente, un evento per frase) e il testo con la parola attiva sopra: con un solo evento a più colori i box si sovrappongono e fanno bande scure.
- **Parola attiva evidenziata** in ogni video: sottotitoli senza evidenziazione sembrano piatti.
- **macOS**: il font "Avenir Next" in libass esce in corsivo; usa "Helvetica Neue" Bold.
- **Sorgenti con titoli già scritti in basso** (per esempio video inoltrati da app di messaggistica): alza i sottotitoli.

## Trascrizione
- Per l'italiano usa `whisper-cli -m <modello>/ggml-small.bin -l it -ml 1 -sow -ojf`: il default di `hyperframes transcribe` è inglese.
- **Le frasi dubbie si ri-trascrivono con un modello più grande** (per esempio `large-v3-turbo`) prima di metterle nei sottotitoli: il modello `small` può inventare parole e saltarne altre.
- I tempi per parola su audio lungo derivano (anche di ~1 s): ri-trascrivi finestre corte (≤ 60 s) e taglia solo sui silenzi (`silencedetect` + inviluppo RMS).
- Usa `--prompt` con i nomi propri e controlla i tratti dubbi riascoltandoli isolati.
- Spezza l'audio alle pause (chunk ≥ 2,5 s), limita la fine di ogni parola (0,25 s + 0,06 s/lettera) e spezza i gruppi alle pause > 0,45 s.

## Inquadratura e composizione
- **Mai scritte grafiche sul volto**: usa la fascia del petto, sopra i sottotitoli. Il check tecnico passa anche con i titoli sul volto: non sostituisce la lettura dei frame.
- **Non ripetere come grafica la stessa parola che è già nei sottotitoli**: usa icone o numeri che aggiungono significato.
- **16:9 → 9:16**: un crop centrale fisso può tagliare il volto o inquadrare un muro, soprattutto se la sorgente ha più camere. Prima estrai un frame per inquadratura (`select='gt(scene,0.06)'` + contact sheet) e decidi il layout.
- **Ritagli di ritratti**: la testa intera deve entrare, con margine sopra e mento incluso. Controlla ogni ritaglio con Read prima del render.
- **Foto a bassa risoluzione**: sostituiscile con ritratti HD (con credito), non ingrandire un ritaglio.
- **Il clip non deve aprirsi con una frase che rimanda a qualcosa non detto**: parti dalla frase che introduce l'argomento.
- **Cerchio o evidenziatore disegnato sul testo**: un SVG con `vector-effect:non-scaling-stroke` + `pathLength` rompe il tratteggio (si vede solo un arco) e uno SVG stirato su parole larghe deforma lo spessore. Meglio un elemento con `border` + `border-radius` ovale, rivelato con `clipPath:inset(0 100% 0 0)` → `inset(0 0 0 0)`.
- **Contrasto**: i colori di accento vanno controllati prima di usarli come testo (un giallo su giallo, un corallo scuro). Una pillola di titolo con testo bianco su giallo ha contrasto 1,3:1.
- **Font reali scaricati**, mai il fallback di sistema.

## Animazione (GSAP / HyperFrames)
- `tl.fromTo` ha `immediateRender:true`: su elementi che partono già visibili (zoom, anelli) applica lo stato iniziale dall'inizio del video. Usa `immediateRender:false` con `opacity:0` iniziale, e nascondi con `gsap.set` quello che deve entrare dopo.
- Un `transform:` statico in CSS (per centrare) su un elemento animato con GSAP viene sovrascritto: centra con `xPercent`/`yPercent` in `gsap.set` o con un contenitore flex, mai mischiando le due tecniche.
- Nei file di frame di HyperFrames i path degli asset sono relativi alla radice del progetto (`assets/...`), mai `../../assets/...`.
- Non animare `letter-spacing` (usa `scaleX`) né `left/top/width/height` (usa `x`, `y`, `scale`).
- Ogni elemento fuori da un gruppo che esce va animato in uscita insieme al gruppo, altrimenti una linea resta in scena dopo che il testo è sparito.
- **Mai rallentare con `data-playback-rate` una sorgente a 30 fps**: i fotogrammi si duplicano e scatta. Riproduci a 1x e allunga con fermi immagine sui momenti chiave, preprocessati con ffmpeg (con un leggerissimo zoom).
- Il fermo immagine finale va estratto dall'ultimo fotogramma esatto del video: misura la differenza tra fotogrammi consecutivi per non vedere lo stacco.
- Camera che zooma: limitala ai bordi (clamp) per non mostrare bordi vuoti.

## Ritmo
- Ogni informazione chiave ha il suo tempo da sola, con camera ferma o lenta. L'ultima scena informativa merita 6-8 s.
- Su una lista di persone ogni persona resta ferma almeno ~0,45 s, ingrandita, con le altre attenuate.
- Le scene sono più lunghe di quanto sembri necessario (15-25%).
- Zoom a caso non servono: ogni zoom parte quando l'elemento compare e finisce su di esso.
- Niente effetti generici (coriandoli): usa effetti legati al contenuto (evidenziatore sul numero, riflesso sul logo).

## Audio
- **SFX**: i suoni sintetici di libreria sono spesso troppo deboli (picchi a −22 dB). Generali o normalizzali a −12/−14 dBFS di picco, uno diverso per ogni evento.
- Gli "accenti" sonori sono gli effetti su transizioni e movimenti: consegna anche la versione solo accenti e fai il ducking della musica sotto i suoni.
- **Musica**: non deve restare "sciapa". Fai crescere piano piano il bed (pad che si riempie, campanelle più fitte, battito morbido) da ~−36 a ~−24 LUFS sotto la voce a −16. Per un ritmo vero servono riff, basso sincopato, sidechain sulla cassa, riser/impatti sui tagli, tagli sulle battute.
- Quando riusi l'outro di un altro video, taglia sul PRIMO frame pulito (misura la luminanza per frame con `signalstats`) e usa solo gli accenti sonori, non il suo fondo.
- **TTS in italiano**: forza la lingua (`--lang it`) anche per i tempi parola; le parole inglesi e i numeri a cifre vanno riscritti foneticamente SOLO nel testo dato alla sintesi (mai a schermo) e verificati ri-trascrivendo la frase di prova.
- Master finale: `loudnorm` a −14 LUFS, picco −1,5 dBFS (a due passaggi, con `linear=true`).

## Processi lunghi
- Mai lanciare ffmpeg o whisper lunghi con `&` o in pipe dentro un comando in background: il processo muore o si duplica e due render sullo stesso file lo corrompono. Un solo comando in background, output su un nome nuovo.
- Prima di promettere tempi su video lunghi con operazioni pesanti (per esempio `hyperframes remove-background --quality best`, ~45-50 s di calcolo per 1 s di video), misura 10-15 s reali e proietta: le stime dei README possono essere sbagliate di un ordine di grandezza.
- Per unire clip che si sovrappongono: allinea con l'inviluppo audio e taglia in una pausa tra due parole, mai a metà parola.

## Stile
- Non portare lo stile di un brand in un altro: usa sempre palette, font e tono catturati dal sito reale del cliente. Un riferimento di processo (brief → storyboard → controllo frame) non è un riferimento visivo da copiare.
- Tra due fonti di riferimento, scegli il preset in base al brief del cliente, non al progetto più recente.
