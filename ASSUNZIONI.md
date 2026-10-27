# Registro delle assunzioni – Banco prova Run100E

Versione 3 · 27 ottobre 2026 · vale per il tool v1.1.0 (`run100e-banco-prova.html`)

Fonti di verità:
- «Handoff tecnico» del 25/10/2026 (H);
- «Addendum tecnico» del 27/10/2026 (A), che prevale su H in caso di contrasto;
- registro della mini pagina del 27/10/2026 alle 16:36, inviato da Stefano (M);
- registro e report del banco prova v1.0.0 del 27/10/2026, 18:05–18:14 (R).

Ogni voce ha uno di tre stati:
- **Confermata**: decisa o confermata da Stefano, oppure verificata su un registro reale.
- **Decisa da Claude**: scelta fatta da Claude su delega («valuta tu»), con la motivazione.
- **Ipotesi**: comportamento assente dai documenti, preso dallo standard FTMS o da un ragionamento. Il tool la mette alla prova e ne registra l'esito.

Il codice e il report citano gli identificativi Axx.

---

## Piattaforma

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A01 | La pagina si pubblica su GitHub Pages (https) e si apre in Chrome per Android. | Confermata | Stefano ha un account GitHub. M: secure context = sì. |
| A02 | Il file locale e l'artifact di claude.ai non sono usabili per Web Bluetooth. | Ipotesi | Non provato. GitHub Pages invece funziona (M). |
| A03 | Dopo una caduta, Chrome si ricollega allo stesso dispositivo senza un nuovo `requestDevice`. | **Confermata** | M: ricollegato in 15 ms. Una disconnessione volontaria non equivale a una perdita di segnale: il caso del segnale perso resta non provato. |
| A04 | Il numero di serie (0x2A25) non si legge. | **Confermata, con correzione** | M: la caratteristica non esiste. Mancano anche 0x2A27 (revisione hardware) e la blocklist non c'entra. Il tool non le legge. |
| A05 | Se la pagina va in background (schermo spento, cambio di app) il tool invia subito Stop. Senza Wake Lock attivo i test non partono. | Decisa da Claude | Chrome può sospendere una pagina in background e il nastro continua da solo (H). Lo Stop parte solo se il codice gira ancora in quel momento: vedi i rischi residui. |
| A06 | Gli attesi di Device Information sono i valori letti da M: produttore «EW. Inc», modello «Domyos FTMS», firmware «V1.5 21090702», software «1.0.0», System ID `7A C2 0E 00 00 19 9E 68`. | Confermata (dato reale) | M. |
| A07 | Il tapis roulant può annunciarsi con «DOMYOS» o con «Domyos». Il filtro accetta entrambi. | **Confermata** | H: «DOMYOS-TC-7417». M: «Domyos-TC-7417». |

## Sicurezza

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A10 | Il tetto è 10,0 km/h, come costante congelata. Un valore oltre il tetto non viene mai scritto: il tool non lo corregge in silenzio, registra un errore, invia Stop e interrompe il test. Il controllo avviene in due punti del codice; il simulatore fa un terzo controllo. | Confermata (regola 1); modalità decisa da Claude | Una correzione silenziosa nasconderebbe un errore di sequenza. |
| A11 | L'unico valore sotto 1,0 km/h ammesso è 0,5 km/h, e solo mentre è in corso T3. | Confermata | Regola 1. Il vincolo «solo in T3» è nel controllore. |
| A12 | Una fase a velocità non nulla dura al massimo 60 s, poi scatta lo Stop automatico. Prima di ogni avvio e di ogni cambio di velocità c'è un avviso: conto alla rovescia di 3 s con bip. | Confermata | Risposta 3. |
| A12b | Nelle rampe T4 l'avviso di 3 s c'è solo prima della rampa; poi un bip breve per ogni comando. Prima di Stop e Pausa c'è un bip senza attesa: fermarsi non va mai ritardato. | Decisa da Claude | Con un comando al secondo non c'è spazio per 3 s di avviso. |
| A13 | «Fase» = intervallo continuo con il nastro in moto, da Start o ripresa fino a Stop, Pausa o chiave. T4 ha tre fasi, una per profilo, ognuna chiusa da Stop e conferma. | Decisa da Claude | Il caso peggiore stimato è di circa 50 s per profilo. |
| A14 | Se un evento `05` o i dati riportano più di 10,0 km/h, il tool invia Stop. | Confermata | Risposta 4. |
| A15 | All'evento `03` il tool invia subito Stop, in ogni test. Se l'evento non era atteso (fuori da T8), il test si interrompe. | Decisa da Claude | Senza Stop, uno Start successivo fa ripartire il nastro alla velocità di prima (A §6). Il caso «chiave rimessa senza Stop» è già stato osservato due volte il 27/10. |
| A16 | Risposta attesa allo Stop dato a chiave tolta: `80 08 01`. | Ipotesi | A non riporta i byte. |
| A17 | Risposta mancante = nessuna indicazione entro 2 s; la conseguenza è lo Stop automatico. Se anche lo Stop resta senza risposta, il tool lo reinvia una volta e poi mostra l'allarme a tutto schermo «Ferma il nastro a mano». | Confermata (2 s); gestione decisa da Claude | Regola 4. |
| A18 | Evento imprevisto = evento esterno non dichiarato atteso nella fase in corso, oppure codice sconosciuto. Provoca Stop e interruzione del test. Gli eventi esterni attesi sono: T7 → `05` e `02 01`; T8 → `03`; T9 dopo il Reset → `01`, `FF`, `02 01`, `02 02`. | Decisa da Claude | Rende applicabile la regola 4 anche nei test in cui la console o la chiave generano eventi di proposito. |
| A19 | Dopo una perdita di collegamento: allarme sonoro e vibrazione, test interrotto, fino a 30 tentativi di ricollegamento (uno al secondo). Al ricollegamento: (1) Stop subito, prima di riattivare le indicazioni, quindi senza risposta visibile; (2) riattivazione di tutte le sottoscrizioni; (3) secondo Stop con risposta verificata. Se la risposta è `80 08 05`, il tool invia Request Control e poi di nuovo Stop. | Decisa da Claude | H: Stop accettato subito dopo una riconnessione; sottoscrizioni azzerate a ogni connessione. |
| A54 | Ricollegamento: 30 tentativi, uno al secondo. Se falliscono tutti: allarme «Ferma il nastro a mano» e nessun test finché non confermi che il nastro è fermo. | Decisa da Claude | H: supervisione 5 s; il nastro va avanti senza limiti di tempo. |
| A20 | La conferma «il nastro è fermo» la dai tu con un pulsante. Training Status e velocità nei dati non bastano a dirlo. | Decisa da Claude | A §2: la velocità nei dati è il valore impostato e Training Status non è affidabile. |
| A21 | T2 invia di proposito 4,0 km/h a nastro fermo: è l'unica velocità inviata prima di uno Start. | Confermata | Richiesta T2. |

## Protocollo e correlazione

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A30 | Correlazione: un evento è «causato dal tool» se arriva entro 1,5 s da un comando dello stesso tipo. Abbinamenti: `04`↔`07`; `05`↔`02` con lo stesso valore; `02 01`↔`08 01` **oppure `08 02`** (A61); `02 02`↔`08 02`; `01`↔`01`. Ogni comando spiega al massimo un evento. In tutti gli altri casi l'evento è «esterno». `03` è sempre esterno. | Decisa da Claude | A §8 suggerisce la finestra. Il confronto del valore serve nelle rampe veloci. |
| A31 | Dopo la Pausa il Run100E invia `02 01`, non il `02 02` dello standard. Esito in T6: `02 01` causato dal tool → OK; `02 02` → diverso dall'atteso; nessun evento → non conclusivo. | **Osservata** (2 volte) | R: 18:11:10.193 (+1004 ms dalla Pausa) e 18:13:18.191 (+1001 ms). In v2 era un'ipotesi presa dallo standard. |
| A32 | Dopo la ripresa da Pausa il nastro riparte da 1,0 km/h. | Atteso da H | H, non riprovato in A. |
| A33 | L'effetto del Reset (`01`) è sconosciuto. Il tool lo invia solo in T9, a nastro fermo e dopo la tua conferma. Registra senza dare un giudizio, poi invia di nuovo Request Control. | Ipotesi | H e A si contraddicono: vale A («non verificato»). |
| A34 | Treadmill Data si decodifica leggendo i flag, con l'ordine dei campi dello standard FTMS. Flag diversi da `A4 01` o una lunghezza diversa da 14 byte → anomalia nel report, senza Stop. | Ipotesi (standard FTMS) | H, A. |
| A35 | Tempo di risposta = dalla chiamata di scrittura all'arrivo dell'indicazione. | Decisa da Claude | Non si sa come misurava nRF Connect: il confronto con 48–88 ms è indicativo. |
| A36 | Un `05` esterno fino a 2,0 km/h entro 3 s da uno Start è tollerato e marcato nel registro. | Ipotesi | Non si sa se lo Start produca un `05`. H: il nastro parte a 1,0 o a 2,0 km/h. Senza tolleranza ogni Start potrebbe causare uno Stop. |
| A37 | Tutti i comandi tranne lo Stop partono solo da un test in corso. Lo Stop è ammesso sempre. | Decisa da Claude | Nessuna azione implicita (H, sicurezza). |
| A38 | Le scritture sono possibili solo su 0x2AD9 e solo con i comandi `00`, `01`, `02 xx xx`, `07`, `08 01`, `08 02`. I servizi proprietari non sono in `optionalServices`, quindi Chrome non li espone. | Decisa da Claude | Regola «mai scrivere sulle caratteristiche sconosciute». |
| A39 | Un solo comando in attesa di risposta alla volta. Lo Stop scavalca un comando in attesa; due Stop non partono mai insieme. Se Chrome risponde «GATT operation already in progress», lo Stop viene ritentato ogni 100 ms per 1,5 s. | Decisa da Claude | Limite noto di Web Bluetooth. |
| A50 | T0 invia Request Control (`00`) per provare che le indicazioni arrivano davvero. | Decisa da Claude | Il comando non muove il nastro; è l'unico modo di confermare le indicazioni. |
| A52 | Dopo lo Stop dato a chiave tolta non si attende alcun `02 01`. | Confermata | A §4: «Non compare se il nastro era già fermo dalla chiave». |
| A61 | `02 01` arrivato entro 1,5 s da una Pausa del tool = causato dal tool. Fuori da questa finestra, `02 01` senza Stop o Pausa del tool resta un evento esterno e provoca Stop (Stop dalla console). | Decisa da Claude, su dati R | In v1.0 due Pause su due sono state prese per uno Stop dalla console. Il tool ha fermato il nastro (comportamento sicuro) e interrotto T6 e T9. |
| A62 | Training Status arriva come notifica circa una volta al secondo, anche se il valore non cambia. Dopo lo Start resta `01 01` per circa 1 s, poi passa a `01 0D`. | Osservata | R. Non usata per decisioni (A20). |
| A63 | Eventi e dati sembrano agganciati a un ciclo del tapis roulant di circa 1 s. Ritardi osservati: `04` 603–1094 ms; `05` 789–1087 ms (A: 0,7–0,8 s); `02 01` dopo Stop 674–1091 ms. Il margine sulla finestra di 1,5 s è di circa 0,4 s. | Osservata | R. La finestra resta 1,5 s; se un evento arriva oltre, il tool ferma il nastro (sicuro, ma interrompe il test). |
| A64 | Body Sensor Location = `02` (Polso). | Osservata | R, T0. Nessun atteso nei documenti. |
| A65 | Una velocità inviata a nastro fermo riceve `80 02 01`, non genera eventi e non cambia i dati (0,0 km/h); lo Start successivo parte da 1,0 km/h. | **Osservata** | R, T2 parte A. Conferma con i dati l'osservazione a voce dell'Addendum. |
| A66 | I tempi di risposta reali arrivano fino a 160 ms (Stop) e 119 ms (Set Target Speed), oltre i 48–88 ms dell'Addendum. Il limite di 2 s ha ampio margine. | Osservata | R; misura secondo A35. |
| A55 | La finestra di correlazione parte dall'invio del comando. | Decisa da Claude | — |

## Test

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A40 | T1: ogni ripetizione esegue Request Control → Start → 2,0 km/h → Stop, con conferma tra una ripetizione e l'altra. | Decisa da Claude | — |
| A41 | T3 si esegue a 2,5 km/h: prima 0,5 km/h (atteso `80 02 03`), poi 2,0 km/h. | Decisa da Claude | Ripete H, test 3. |
| A42 | T4: Start, poi 6,0 km/h con un solo comando, poi la rampa. Se 6,0 km/h non compare nei dati entro 5 s, il profilo si salta (non conclusivo). | Decisa da Claude | A: un salto 1,0 → 8,0 km/h è già stato fatto. |
| A43 | T5: il tempo misurato con il tuo pulsante è un'osservazione, non un dato. | Confermata | Richiesta T5. |
| A44 | T5: atteso «nastro a velocità» a 2–3 s dal comando, con ±0,5 s di tolleranza per il tempo di reazione. | Decisa da Claude | A §5 (osservazione a voce). |
| A45 | T7: ti chiede di restare sotto 5 km/h e ti dà 45 s. | Decisa da Claude | Margine rispetto al tetto; la fase resta sotto i 60 s. |
| A46 | T9: per avere contatori non a zero a nastro fermo, 15 s a 6,0 km/h e poi Pausa. | Decisa da Claude | H: la Pausa conserva i contatori, lo Stop li azzera. |
| A47 | Dopo la chiave, lo Start richiede il tuo tocco e un conto alla rovescia di 5 s. Il permesso vale 3 s e una sola volta. Vale in qualunque test che segue un `03`. | Decisa da Claude | Regola 8. |
| A48 | Feature: `24 06 00 00 01 00 00 00` = macchina: distanza totale, passo, energia, battito; target: solo velocità. | **Confermata** | R, T0. |
| A49 | Training Status atteso a nastro fermo: `01 01` Idle. Indicativo. | Confermata con riserva | H; A §2.7 lo dichiara non affidabile. |
| A58 | T4: dopo l'ultimo comando di salita e di discesa, il tool aspetta al massimo 3 s il valore finale nei dati. | Decisa da Claude | Tiene ogni profilo sotto i 60 s. |
| A59 | Bip di preavviso 5 s prima della durata massima. | Decisa da Claude | — |
| A60 | T2: la risposta e l'eventuale evento al comando 4,0 km/h a nastro fermo non sono documentati. Si registrano come non conclusivi. | Ipotesi | A riporta solo l'osservazione a voce. |

---

## Storico delle modifiche

- v1, 27/10/2026 12:35: prima stesura, dopo le risposte alle cinque domande.
- v3, 27/10/2026 sera: dati della prima sessione reale (R). A31 osservata; A30 estesa; A48 confermata; nuove A61–A66.
- v2, 27/10/2026: esito della mini pagina (A03 e A07 confermate, A04 corretta, A06 nuova). Aggiunte le voci A12b, A36–A39, A44–A50, A52, A55, A58–A60. Precisate A19 (Stop immediato prima delle indicazioni) e A35 (misura dalla chiamata di scrittura).
