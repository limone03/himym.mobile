# Una sera a New York

Prototipo giocabile: 5 personaggi selezionabili, un mondo 3D esplorabile
di giorno (Three.js) e serate al bar con dialoghi a scelta multipla che
cambiano la storia dei giorni successivi.

È un sito **statico**: nessun server, nessuna build, nessuna dipendenza
da installare. Apri `index.html` e funziona.

## Come si gioca

- **WASD / frecce**: muovi il personaggio nel mondo di giorno
- **E** (o il pulsante "Entra" a schermo): entra in un edificio quando sei vicino
- Su mobile compare un pad direzionale in basso a sinistra
- Visita gli edifici di giorno, poi vai da **MacLaren's** la sera:
  lì rispondi alle domande degli amici. Le risposte cambiano cosa
  succede il giorno dopo.

## Pubblicarlo online (GitHub Pages, gratis)

1. Crea un repository nuovo su GitHub (es. `una-sera-a-rivalta`)
2. Carica dentro tutti i file di questa cartella (mantenendo la struttura:
   `index.html`, `style.css`, `js/...`)
3. Vai su **Settings → Pages** del repository
4. In "Source" seleziona il branch `main` e la cartella `/ (root)`
5. Dopo un minuto il gioco sarà online su:
   `https://<tuo-utente>.github.io/una-sera-a-rivalta/`

Da riga di comando, se hai già `git` configurato:

```bash
cd himym-game
git init
git add .
git commit -m "Prima versione del gioco"
git branch -M main
git remote add origin https://github.com/<tuo-utente>/una-sera-a-rivalta.git
git push -u origin main
```

Poi attiva Pages come al punto 3-4.

## Struttura del progetto

```
index.html          punto di ingresso della pagina
style.css            tutto lo stile visivo (palette, tipografia, layout)
js/main.js            motore di gioco: stato, schermate, HUD
js/world3d.js          mondo 3D esplorabile (Three.js), movimento, edifici
js/dialogue.js         motore dei dialoghi ramificati della serata al bar
js/data/characters.js  i 5 personaggi (nome, ruolo, colore, descrizione)
js/data/locations.js   le location nel mondo 3D (posizione, dimensione)
js/data/story.js       TUTTA la trama: vignette di giorno + dialoghi serali
```

## Come modificare / aggiungere contenuti

Tutta la scrittura vive in `js/data/story.js`, separata dal motore di
gioco: puoi riscrivere completamente la trama senza toccare il resto
del codice.

### Rinominare i personaggi

Apri `js/data/characters.js` e cambia `name`, `role`, `bio`, `color`.
Gli `id` (es. `teo`, `barna`) sono usati internamente per collegare i
personaggi alle loro storie in `story.js` — se li cambi, aggiorna anche
le chiavi corrispondenti in `story.js`.

### Aggiungere un nuovo giorno/serata a un personaggio

In `js/data/story.js`, ogni personaggio ha un array `days`. Aggiungi un
nuovo oggetto in fondo all'array seguendo questo schema minimo:

```js
{
  intro: 'Testo mostrato a inizio giornata',
  vignettes: {
    casa:   { text: '...', choices: [ { label: '...', mood: 1, result: '...' } ] },
    lavoro: { text: '...', choices: [ { label: '...', mood: 0, result: '...' } ] },
    parco:  { text: '...', choices: [ { label: '...', mood: 1, result: '...' } ] }
  },
  evening: {
    start: 'q1',
    nodes: {
      q1: {
        speaker: 'Barna',
        text: 'Una domanda al bar...',
        choices: [
          { label: 'Risposta A', next: 'end_a' },
          { label: 'Risposta B', next: 'end_b' }
        ]
      },
      end_a: { end: true, outcome: 'Cosa succede con la risposta A', moodDelta: 1 },
      end_b: { end: true, outcome: 'Cosa succede con la risposta B', moodDelta: -1 }
    }
  }
}
```

Note utili:

- `mood` / `moodDelta`: numero che sale o scende l'umore del personaggio
  (influenza solo la schermata finale, per ora — puoi usarlo per altro)
- `flag` / `setFlag`: imposta un "ricordo" persistente (es. `metSara: true`)
  che puoi poi leggere altrove con `when: 'metSara'` su una scelta, per
  sbloccarla solo se è già successo qualcosa in precedenza
- `text`, `intro` e `result` possono essere anche **funzioni** invece di
  stringhe, per cambiare il testo in base alle scelte fatte prima:
  ```js
  intro: (flags) => flags.metSara ? 'Testo se hai conosciuto Sara' : 'Testo altrimenti'
  ```

### Aggiungere una nuova location nel mondo 3D

Apri `js/data/locations.js` e aggiungi un oggetto con `id`, `label`,
`icon`, `position` (coordinate x/z nel mondo), `size` e `trigger`
(raggio di interazione). Poi aggiungi la stessa chiave `id` dentro
`vignettes` in `story.js` per ogni giorno in cui vuoi che sia visitabile.

### Cambiare l'aspetto del mondo 3D

Tutto il mondo (colori, edifici, alberi, luce) è generato via codice in
`js/world3d.js`, con geometrie di base di Three.js — nessun file 3D
esterno da gestire. Puoi cambiare colori, dimensioni, numero di alberi,
o sostituire le forme con modelli `.glb` caricati con `GLTFLoader` se
in futuro vuoi grafica più elaborata.

## Limiti di questo prototipo (idee per continuare)

- Le "vignette" di giorno sono scelte singole senza rami successivi:
  puoi trasformarle in mini-alberi di dialogo come quelli della sera
- Il "mood" non cambia ancora nulla se non il finale — potresti farlo
  influenzare quali opzioni sono disponibili al bar
- Ogni personaggio ha solo 2 giorni di contenuto: la struttura dati
  regge tranquillamente un numero maggiore di giorni/episodi
- Se vuoi salvataggi tra sessioni, puoi aggiungere `localStorage` per
  ricordare a che punto della storia è arrivato ogni personaggio
