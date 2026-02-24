# 1️⃣ Mindmap архітектури Metasploit

```mermaid
mindmap
  root((Metasploit Framework))
    Core
      Module Loader
      Handler
      Session Manager
    Modules
      Exploit
      Auxiliary
      Post
      Encoder
      Payload
    Payload Types
      Single
      Staged
      Stageless
    Sessions
      Shell
      Meterpreter
    Interfaces
      msfconsole
      RPC API
      Automation
```

---

# 2️⃣ Логічна модель атаки (зв’язок із Kill Chain)

```mermaid
flowchart LR
    A[Recon] --> B[Exploit Selection]
    B --> C[Exploit Execution]
    C --> D[Payload Delivery]
    D --> E[Session Creation]
    E --> F[Post-Exploitation]
    F --> G[Privilege Escalation]
    G --> H[Lateral Movement]
    H --> I[Persistence]
```

---

# 3️⃣ Reverse TCP — мережева взаємодія attacker ↔ victim

```mermaid
sequenceDiagram
    participant Attacker
    participant Victim

    Attacker->>Victim: Deliver exploit
    Victim->>Victim: Execute shellcode
    Victim->>Attacker: Reverse TCP connect
    Attacker->>Victim: Stage 2 (if staged)
    Attacker->>Attacker: Session created
```

---

# 4️⃣ Bind TCP — мережева взаємодія

```mermaid
sequenceDiagram
    participant Attacker
    participant Victim

    Attacker->>Victim: Deliver exploit
    Victim->>Victim: Start listening port
    Attacker->>Victim: Connect to open port
    Attacker->>Attacker: Session created
```

---

# 5️⃣ Staged vs Stageless Payload

```mermaid
flowchart TD
    A[Exploit Triggered]

    A --> B1[Stage 1 Loader]
    B1 --> C1[Download Stage 2]
    C1 --> D1[Meterpreter Session]

    A --> B2[Full Payload Delivered]
    B2 --> D2[Session Created]
```

---

# 6️⃣ Життєвий цикл сесії

```mermaid
flowchart LR
    A[Exploit Start] --> B[Shellcode Execution]
    B --> C[Payload Init]
    C --> D[Network Callback]
    D --> E[Handler Accepts]
    E --> F[Session Object Created]
    F --> G[Post Modules]
    G --> H[Session Closed]
```

---

# 7️⃣ Архітектурна взаємодія в лабораторії

```mermaid
flowchart LR
    Kali[Kali Linux - Attacker] -->|Exploit| MS3[Metasploitable3 - Victim]
    MS3 -->|Reverse TCP| Kali
    Kali -->|Handler| Session[Meterpreter Session]
    Session --> Post[Post-Exploitation Modules]
```

---

# Як користуватись

Логіка наступна:

1. Спочатку розібрати архітекруту **Metasploit**
2. Потім з'ясувати/згадати як відбувається **Reverse TCP sequence**
3. Далі узагальнення у вигляді **Lifecycle**
4. **Бажано**, після практики  — повернутись та передивитсь діаграми

Така послідовність спрямована на формування **замкненого когнітивного циклу навчання**.


