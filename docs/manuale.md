# Manuale Tecnico e Utente - Guerra Stellare

## 1. Panoramica
Guerra Stellare e un gioco arcade 2D in puro HTML/CSS/JavaScript, tutto contenuto in un solo file: `index.html`.
Non usa framework, build tool o package manager: apre un canvas, carica asset locali (immagini/audio) e gestisce il game loop via `requestAnimationFrame`.

## 2. Requisiti e avvio
### Requisiti
- Browser moderno desktop o mobile
- Supporto Canvas 2D
- Supporto Web Audio API (facoltativo ma consigliato)
- Accesso ai file locali del progetto

### Avvio rapido
1. Aprire `index.html` nel browser.
2. Attendere il menu iniziale.
3. Premere `Invio` per entrare nel briefing e poi avviare la partita.

Nota: il progetto e pensato per funzionare anche in modalita file locale (`file:///...`).

## 3. Struttura del progetto
- `index.html`: unico file con layout, stili, logica gioco, rendering, audio e input.
- `docs/manuale.md`: questo manuale.
- `falcon_b64.txt`: risorsa testuale non usata dal runtime corrente.
- Asset grafici/audio nella root:
  - `falcon_millenian.png` (sprite player)
  - `tie_fighter.png` (sprite nemici)
  - `caza_dark_vater.png` (sprite mini-boss Vader)
  - `star-wars.mp3` (traccia intro)
  - `star-wars-battlie.mp3` (musica battaglia)
  - `star-wars-battlie2.mp3` (SFX sparo Falcon)
  - altri mp3 presenti ma non agganciati alla logica attuale

## 4. Controlli
### Tastiera
- `ArrowLeft`, `ArrowRight`, `ArrowUp`, `ArrowDown`: movimento
- `Space`: fuoco
- `P`: pausa/riprendi
- `M`: mute/unmute
- `Esc`: ritorno al menu (da gioco/pausa/intro)
- `Invio`: start, skip briefing, restart dopo game over

### Touch (mobile/coarse pointer)
- D-pad virtuale a schermo
- Pulsante `FUOCO`
- Pulsante `PAUSA`

## 5. Flusso schermate e stati
La macchina a stati usa `GS.mode` con questi valori:
- `menu`: schermata titolo e istruzioni
- `intro`: crawl testuale prima della partita
- `play`: partita in corso
- `pause`: gioco congelato con overlay
- `over`: game over con punteggio finale
- `victory`: schermata vittoria dopo distruzione punto debole
- `cinematic`: sequenza di fuga/esplosioni prima della vittoria finale

Transizioni principali:
- `menu -> intro` con `Invio`
- `intro -> play` a fine timer o con `Invio`
- `play <-> pause` con `P`
- `play/pause/intro -> menu` con `Esc`
- `over -> play` con `Invio`
- `play (fase spazio) -> play (fase trincea)` automaticamente dopo pulizia totale nemici
- `play (fase trincea) -> cinematic` quando il punto debole viene colpito definitivamente
- `cinematic -> victory` al termine della fuga del Falcon

## 6. Entita principali
- Player (Millennium Falcon): posizione, velocita, invulnerabilita, scudo, cooldown sparo.
- Nemici in formazione: tipi `basic`, `mid`, `boss`, piu un `vader` speciale in testa.
- Nave nodriza: spawn periodico, HP elevati, premio importante alla distruzione.
- Proiettili player/nemici.
- Asteroidi dinamici.
- Esplosioni particellari.
- Campo stelle di sfondo.

## 7. Regole gameplay
- Le ondate sono composte da 30 nemici (`WAVE_SIZE = 30`) con difficolta crescente.
- Sequenza campagna attuale:
  - Livello 1: battaglia spaziale classica (formazioni + minaccia nodriza)
  - Livello 2: Death Star Run nella trincea con spawn nemici da lati/alto
- In livello 2 il Falcon resta nella corsia della trincea e deve avanzare fino al punto debole.
  - A fine avanzamento compare il punto debole (`exhaust port`) da colpire piu volte.
  - Il punto debole alterna finestre `aperto/chiuso`: i colpi entrano solo quando e aperto.
  - Nemici livello 2 differenziati:
    - `interceptor`: piu rapidi e con traiettorie nervose
    - `turret`: fuoco a ventaglio
    - `bomber`: piu lenti ma resistenti e aggressivi
- Distrutto il punto debole: vittoria missione e salvataggio record.
  - Dopo il colpo finale parte una sequenza cinematica di fuga con esplosioni progressive della struttura.
- Distruggere nemici incrementa punteggio e combo (max x5).
- Se il player viene colpito: perde una vita, reset combo, breve invulnerabilita.
- Distruzione nave nodriza:
  - assegna punti bonus
  - concede 1 vita extra (cap vite a 6)
  - attiva scudo temporaneo (`SHIELD_DURATION = 10s`)
- Se le vite arrivano a 0: `game over`.

## 8. HUD e feedback
HUD mostra:
- Punteggio corrente
- Record (localStorage)
- Onda/kill (fase spazio) oppure livello/progresso trincea (fase Death Star Run)
- Combo e barra tempo combo
- Scudo attivo e timer residuo
- Vite (icone falcon)
- Stato audio (`[M]`)
- Hint pausa (`[P]`)

Messaggi di stato temporanei (`GS.message`) segnalano eventi chiave: combo alte, ondata, allerta nodriza, distruzioni importanti.

## 9. Audio
Pipeline audio mista:
- HTMLAudioElement per musica:
  - intro: `star-wars.mp3`
  - battaglia: `star-wars-battlie.mp3`
- HTMLAudioElement per SFX sparo Falcon:
  - `star-wars-battlie2.mp3` (con cooldown anti-sovrapposizione)
- Web Audio API per effetti sonori sintetici (`tone`, `noise`, oggetto `SFX`)

Comportamento:
- `ensureAudio()` crea `AudioContext` e master gain on demand.
- `M` toggla `muted` e sincronizza gain + mute delle tracce.
- Intro usa fallback synth se il browser blocca autoplay della traccia.
- Negli ultimi secondi del briefing intro la traccia `star-wars.mp3` applica un crescendo progressivo prima della transizione al gioco.
- Negli ultimissimi 2 secondi l intro applica anche un boost piu aggressivo del volume per un finale piu epico.

## 10. Rendering e performance
- Canvas fisso: `820x660`, ridimensionamento visuale via `syncViewport()`.
- Rendering separato in:
  - sfondo e stelle
  - scena di gioco (entita)
  - HUD/overlay
- Effetti principali:
  - shake camera
  - glow motori/laser/esplosioni
  - sprite fallback vettoriali se immagini non caricate

## 11. Persistenza
Record salvato con chiave localStorage:
- `guerra-stellare-best`

Funzioni dedicate:
- `loadBest()`
- `saveBest(value)`

Entrambe gestiscono errori in modo safe (es. limiti modalita file locale).

## 12. Costanti importanti
- `W = 820`, `H = 660`
- `WAVE_SIZE = 30`
- `ENEMY_SIZE_FACTOR = 0.55`
- `SHIELD_DURATION = 10`
- `INTRO_DURATION = 26`
- `TRENCH_GOAL = 3600`

## 13. Funzioni chiave (mappa rapida)
- Setup/responsive: `syncViewport()`
- Audio: `ensureAudio()`, `startIntroMusic()`, `startBattleMusic()`, `stopIntroMusic()`, `stopBattleMusic()`
- Stato gioco: `init()`, `startIntro()`, `startGame()`, `spawnWave()`, `update(dt)`, `hitPlayer()`
- Stato livello 2: `startDeathStarRun()`, `spawnTrenchEnemy()`, `updateTrenchRun(dt)`
- Cinematica finale: `startDeathStarEscapeCinematic()`, `updateCinematic(dt)`
- Rendering: `render()`, `renderScene()`, `drawTrenchScene()`, `drawHUD()`, `drawMenu()`, `drawIntroCrawl()`, `drawOverScreen()`, `drawVictoryScreen()`
- Main loop: `loop(ts)`

## 14. Manutenzione e modifiche consigliate
Per estendere il progetto senza rompere il flusso:
1. Aggiungere nuove meccaniche dentro `update(dt)` in blocchi separati.
2. Tenere il rendering puro (niente logica di stato pesante dentro draw).
3. Riutilizzare `setMessage()` e `bumpShake()` per feedback coerente.
4. Aggiornare HUD quando si aggiungono nuove risorse (es. energia, power-up).
5. Se aggiungi asset, mantieni nomi consistenti e preload esplicito.

## 15. Troubleshooting rapido
- Schermo fermo o nero:
  - controllare errori JavaScript in console browser.
  - verificare presenza di tutti gli asset nella root.
- Nessun audio:
  - premere un tasto (sblocca contesto audio su alcuni browser).
  - verificare `M` non attivo su muto.
- Touch non visibile:
  - UI touch compare su device coarse pointer o viewport piccola.

## 16. Possibili evoluzioni
- File JS separato per migliorare leggibilita.
- Loader asset con schermata caricamento.
- Sistema livelli/upgrade persistenti.
- Bilanciamento difficolta via parametri centralizzati.
- Test automatici su funzioni utility e collisioni.
