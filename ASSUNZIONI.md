# Registro delle assunzioni – Banco prova Run100E

Versione 7 · 30 ottobre 2026 · vale per il tool v1.3.0 (`run100e-banco-prova.html`)

Fonti di verità:
- «Handoff tecnico» del 25/10/2026 (H);
- «Addendum tecnico» del 27/10/2026 (A), che prevale su H in caso di contrasto;
- registro della mini pagina del 27/10/2026 alle 16:36, inviato da Stefano (M);
- registro e report del banco prova v1.0.0 del 27/10/2026, 18:05–18:14 (R);
- registro del banco prova v1.1.0 del 30/10/2026, 09:37–09:41, T5 e T9 (R2);
- registro del banco prova v1.1.0 del 30/10/2026, 09:45, T6 (R3);
- registro del banco prova v1.2.0 del 30/10/2026, 10:18–10:21, T10 e T11, con le spiegazioni di Stefano su T11 (R4).

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
| A03 | Dopo una caduta, Chrome si ricollega allo stesso dispositivo senza un nuovo `requestDevice`. | Confermata per una disconnessione volontaria (M); **non verificata** dopo Bluetooth spento e riacceso | R4: dopo la riaccensione Chrome non si è ricollegato da solo; le scritture fallivano ancora circa 30 s dopo. Va riprovato con la v1.3 (che chiude il collegamento appeso prima di riprovare). |
| A04 | Il numero di serie (0x2A25) non si legge. | **Confermata, con correzione** | M: la caratteristica non esiste. Mancano anche 0x2A27 (revisione hardware) e la blocklist non c'entra. Il tool non le legge. |
| A05 | Se la pagina va in background il tool invia subito Stop. Senza Wake Lock attivo i test non partono. | **Confermata** | R4, T10: Stop 2 ms dopo l'uscita da Chrome, risposta `80 08 01` dopo 97 ms, prima del ritorno; dati ricevuti anche in background; il nastro si è fermato subito. Assenza di circa 2,5 s: un background più lungo non è stato provato, ma lo Stop parte all'uscita. |
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
| A32 | Dopo la ripresa da Pausa il nastro riparte da 1,0 km/h. | **Confermata** | R3: 1,0 km/h nei dati 1,8 s dopo `07` e osservazione a voce «lento, circa 1 km/h». Coincide con H. |
| A33 | Reset (`01`) a nastro fermo: risposta `80 01 01`, nessun evento, contatori a zero nel pacchetto successivo (circa 1,2 s). La console torna alla schermata iniziale, quella che compare quando l'app si collega, con tutti i contatori azzerati. Il Request Control inviato subito dopo risponde `80 00 01`. Resta **non verificato** se il Reset revochi il controllo: il tool rinvia comunque Request Control. | **Osservata** (1 volta) | R2, 09:39:52; osservazione a voce di Stefano. H e A si contraddicevano; ora c'è un dato. |
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
| A67 | Dopo la Pausa: velocità nei dati a 0,0 in circa 1 s, Training Status `01 01` Idle nello stesso secondo, contatori **conservati** (almeno 10 s). | **Osservata** (1 volta) | R2, 09:39:42–09:39:52. Conferma l'Handoff sui contatori. |
| A68 | La correzione A61 funziona: dopo la Pausa il `02 01` è classificato come causato dal tool (+591 ms in R2, +613 ms in R3) e il test non si interrompe. Pausa → `02 01` osservato 4 volte su 4. | **Confermata** | R2, R3. |
| A69 | Durante un salto di velocità i dati (e l'evento `05`) riportano il valore di arrivo già dopo circa 2 s, mentre il nastro accelera gradualmente: da 2 a 8 km/h in circa 6–7 s, cioè circa 1 km/h al secondo (stima). Conseguenza per l'app: la velocità nei dati non dice quando il nastro è davvero a regime. | Osservata (A44 + R2) | Conferma A §2 («valore impostato, non misura»). La stima di 1 km/h al secondo è un'ipotesi ricavata da un solo salto. |
| A70 | In T10 e T11 il background e la perdita del collegamento sono provocati di proposito. Tutte le protezioni restano attive (Stop, allarme, ricollegamento, Stop dopo il ricollegamento); cambia solo che il test non si interrompe, perché deve osservare l'esito. In ogni altro test questi eventi interrompono il test come prima. | Decisa da Claude | Senza questa eccezione i due test si interromperebbero nel momento esatto che devono misurare. |
| A71 | T10 giudica se il codice gira in background da tre segnali: Stop inviato entro 200 ms dall'uscita; risposta allo Stop arrivata prima del ritorno su Chrome; pacchetti Treadmill Data ricevuti durante il background (circa uno al secondo). Se Chrome congela la pagina, l'uscita viene registrata solo al ritorno: in quel caso lo rivelano la tua osservazione e i pacchetti mancanti. | Decisa da Claude | Il tool registra anche gli eventi `freeze` e `resume` di Chrome, se arrivano. |
| A72 | Ordine dei nuovi test: T10 (background, il nastro resta senza controllo per pochi secondi) prima di T11 (perdita del collegamento, il nastro gira per tutto il tempo in cui il Bluetooth resta spento). Entrambi a 2,0 km/h. T9 resta facoltativo ma non è più l'ultimo. | Decisa da Claude | Regola 6: dal più sicuro al più impegnativo. Nella proposta del 30/10 l'ordine era invertito. |
| A73 | Se un ricollegamento manuale fallisce (per esempio dopo aver spento e riacceso il Bluetooth), al tocco successivo su «Collega» si sceglie di nuovo il dispositivo. | Decisa da Claude | Dopo un riavvio del Bluetooth il dispositivo memorizzato da Chrome potrebbe non essere più valido (ipotesi). |
| A74 | Sorveglianza del collegamento: se per più di 3 s non arriva alcuna notifica (dati, Training Status, eventi, indicazioni), il tool considera il collegamento perso: allarme, test interrotto (salvo T11), tentativo di Stop, ricollegamento. Rilevazione attesa tra 3,0 e 3,25 s dall'ultimo pacchetto. Un pacchetto perso (buco di 2 s) è tollerato. | Decisa da Claude, su dati R4 | R4: spegnendo il Bluetooth del telefono **Chrome non ha segnalato la disconnessione** per 42 s; il tool v1.2 non se n'è accorto. Le notifiche arrivano ogni secondo in tutte le sessioni (A62, A63). Lo stesso controllo serve nell'app. |
| A75 | Una scrittura che fallisce («GATT operation failed», NetworkError) = collegamento perso. Il ricollegamento chiude prima ogni collegamento rimasto «appeso» in Chrome e non si arrende al primo Stop fallito: chiude e riprova, fino a 30 tentativi. | Decisa da Claude, su dati R4 | R4: alle 10:20:43 Chrome risultava ancora collegato ma ogni scrittura falliva. |
| A76 | Sull'allarme compare il pulsante «Ricollega scegliendo il dispositivo». Serve se Chrome non riesce a riusare il dispositivo memorizzato dopo un riavvio del Bluetooth. La scelta richiede un tuo tocco. Solo il dispositivo scelto per ultimo può segnalare disconnessioni. | Decisa da Claude | Ipotesi: dopo il riavvio del Bluetooth il dispositivo memorizzato potrebbe non essere più valido. In T11 conviene aspettare 20 s prima di toccarlo, per sapere se il ricollegamento automatico funziona. |
| A77 | Quando il collegamento cade, il tapis roulant continua alla stessa velocità e la console passa alla schermata di una sessione avviata senza Bluetooth. Si ferma solo con la console o con la chiave. | **Osservata** | R4 (Stefano), coerente con H. Dopo il ricollegamento lo Stop potrebbe richiedere un nuovo Request Control: il tool lo gestisce (A19), non ancora osservato. |
| A55 | La finestra di correlazione parte dall'invio del comando. | Decisa da Claude | — |

## Test

| ID | Assunzione | Stato | Fonte / motivo |
|---|---|---|---|
| A40 | T1: ogni ripetizione esegue Request Control → Start → 2,0 km/h → Stop, con conferma tra una ripetizione e l'altra. | Decisa da Claude | — |
| A41 | T3 si esegue a 2,5 km/h: prima 0,5 km/h (atteso `80 02 03`), poi 2,0 km/h. | Decisa da Claude | Ripete H, test 3. |
| A42 | T4: Start, poi 6,0 km/h con un solo comando, poi la rampa. Se 6,0 km/h non compare nei dati entro 5 s, il profilo si salta (non conclusivo). | Decisa da Claude | A: un salto 1,0 → 8,0 km/h è già stato fatto. |
| A43 | T5: il tempo misurato con il tuo pulsante è un'osservazione, non un dato. | Confermata | Richiesta T5. |
| A44 | T5: il nastro arriva a 8 km/h circa 6–7 s dopo il comando. Atteso del tempo misurato a occhio: 5–8 s. | **Osservata** (Stefano, 30/10, guardando la console e il nastro) | Le misure a pulsante (8,7 s il 27/10 per distrazione, 6,9 s il 30/10) restano indicative. Il valore di 2–3 s di A §5 non è confermato. |
| A45 | T7: ti chiede di restare sotto 5 km/h e ti dà 45 s. | Decisa da Claude | Margine rispetto al tetto; la fase resta sotto i 60 s. |
| A46 | T9: per avere contatori non a zero a nastro fermo, 15 s a 6,0 km/h e poi Pausa. | **Confermata** | R2: dopo la Pausa l'energia è rimasta a 1 kcal per 10 s, fino al Reset. Dopo uno Stop i contatori si azzerano in circa 2 s (A §5). |
| A47 | Dopo la chiave, lo Start richiede il tuo tocco e un conto alla rovescia di 5 s. Il permesso vale 3 s e una sola volta. Vale in qualunque test che segue un `03`. | Decisa da Claude | Regola 8. |
| A48 | Feature: `24 06 00 00 01 00 00 00` = macchina: distanza totale, passo, energia, battito; target: solo velocità. | **Confermata** | R, T0. |
| A49 | Training Status atteso a nastro fermo: `01 01` Idle. Indicativo. | Confermata con riserva | H; A §2.7 lo dichiara non affidabile. |
| A58 | T4: dopo l'ultimo comando di salita e di discesa, il tool aspetta al massimo 3 s il valore finale nei dati. | Decisa da Claude | Tiene ogni profilo sotto i 60 s. |
| A59 | Bip di preavviso 5 s prima della durata massima. | Decisa da Claude | — |
| A60 | T2: la risposta e l'eventuale evento al comando 4,0 km/h a nastro fermo non sono documentati. Si registrano come non conclusivi. | Ipotesi | A riporta solo l'osservazione a voce. |

---

## Storico delle modifiche

- v1, 27/10/2026 12:35: prima stesura, dopo le risposte alle cinque domande.
- v7, 30/10/2026: tool v1.3.0. T10 riuscito (A05 confermata). T11 ha mostrato che Chrome non segnala la perdita del collegamento con il Bluetooth spento: nuove A74–A77, A03 da riverificare.
- v6, 30/10/2026: tool v1.2.0 con T10 e T11. A44 aggiornata con l'osservazione di Stefano; nuove A69–A73. L'intestazione del registro ora riporta «assunzioni v6».
- v5, 30/10/2026: T6 completato (R3). A32 confermata, A68 aggiornata. Il tool v1.1.0 riporta ancora «assunzioni v3» nell'intestazione: l'etichetta verrà allineata alla prossima versione del tool.
- v4, 30/10/2026: dati di T5 e T9 (R2). A33 osservata, A46 confermata, A44 da rivedere; nuove A67–A68. T6 non ancora eseguito con la v1.1.
- v3, 27/10/2026 sera: dati della prima sessione reale (R). A31 osservata; A30 estesa; A48 confermata; nuove A61–A66.
- v2, 27/10/2026: esito della mini pagina (A03 e A07 confermate, A04 corretta, A06 nuova). Aggiunte le voci A12b, A36–A39, A44–A50, A52, A55, A58–A60. Precisate A19 (Stop immediato prima delle indicazioni) e A35 (misura dalla chiamata di scrittura).
