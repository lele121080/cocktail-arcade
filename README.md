# Cocktail Arcade

App web personale per imparare a fare i cocktail da zero, con grafica retro anni 80 da cabinato arcade.
Nata come strumento di allenamento domestico (per due persone), con l'idea di arrivare un domani a lavorare dietro a un bancone.

**Online:** https://lele121080.github.io/cocktail-arcade/

---

## 1. Parte funzionale

### Obiettivo
Imparare le basi della miscelazione senza spendere soldi inutilmente: comprare l'attrezzatura e le bottiglie un po' alla volta, partire da pochi cocktail, e memorizzare le dosi con un metodo invece che a forza.

### Il metodo (approvato)
Invece di imparare ricette una per una, si imparano **3 famiglie con uno schema fisso**:

| Famiglia | Tecnica | Schema |
|---|---|---|
| Old Fashioned | build (nel bicchiere) | spirito + dolcificante + bitter + ghiaccio |
| Sour | shake | spirito : acido : dolce = **2 : 0,75 : 0,75** (6 cl : 2,5 cl : 2,5 cl) |
| Martini / stirred | stir (nel mixing glass) | spirito + vermouth, mai shakerato |

Regola extra: il Negroni è l'unico a parti uguali (1:1:1).

### I cocktail inclusi (dosi per singolo drink)

| Cocktail | Ingredienti | Tecnica | Bicchiere |
|---|---|---|---|
| Old Fashioned | bourbon 6 cl, sciroppo 1 cl, Angostura 2 dash, twist d'arancia | build + stir 20 s | Tumbler basso |
| Negroni | gin 3, vermouth rosso 3, Campari 3 cl, twist d'arancia | build + stir 15 s | Tumbler basso |
| Whiskey Sour | bourbon 6, limone 2,5, sciroppo 2 cl | shake 12 s, filtrato | Coupe |
| Daiquiri | rum bianco 6, lime 2,5, sciroppo 2 cl | shake 12 s, filtrato | Coupe |
| Margarita | tequila 5, triple sec 2, lime 2,5 cl, bordo di sale | shake 12 s | Tumbler |
| Moscow Mule | vodka 5, lime 1,5, ginger beer 10 cl | build + stir 5 s | Highball / mug |
| Mojito | rum bianco 6, lime 2, sciroppo 2 cl, 9 foglie di menta, soda 6 cl | muddle + build | Highball |
| Manhattan | bourbon 6, vermouth rosso 3 cl, Angostura 2 dash | stir 20 s, filtrato | Coupe |

### Sezioni dell'app

- **Schermata iniziale**: "Inserisci coin", con suono di moneta da cabinato (sintetizzato, nessun file audio).
- **HUD in alto**: `PERSONE` è il contatore ospiti globale (scala le dosi nella preparazione); `PRONTI` indica quanti cocktail su 8 si possono fare con quello che si ha in casa.
- **SCORTE**: checklist di attrezzatura, bicchieri, alcolici e altri ingredienti. Le spunte restano salvate.
- **COCKTAIL**: lista degli 8 cocktail. Il badge è `PLAY` se si ha tutto, oppure `-N` con il numero di cose mancanti. Si può aprire comunque qualunque cocktail.
- **Preparazione (stile videogioco)**, passo per passo (STAGE 1/N):
  - bicchiere consigliato sempre visibile (nome + forma);
  - il contenitore cambia in base alla tecnica: **shaker** (si scuote da solo durante lo shake), **mixing glass**, poi il bicchiere di servizio dopo il filtraggio;
  - ogni ingrediente ha il suo colore reale e un'altezza proporzionale alla dose, con legenda a fianco;
  - bottigliette colorate che si capovolgono e versano; paletta che fa cadere il ghiaccio (cubo grande, cubetti, o tritato a seconda del cocktail);
  - timer con barra per shake e stir, anello che gira durante la mescolata;
  - tasto **RICOMINCIA** in ogni momento.
- **BASI**: le tecniche fondamentali (sciroppo, ghiaccio, twist, succo fresco, bordo salato) e **Calcola la serata**: per ogni cocktail scelto si imposta il numero di persone e quanti drink a testa, e l'app somma le quantità totali (con conversione in lime/limoni interi e in ml di sciroppo da preparare).
- **MUSICA**: login Spotify e fascia fissa in basso con il brano in riproduzione in quel momento; scorciatoia verso una ricerca di musica retro/videogame e possibilità di salvare il link di una propria playlist.

### Decisioni prese

- Si parte da 8 cocktail e si impara bene quelli prima di aggiungerne altri.
- Spotify in **sola lettura**: la musica si fa partire dall'app Spotify (o da qualunque dispositivo) e l'app mostra cosa sta suonando. Il player integrato (Web Playback SDK) è stato valutato e **scartato**: più fragile (si ferma se la pagina va in background) e ridondante.
- Non è un prodotto commerciale per ora: resta uno strumento personale (vedi limiti sotto).

### Limiti noti

- **Spotify in Development mode**: massimo 5 utenti di test, aggiunti a mano nel pannello sviluppatore. Per aprirlo a molti utenti servirebbe la Extended Quota di Spotify, che richiede un'azienda e una base utenti molto ampia.
- Il refresh token Spotify dura circa 180 giorni: dopo mesi di inattività basta premere di nuovo "Accedi con Spotify".
- Le quantità di succo per frutto sono stime (1 lime ≈ 2,5 cl, 1 limone ≈ 3 cl).
- Le proporzioni nel bicchiere sono uno **schema didattico**, non una simulazione fisica.

---

## 2. Parte tecnica

### Architettura
Un unico file `index.html` (HTML + CSS + JavaScript vanilla), senza build, senza dipendenze, senza backend. Ospitato su GitHub Pages (repository pubblico, HTTPS obbligatorio).
Risorse esterne: solo i font Google (`Press Start 2P`, `VT323`) e, per Spotify, `accounts.spotify.com` e `api.spotify.com`.

### Stato e persistenza
Stato in un oggetto `state`; il rendering è una funzione `render()` che ricostruisce l'HTML della vista corrente, con event delegation su `#app` (`data-action`).
Persistenza in `localStorage`, chiave `cocktailArcadeData`: inventario, numero persone, link playlist, Client ID e token Spotify.

### Modello dati
- `INVENTORY_CATS`: categorie e voci dell'inventario (ogni voce ha una chiave `inv`).
- `COCKTAILS[]`: `id`, `name`, `glassShape`, `tools`, `extraTools`, `ingredients[]` (`inv`, `amount`, `unit` tra `cl`/`dash`/`pz`/`foglie`), `steps[]`.
- Ogni step: `tag`, `vessel` (`glass`/`shaker`/`mixing`), `reveal[]` (ingredienti che compaiono in quello step), `ice` (`none`/`big`/`cubes`/`crushed`), `duration` (secondi, opzionale), `text` (con segnaposto `{inv}`).
- `COLORS`: colore di ogni ingrediente. `GLASS_INFO`: nome ed etichetta di ogni forma di bicchiere.

### Logica principale
- `scaleAmount()` moltiplica la dose per persone (e drink a testa nel calcolatore serata).
- `fillTemplate()` sostituisce `{inv}` con **solo il numero**: l'unità di misura è scritta nel testo dello step (evita duplicati tipo "4dash dash").
- `buildLayers()` costruisce gli strati colorati. Il totale è calcolato sull'**intera ricetta finale**, non solo su quanto versato finora, altrimenti un ingrediente piccolo sembra riempire il bicchiere. Un dash vale 0,6 "cl visivi" (`NOMINAL_DASH_CL`) solo per il disegno.
- Animazioni solo CSS: `vesselShake`, `stirSpin`, `scoopMove`, `iceDrop`, `pourTilt`, `streamFlow`. Il timer aggiorna soltanto la barra (non ridisegna tutto) per non riavviare le animazioni a ogni tick.
- Suono moneta: Web Audio API (due onde quadre), parte al tocco.

### Integrazione Spotify (OAuth 2.0 Authorization Code + PKCE, senza server)
1. L'utente salva il **Client ID** nella sezione MUSICA (il Client ID non è un segreto; il Client Secret **non** viene mai usato).
2. `spotifyLogin()` genera `code_verifier` e `code_challenge` (SHA-256), poi reindirizza ad `accounts.spotify.com/authorize` con scope `user-read-currently-playing`.
3. Al ritorno `spotifyHandleRedirect()` scambia il `code` con access e refresh token e ripulisce l'URL.
4. `fetchNowPlaying()` interroga `GET /v1/me/player/currently-playing` ogni 8 secondi; `spotifyRefreshToken()` rinnova il token prima della scadenza o su risposta 401.
5. La fascia `#now-playing-bar` è fissa in basso e visibile in tutte le schermate.

Il **Redirect URI** registrato su Spotify deve coincidere esattamente (carattere per carattere) con `origin + pathname` della pagina: `https://lele121080.github.io/cocktail-arcade/`.

### Setup da zero
1. Creare un repository **pubblico** e caricare `index.html` (e questo README) nella root.
2. Settings → Pages → Deploy from a branch → `main` / root.
3. Su developer.spotify.com creare un'app (API: Web API) con il Redirect URI qui sopra. Dal 2026 serve un account Premium per creare l'app.
4. Aprire il sito, andare in MUSICA, incollare il Client ID, salvare e premere "Accedi con Spotify".
5. Per far entrare altre persone: aggiungerle in User Management dell'app Spotify (max 5).

### Manutenzione e sicurezza
- Nessuna dipendenza da aggiornare (niente `package.json`, niente librerie, niente backend).
- Gli endpoint Spotify usati (login/token e `currently-playing`) non sono toccati dalle modifiche del febbraio 2026 (cambi su playlist, ricerca, creazione playlist).
- Token Spotify salvati in `localStorage` (prassi standard per un client pubblico PKCE).
- Non committare mai Client Secret o token nel repository.

### Due versioni dello stesso progetto
- **Questo repository**: versione con Spotify reale.
- Una versione semplificata ospitata dentro Claude (senza login Spotify, perché l'ambiente protetto di Claude blocca le chiamate verso Spotify). Condivide dati e grafica; la sezione MUSICA si limita ad aprire Spotify.

---

## 3. Storico delle modifiche

1. Corso "da zero" (documento) con attrezzatura, famiglie di cocktail, mise en place, piano di allenamento.
2. Prima app: inventario, lista cocktail, preparazione passo per passo, calcolo serata.
3. Bicchiere con strati colorati per ingrediente, righe di livello, ghiaccio grafico.
4. Legenda in elenco (etichette sovrapposte risolte), badge `-N` al posto di `LOCK`, persone e quantità per singolo cocktail nel calcolo serata.
5. Suono moneta e sezione MUSICA; poi pubblicazione su GitHub Pages con login Spotify e "ora in riproduzione".
6. Correzione testi (unità duplicate), proporzioni calcolate sulla ricetta finale, tasto RICOMINCIA.
7. Preparazione animata: shaker/mixing glass, anello di mescolata, paletta del ghiaccio, bottiglie che versano, bicchiere consigliato sempre visibile.
8. Bottiglie ridisegnate: si capovolgono con il rivolo dal collo, e c'è più spazio sopra il bicchiere per non sovrapporsi al testo.

## 4. Idee future (non approvate, solo appuntate)

- Filtro "cosa posso fare con quello che ho" (i dati ci sono già).
- Più cocktail, suddivisi per spirito.
- Player Spotify integrato: scartato per ora.
- Trasformazione in prodotto: da rivalutare più avanti; richiederebbe una riscrittura e di ripensare la parte Spotify.
