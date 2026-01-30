

* **SMBIOS/BIOS version**
* **ReleaseDate**
* **SerialNumber**
* **Placa-mãe (modelo)**
* **CPU**
* **Arquitetura (32/64 bits)**
* **Modo atual (Legacy/UEFI)**
* **TPM (se existir)**

E eu vou te mostrar **um método exato**, prático e replicável para qualquer notebook — inclusive máquinas antigas da manutenção industrial.

---

# ✅ **CAMINHO PROFISSIONAL PARA SABER SE SUPORTA WINDOWS 11**

O Windows 11 depende de **5 requisitos principais**:

### ✔ 1) CPU compatível

* Intel: 8ª geração ou superior
* AMD: Ryzen 2000 ou superior
* Alguns Celeron/Pentium modernos entram também

### ✔ 2) 64 bits obrigatório

Não existe Windows 11 de 32 bits.

### ✔ 3) UEFI + Secure Boot

BIOS antiga (Legacy/CSM) → NÃO suporta Windows 11.

### ✔ 4) TPM 2.0

Computadores industriais antigos quase sempre **não têm**.

### ✔ 5) 4 GB de RAM (mínimo)

---

# 🎯 **COMO DESCOBRIR TUDO ISSO COM OS DADOS QUE VOCÊ JÁ COLETA**

Vou mostrar como interpretar **cada pedaço do seu inventário** para responder:

> “Suporta Windows 11 ou não?”

---

# 🟦 **1) Verificar CPU (fundamental)**

Você pega isso do inventário:

```
Win32_Processor → Name
```

Exemplos:

| CPU                            | Win11? |
| ------------------------------ | ------ |
| Intel Core i5-2520M (2ª gen)   | ❌ Não  |
| Intel Core i5-3320M (3ª gen)   | ❌ Não  |
| Intel Core i5-6200U (6ª gen)   | ❌ Não  |
| Intel Core i5-8250U (8ª gen)   | ✔ Sim  |
| Intel Core i7-10510U (10ª gen) | ✔ Sim  |
| Ryzen 5 2500U                  | ✔ Sim  |
| Ryzen 5 1600                   | ❌ Não  |

🧠 Identificação da geração Intel:
Core **iX-Y###**
→ O número depois do “-” indica a geração.

Exemplo:
i5-8250U → 8ª geração → **Win11 OK**

---

# 🟩 **2) Verificar arquitetura (32/64 bits)**

Seu inventário traz:

```
OSArchitecture : 32-bit ou 64-bit
```

* 32 bits → ❌ NUNCA suporta Win11
* 64 bits → ✔ passa nesse requisito

---

# 🟧 **3) Verificar se BIOS é Nova o Suficiente (ReleaseDate + SMBIOS)**

Win11 costuma exigir **SMBIOS 3.0+**, mas isso depende do fabricante.

### Regra prática:

| ReleaseDate da BIOS | Situação            |
| ------------------- | ------------------- |
| Antes de 2012       | ❌ Não suporta Win11 |
| 2012–2016           | ❌ Provavelmente não |
| 2017–2021           | ✔ Pode suportar     |
| 2021+               | ✔ Suporta           |

Máquinas com BIOS 2018+ geralmente já têm:

* UEFI
* SecureBoot
* suporte a TPM 2.0
* processador mais moderno

---

# 🟫 **4) Verificar se TPM existe (MESMO EM XP/7!)**

Use este comando:

```powershell
Get-WmiObject -Namespace "root\CIMV2\Security\MicrosoftTpm" -Class Win32_Tpm
```

Se retornar algo → TPM existe.

Se der erro → sem TPM.

Win11 exige **TPM 2.0**, mas presença já indica hardware moderno.

---

# 🟪 **5) Verificar se a máquina está em Legacy ou UEFI**

```powershell
Get-WmiObject -Class Win32_BIOS | Select FirmwareType
```

* `UEFI` → ✔ ok
* `Legacy` → ❌ não suporta Win11 até trocar modo na BIOS (se tiver opção)

Notebooks velhos XP/7 costumam ser **Legacy-only**.

---

# 🟨 **6) Juntando tudo — Regras para decidir**

### ✔ SUPORTA Windows 11 se:

* CPU 8ª geração Intel / Ryzen 2000 ou superior
* 64 bits
* BIOS moderna (2017+)
* TPM presente (idealmente 2.0)
* UEFI disponível

### ❌ NÃO SUPORTA Windows 11 se:

* Intel até 7ª geração
* AMD até Ryzen 1000
* BIOS anterior a 2012
* 32 bits
* sem TPM
* só Legacy, sem modo UEFI

---

# ⭐ **AGORA O MAIS IMPORTANTE PARA VOCÊ (manutenção industrial)**

Você tem máquinas:

* Windows XP
* Windows 7
* Windows 10
* Windows 11

E **muitas dessas máquinas antigas NÃO devem ser forçadas a ir para Win10/11**, especialmente se possuem:

* softwares de PLC antigos (Step7, GX Developer, RSLogix 500…)
* drivers PCMCIA / USB industriais antigos
* interfaces seriais reais
* chaves de proteção paralela (LPT)
* Hardlocks HASP muito antigos

Então você deve usar isso assim:

---

# 🧠 **Método Prático que VOCÊ pode usar:**

### ✔ Passo 1: Coletar inventário (CPU, BIOS, TPM, Sistema Operacional)

### ✔ Passo 2: Verificar CPU (se é 8ª geração ou superior)

### ✔ Passo 3: Verificar modo Legacy/UEFI

### ✔ Passo 4: Verificar BIOS ReleaseDate

### ✔ Passo 5: Verificar TPM

### ✔ Passo 6: Decidir:

---

# 🟢 **Máquinas que podem ir para Windows 11**

* Dell Latitude 5490, 5590
* Lenovo ThinkPad T480
* HP ProBook 450 G6
* Ryzen 2500U+
* i5-8250U+
* Qualquer coisa a partir de 2017
* BIOS moderna
* UEFI
* TPM 2.0

---

# 🔴 **Máquinas que NÃO devem ir para Win11 (XP/7)**

Essas devem ficar em:

* Windows 7
* Windows 10 LTSC
* ou **máquinas virtuais XP/7** para rodar softwares antigos

Exemplos:

* Dell Latitude E5400/E5420/E5430
* Lenovo T420/T430
* HP EliteBook 8460p
* Qualquer Core 2 Duo
* Qualquer 1ª–6ª geração
* BIOS antiga
* só Legacy
* sem TPM

---

# ❓ Quer que eu gere um **comando PowerShell único** que verifica automaticamente se o computador suporta Windows 11?

Ele verificaria:

* CPU geração
* TPM
* UEFI/Legacy
* Arquitetura
* BIOS ReleaseDate
* SMBIOS version

E te daria:

```
Suporta Windows 11: SIM/NÃO
Motivo: CPU antiga / sem TPM / BIOS antiga / Legacy-only etc.
```
