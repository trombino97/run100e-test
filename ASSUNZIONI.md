# Registro delle assunzioni – Tool di test Domyos Run100E

Versione 1 · 27 ottobre 2026

Fonti di verità: «Handoff tecnico» del 25/10/2026 (H) e «Addendum tecnico» del 27/10/2026 (A). Se si contraddicono vale A.

Stato di ogni voce:
- **Confermata**: decisa o confermata da Stefano.
- **Decisa da Claude**: scelta di progetto fatta da Claude su delega («valuta tu»), con la motivazione.
- **Ipotesi**: comportamento non presente nei documenti, preso dallo standard FTMS o da ragionamento; il tool la verifica e segna l'esito.

Ogni voce ha un identificativo (Axx). Il codice e il report citano questi identificativi.

---

## Piattaforma

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A01 | La pagina si pubblica su GitHub Pages (https) e si apre in Chrome per Android. | Confermata | Stefano ha un account GitHub. Web Bluetooth richiede un secure context. |
| A02 | Il file locale e l'artifact di claude.ai non sono usabili per Web Bluetooth. | Ipotesi | `content://` probabilmente non è secure context; l'artifact gira in un iframe isolato. La mini pagina lo mostra se provata. |
| A03 | Dopo una caduta del collegamento Chrome permette `gatt.connect()` sullo stesso dispositivo senza un nuovo `requestDevice`. | Ipotesi | Necessaria per la regola 4. Verificata con il pulsante «Ricollega senza scegliere» della mini pagina. |
| A04 | La lettura del numero di serie (0x2A25) fallisce con SecurityError. | Ipotesi | Caratteristica nella blocklist di Web Bluetooth. Non è un errore di collegamento. |
| A05 | Se la pagina passa in background (schermo spento, cambio app) con il nastro in moto, il tool invia subito Stop finché il codice è ancora in esecuzione. Senza Wake Lock attivo i test non partono. | Decisa da Claude | Chrome può sospendere o rallentare una pagina in background, e il nastro continua da solo (H). Meglio fermarsi che perdere il controllo. |

## Sicurezza

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A10 | Tetto 10,0 km/h come costante. Un valore superiore non viene mai inviato: il controllore lo rifiuta, registra un errore di programma, invia Stop e interrompe il test. Nessuna correzione silenziosa. | Confermata (regola 1); modalità decisa da Claude | Correggere in silenzio nasconderebbe un errore nella sequenza. |
| A11 | Unico valore sotto 1,0 km/h ammesso: 0,5 km/h, solo in T3. | Confermata | Regola 1. |
| A12 | Durata massima di una fase a velocità non nulla: 60 s, poi Stop automatico. Avviso sonoro 3 s prima di ogni cambio di velocità. | Confermata | Risposta 3. |
| A13 | Per «fase» si intende una singola esecuzione continua con il nastro in moto, da Start a Stop. T4 è divisa in tre fasi (una per profilo di rampa), ciascuna con Stop finale e conferma. | Decisa da Claude | Ogni profilo dura circa 40 s e resta sotto 60 s. Dopo ogni profilo il nastro torna fermo (regola 5). |
| A14 | Se un evento `05` o i dati riportano più di 10,0 km/h, il tool invia Stop. | Confermata | Risposta 4. La console può superare il tetto. |
| A15 | All'evento `03` (chiave tolta) il tool invia **subito** Stop (`08 01`), in qualsiasi test, T8 compreso. | Decisa da Claude | Senza Stop, uno Start successivo fa ripartire il nastro alla velocità di prima appena la chiave è rimessa (A, §6). Con lo Stop questa situazione non può più crearsi. L'osservazione «chiave rimessa senza comandi: nessun evento» è già stata fatta due volte il 27/10 (A, §6), quindi non si perde informazione. Il report segnala che in T8 la condizione differisce da quella del 27/10. |
| A16 | Risposta attesa allo Stop a chiave tolta: `80 08 01`. | Ipotesi | A dice che a chiave tolta i comandi sono accettati con successo e che lo Stop a chiave tolta ha effetto, ma non riporta i byte della risposta. |
| A17 | Risposta mancante: nessuna indicazione entro 2 s dalla scrittura. Conseguenza: Stop automatico. Se manca anche la risposta allo Stop, il tool lo reinvia una volta e poi mostra a tutto schermo «Ferma il nastro con la console o con la chiave». | Confermata (2 s); gestione dello Stop senza risposta decisa da Claude | Regola 4. |
| A18 | Evento imprevisto = un evento non presente nella lista degli eventi attesi della fase in corso. Ogni fase dichiara la sua lista. In T7 gli eventi `05` e `02 01` esterni sono attesi; in T8 lo è `03`. Un evento imprevisto provoca Stop. | Decisa da Claude | Rende applicabile la regola 4 nei test in cui la console o la chiave generano eventi di proposito. |
| A19 | Dopo una perdita di collegamento: allarme sonoro e vibrazione, tentativi di ricollegamento, riattivazione di tutte le sottoscrizioni, poi Stop. Lo Stop si invia prima del Request Control. Se è rifiutato con `80 08 05` (controllo non consentito), il tool invia Request Control e di nuovo Stop. | Decisa da Claude | H: Stop accettato subito dopo una riconnessione senza nuovo Request Control. H: le sottoscrizioni si azzerano a ogni connessione. |
| A20 | La conferma «il nastro è fermo» prima del test successivo la dà Stefano con un pulsante. Il tool non usa Training Status per decidere che il nastro è fermo e considera la velocità nei dati solo un indizio. | Decisa da Claude | A: la velocità nei dati è il valore impostato, Training Status non è affidabile. |
| A21 | T2 invia di proposito una velocità (4,0 km/h) a nastro fermo prima dello Start, unica eccezione alla regola «mai velocità prima dello Start». | Confermata (richiesta in T2) | A, osservazione a voce: velocità ignorata. Nastro vuoto e valore sotto il tetto. |

## Protocollo e correlazione

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A30 | Regola di correlazione: un evento è «causato dal tool» se arriva entro 1,5 s da un comando del tool dello stesso tipo: `04` ↔ `07`; `05` ↔ `02` **con lo stesso valore di velocità**; `02 01` ↔ `08 01`; `02 02` ↔ `08 02`. Altrimenti è «esterno». | Decisa da Claude | A suggerisce la finestra di 1,5 s. Il confronto del valore serve nelle rampe a 1 comando al secondo, dove gli eventi di due comandi successivi cadono nella stessa finestra. |
| A31 | Dopo Pausa (`08 02`) l'evento atteso è `02 02`. Esiti: `02 02` → OK; `02 01` → diverso dall'atteso (non provoca Stop); nessun evento entro 2 s → non conclusivo. | Ipotesi (standard FTMS) | Né H né A riportano l'evento della Pausa. La risposta `80 08 01` alla Pausa è invece verificata (H). |
| A32 | Dopo la ripresa da Pausa il nastro riparte da 1,0 km/h. | Atteso da H, non riprovato in A | H, 25/10. |
| A33 | Effetto del Reset (`01`) sconosciuto. Si invia solo a nastro fermo e confermato, come ultimo test. Il tool registra risposta, eventi e contatori senza giudicare OK o diverso. Dopo il Reset il tool considera perso il controllo e invia di nuovo Request Control prima di qualsiasi altro comando. | Ipotesi (standard FTMS) | H e A si contraddicono (H «provato», A «non verificato»): vale A. Lo standard FTMS prevede che il Reset revochi il controllo. |
| A34 | La decodifica di Treadmill Data segue i flag dello standard FTMS. Solo il layout con flag `A4 01` è stato osservato. Un pacchetto con flag diversi viene decodificato secondo lo standard e segnalato come anomalia nel report. | Ipotesi (standard FTMS) | H, A. |
| A35 | Il «tempo di risposta» si misura dalla risoluzione della scrittura (`writeValue` completato) alla ricezione dell'indicazione. | Decisa da Claude | nRF Connect non specifica come misurava i 48–88 ms di A. Il confronto è quindi indicativo. |

## Test

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A40 | T1: ogni ripetizione è Request Control → Start → 2,0 km/h → Stop, con attesa dei dati a 0 e conferma prima della ripetizione seguente. Tre ripetizioni. | Decisa da Claude | Richiesta T1. |
| A41 | T3 si esegue a 2,5 km/h: invio 0,5 km/h (atteso `80 02 03`), poi 2,0 km/h per verificare che il controllo sia rimasto. | Decisa da Claude | Ripete la prova di H. |
| A42 | T4: il tool porta il nastro a 6,0 km/h con un solo comando dopo lo Start, attende che i dati mostrino 6,0, poi esegue la rampa fino a 10,0 e ritorno a 6,0, e chiude la fase con Stop. | Decisa da Claude | A: un salto 1,0 → 8,0 è già stato fatto senza problemi. |
| A43 | T5: il pulsante «Il nastro è a velocità» misura il tempo dal comando al tuo tocco. Il risultato è un'osservazione a voce, non un dato. | Confermata | Richiesta T5. |

---

## Storico delle modifiche

- v1, 27/10/2026: prima stesura, dopo le risposte di Stefano alle cinque domande.
