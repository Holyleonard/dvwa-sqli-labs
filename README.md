# 🔐 DVWA SQL Injection Labs - Report Collection

Collection di report tecnici sui lab di **SQL Injection** di [DVWA (Damn Vulnerable Web Application)](https://github.com/Holyleonard/dvwa-sqli-labs), eseguiti in ambiente containerizzato isolato a scopo formativo.

## 📁 Struttura
├── README.md # Questo file
│ └── dvwa-sqli-low-lab.md
│ └── dvwa-sqli-medium-lab.md
│ └── dvwa-sqli-high-blind.md


## 🎯 Obiettivo
Documentare metodologie, payload e mitigazioni per vulnerabilità SQL Injection a diversi livelli di difficoltà, dimostrando competenze in:
- ✅ HTTP Interception (Burp Suite)
- ✅ SQL Injection (UNION, Boolean-Based, Time-Based Blind)
- ✅ Automazione con sqlmap (cookie injection, tamper, level/risk tuning)
- ✅ Analisi root-cause e secure coding
- ✅ Technical reporting per portfolio/CV

## 🛠️ Ambiente di Test
- **OS**: Kali Linux (Rolling)
- **Target**: `vulnerables/web-dvwa` (Docker)
- **Proxy**: Burp Suite Community
- **Isolamento**: Rete localhost, nessun accesso esterno

## ⚠️ Disclaimer
> *"Tutte le attività documentate sono state eseguite esclusivamente su infrastruttura containerizzata isolata di proprietà. Nessun accesso, modifica o esposizione di sistemi o dati di terzi. Scopo puramente formativo, conforme alle linee guida OWASP e alla normativa vigente in materia di cybersecurity."*

## 📚 Riferimenti
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [DVWA GitHub](https://github.com/digininja/DVWA)
- [sqlmap Documentation](https://sqlmap.org/doc/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---
*Report creati da Leonardo Sorrentini - Maggio 2026*
