---
name: qualita-montaggio
description: Protocollo di qualità per OGNI video, montaggio, reel o grafica animata. Usare SEMPRE, prima di qualsiasi richiesta di montaggio, grafiche, sottotitoli, promo o reel, anche se l'utente non lo nomina. Impone riferimento di stile, brief, storyboard, controllo visivo dei frame e correzioni salvate, per evitare grafiche scadenti da template.
---

# Qualità montaggio

I video che funzionano seguono sempre lo stesso metodo: fonte visiva reale, brief chiaro, storyboard, frame costruiti uno a uno, controllo tecnico e visivo prima della consegna. Il metodo qui sotto è obbligatorio per ogni video, anche se piccolo. Salta un passo solo se l'utente lo chiede.

## 0. Prima di toccare qualsiasi cosa
1. Leggi `APPUNTI.md` della cartella di progetto (se esiste) e `references/correzioni.md` (errori già fatti e regole imparate).
2. Se hai una libreria di effetti o template (per esempio la galleria di HyperFrames), cercala prima: gli effetti si riusano, non si inventano da zero.

## 1. Riferimento di stile (mai partire dai default)
Serve una di queste fonti, in ordine:
- un reel/sito/account indicato dall'utente → catturalo (`npx hyperframes capture`, o screenshot/frame con ffmpeg) e ricava colori, font, ritmo, tipo di sottotitoli;
- una skill di marca già esistente (loghi, persone, palette del brand);
- se non c'è niente: **una sola domanda** all'utente ("hai un reel o uno stile che ti piace come riferimento?"). Se risponde "fai tu", scegli UNO stile dal catalogo sotto e dichiaralo in una riga.

### Catalogo stili (scegline uno, non mescolarli)
| Stile | Quando | Caratteristiche |
|---|---|---|
| Hormozi | parlato, business, educativo | sottotitoli grandi 2-3 parole, parola attiva gialla/verde, sfondo scuro, zoom a ogni taglio |
| Editoriale | cultura, istituzioni | serif, molto spazio bianco, una palette di 2-3 colori, animazioni lente e sobrie |
| Dev-tool | software, promo di prodotti | monospace, terminale, prima→dopo, accento colore unico |
| Creator pop | reel virali, intrattenimento | colori saturi, sticker, glitch/scossa, SFX, ritmo veloce |
| Notizia | news, titoli-notizia | lower third, barra titoli, font grotesque, grafica pulita |

## 2. Brief in 5 righe (scrivilo in `APPUNTI.md`)
Obiettivo · pubblico · formato (9:16 / 16:9) · durata · stile scelto + fonte del riferimento.

## 3. Storyboard prima del codice
Elenca le scene con tempo, cosa si vede, cosa si legge, quale effetto/transizione. Regole:
- un solo messaggio per scena; un solo elemento "protagonista" alla volta;
- gancio nei primi 2 secondi; cambio visivo ogni 2-4 secondi;
- ultima scena = chiamata all'azione chiara;
- progetta tutto nelle safe zone (lascia libero ~15% in alto e ~20% in basso nei 9:16).

## 3b. Analisi della sorgente (prima di ritagliare o trascrivere)
- Se la sorgente è 16:9 e va in 9:16: trova i tagli di camera (`ffmpeg -vf "select='gt(scene,0.06)',showinfo"`), estrai un frame per inquadratura, guardali e scegli il layout (crop seguendo il volto, oppure video a tutta larghezza su sfondo sfocato). Mai un crop centrale fisso senza averlo verificato.
- Trascrizione italiana: `whisper-cli ... -l it` (il default di `hyperframes transcribe` è inglese). Dopo la trascrizione correggi i nomi e controlla la sincronia sui frame. Le frasi dubbie si ri-trascrivono con un modello più grande prima di metterle nei sottotitoli.
- Taglio: usa sempre i tempi parola-per-parola della trascrizione, mai a occhio — un taglio non cade mai a metà parola. Tieni l'ultima ripresa buona quando ce ne sono più di una, togli le pause sopra ~0,25 s, lascia un piccolo margine ai bordi del taglio e applica 30 ms di dissolvenza audio su ogni punto di taglio per evitare i click.

## 4. Costruzione
- Strumento: HyperFrames (`/hyperframes` come entrata, `/montamelo` o equivalente per footage grezzo). Remotion solo se serve (mappe, dati).
- Scene indipendenti in sotto-composizioni; per video lunghi costruiscile in parallelo con agenti.
- Un font, una palette, un'unica logica di animazione per tutto il video. I font reali vanno scaricati, non lasciati al fallback.
- Immagini vere (foto, screenshot, loghi) invece di segnaposto. Niente emoji come grafica.
- Audio: un effetto sonoro diverso per ogni evento visivo (non riusare lo stesso suono per cose diverse), tutti gli SFX allo stesso livello tra loro e a circa −7 dB sotto la voce — mai così bassi da non sentirsi, mai così alti da coprire il parlato.

## 5. Controllo obbligatorio prima di consegnare
Non mostrare mai un video che non hai guardato.
1. `npx hyperframes check` → 0 errori (lint, runtime, layout, contrasto).
2. `npx hyperframes snapshot --at <6-8 tempi>` oppure estrai frame con ffmpeg, e **aprili con Read** uno a uno.
3. Per ogni frame rispondi: testo leggibile a dimensione telefono? niente testo tagliato o fuori safe zone? niente sovrapposizioni con volto o altro testo? contrasto sufficiente? lo stile è coerente con gli altri frame?
3b. Per animazioni, entrate e sottotitoli dinamici i frame fermi non bastano: registra l'anteprima (`screencast_start` / `screencast_stop` di chrome-devtools) oppure estrai una sequenza fitta (es. 10 frame al secondo attorno a ogni entrata, `ffmpeg -ss <t> -t 1.5 -vf fps=10`) e guardala. Controlla ritmo, entrata/uscita, sincronia della parola attiva con l'audio.
4. Correggi e riguarda. Fai almeno un giro di correzione dopo la prima versione.
5. Render finale, controlla durata, audio (`loudnorm` a target −14 LUFS, voce sempre davanti agli effetti), peso del file.

## 6. Consegna e memoria
- Dì in poche righe: file, durata, stile usato, cosa hai corretto, cosa si può cambiare.
- Ogni correzione dell'utente va **subito** aggiunta a `references/correzioni.md` (data, problema, regola). Se riguarda un brand con una skill dedicata, aggiornala anche lì.
- Aggiorna `APPUNTI.md` della cartella.

## Segnali di grafica scadente (da evitare)
Font di sistema, testo bianco piccolo su foto chiara, ombre pesanti, animazioni identiche ripetute su ogni scena, più di tre colori, testo che compare tutto insieme senza ritmo, sottotitoli senza evidenziare la parola attiva, segnaposto, elementi non allineati, ritmo piatto senza zoom né cambi.
