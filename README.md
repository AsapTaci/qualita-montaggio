# qualita-montaggio

**Una skill per [Claude Code](https://claude.com/claude-code) che impedisce i video scadenti da template.**
Prima di montare qualsiasi video, reel o grafica animata, Claude segue un protocollo fisso: riferimento di stile, brief, storyboard, controllo dei fotogrammi e un diario degli errori che cresce a ogni correzione.

> Non è un editor video: è il **metodo** che Claude applica sopra gli strumenti che già usi (HyperFrames, Remotion, ffmpeg, whisper).

## Il problema

Chiedi a un'IA "fammi un reel" e ottieni quasi sempre la stessa cosa: font di sistema, testo bianco piccolo su foto chiara, la stessa animazione su ogni scena, sottotitoli senza ritmo, titoli sopra il volto. E il controllo tecnico passa lo stesso, perché un video può essere "valido" ed essere brutto.

Questa skill sposta il lavoro da "genera e consegna" a **"riferimento → piano → costruzione → guardo davvero i fotogrammi → correggo"**.

## Come funziona

| Passo | Cosa impone |
|---|---|
| **0. Prima di tutto** | Legge gli appunti del progetto e il diario delle lezioni imparate; cerca effetti già pronti prima di inventarli. |
| **1. Riferimento di stile** | Mai partire dai default: cattura un reel/sito indicato da te, usa la skill del tuo brand, oppure ti fa **una sola domanda**. Se rispondi "fai tu", sceglie UNO stile da un catalogo (Hormozi, Editoriale, Dev-tool, Creator pop, Notizia). |
| **2. Brief in 5 righe** | Obiettivo · pubblico · formato · durata · stile e fonte, salvati in `APPUNTI.md`. |
| **3. Storyboard** | Scene con tempi, un solo messaggio per scena, gancio nei primi 2 secondi, safe zone per i 9:16. |
| **3b. Analisi sorgente** | Tagli di camera, un frame per inquadratura, trascrizione in italiano con tempi parola per parola, tagli mai a metà parola. |
| **4. Costruzione** | Un font, una palette, una logica di animazione; font reali scaricati; immagini vere; un effetto sonoro diverso per ogni evento. |
| **5. Controllo** | `check` tecnico a 0 errori, poi **apre i fotogrammi uno a uno** e risponde a una checklist (leggibile da telefono? testo sul volto? contrasto?). Per le animazioni guarda una sequenza fitta, non solo frame fermi. Poi render e master a −14 LUFS. |
| **6. Memoria** | Ogni correzione che ricevi finisce subito nel diario `references/correzioni.md`, così l'errore non si ripete. |

Include anche una lista di **segnali di grafica scadente** da evitare.

## Installazione

```bash
git clone https://github.com/AsapTaci/qualita-montaggio.git
cp -R qualita-montaggio/skills/qualita-montaggio ~/.claude/skills/
```

Riapri Claude Code: la skill compare tra quelle disponibili. Si attiva da sola quando chiedi un video, un reel, sottotitoli o una grafica animata.

## Come si usa

Non serve invocarla. Basta chiedere:

- "Montami questo video in verticale con i sottotitoli."
- "Fai un promo di 30 secondi per questo prodotto."
- "Imita lo stile di questo reel: *link o file*."

Claude parte dal riferimento (o ti chiede di indicarlo), ti mostra brief e storyboard, costruisce, **guarda i fotogrammi** e ti consegna con un riepilogo di cosa ha corretto.

## Cosa c'è nel repo

```
skills/qualita-montaggio/
├── SKILL.md                    # il protocollo (si carica ogni volta)
└── references/
    └── correzioni.md           # diario delle lezioni imparate
```

### Il diario delle lezioni

`correzioni.md` parte con un insieme di lezioni tecniche reali, raggruppate per tema: sottotitoli (buchi tra frasi, larghezza, box), trascrizione (modello giusto, frasi dubbie), inquadratura (volto, 16:9 → 9:16), animazione (GSAP, trasformazioni, cerchi disegnati), ritmo, audio (livelli SFX, musica, master) e processi lunghi.
**È pensato per crescere**: ogni correzione che dai a Claude diventa una riga, e al video successivo la regola è già lì.

## Requisiti

Nessuno obbligatorio: la skill è solo testo. Rende al meglio con:

- [HyperFrames](https://github.com/heygen-com/hyperframes) per costruire e controllare i video (`check`, `snapshot`, `render`)
- `ffmpeg` / `ffprobe` per analisi e master audio
- [`whisper.cpp`](https://github.com/ggml-org/whisper.cpp) (`whisper-cli`) per la trascrizione con tempi per parola

Se non usi HyperFrames, i passi 1-4 e 6 valgono comunque; adatta i comandi del passo 5 al tuo strumento.

## Personalizzare

- Cambia o aggiungi righe al **catalogo stili** in `SKILL.md`.
- Se hai una libreria di effetti o una skill di marca, citala nei passi 0 e 1.
- Svuota o riscrivi `correzioni.md` con le tue lezioni.

## English summary

**qualita-montaggio** is a Claude Code skill (written in Italian) that enforces a quality protocol for any video, reel or motion-graphics task: pick a style reference, write a 5-line brief, storyboard before coding, build with one font/palette/animation logic, then **actually look at the rendered frames** (including dense frame sequences for animations) before delivering, and log every correction in a growing lessons file. It is a method, not an editor, and works best with HyperFrames, ffmpeg and whisper.cpp. Install by copying `skills/qualita-montaggio` into `~/.claude/skills/`.

## Licenza

[MIT](LICENSE)
