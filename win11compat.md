# ============================
# VERIFICADOR DE COMPATIBILIDADE WINDOWS 11
# Compatível com XP → 11
# ============================

Write-Host "`n--- Verificando compatibilidade com Windows 11 ---`n"

# Resultado final
$Resultado = [ordered]@{
    CPU_Geracao = "N/D"
    CPU_Compativel = $false
    Arquitetura_64bits = $false
    UEFI = $false
    TPM_Presente = $false
    BIOS_Recente = $false
    Suporta_Windows_11 = $false
    Motivo = @()
}

# ===============================
# 1) CPU → determinar geração Intel / AMD
# ===============================
try {
    $cpuName = (Get-WmiObject Win32_Processor).Name
} catch {
    $cpuName = "N/D"
}

$Resultado.CPU = $cpuName

# Detectar geração Intel (se for Core iX-XXXX)
if ($cpuName -match "i[3579]-([0-9]{4})") {
    $gen = [int]$matches[1].Substring(0,1)
    $Resultado.CPU_Geracao = $gen
    if ($gen -ge 8) { $Resultado.CPU_Compativel = $true }
} elseif ($cpuName -match "Ryzen\s*([0-9]{4})") {
    # Ryzen 2000+ suportam Win11
    $rz = [int]$matches[1]
    if ($rz -ge 2000) { $Resultado.CPU_Compativel = $true }
    $Resultado.CPU_Geracao = "Ryzen $rz"
}

# ===============================
# 2) Arquitetura (32/64 bits)
# ===============================

try {
    $arch = (Get-WmiObject Win32_OperatingSystem).OSArchitecture
    if ($arch -match "64") { $Resultado.Arquitetura_64bits = $true }
} catch {}

# ===============================
# 3) BIOS / UEFI
# ===============================

try {
    $bios = Get-WmiObject Win32_BIOS
    $Resultado.BIOS = $bios.SMBIOSBIOSVersion

    $rd = $bios.ReleaseDate
    $DataStr = [System.Management.ManagementDateTimeConverter]::ToDateTime($rd)
    $Resultado.BIOS_Data = $DataStr

    # BIOS recente: 2017+
    if ($DataStr.Year -ge 2017) { $Resultado.BIOS_Recente = $true }
} catch {}

# Checar UEFI
try {
    $firm = Get-WmiObject -Class Win32_ComputerSystem
    if ($firm.FirmwareType -eq 2 -or $firm.FirmwareType -eq "UEFI") {
        $Resultado.UEFI = $true
    }
} catch {}

# ===============================
# 4) TPM
# ===============================

try {
    $tpm = Get-WmiObject -Namespace "Root\CIMV2\Security\MicrosoftTpm" -Class Win32_Tpm -ErrorAction Stop
    if ($tpm) { $Resultado.TPM_Presente = $true }
} catch {}

# ===============================
# 5) Decisão final
# ===============================

if (-not $Resultado.CPU_Compativel) { $Resultado.Motivo += "CPU muito antiga" }
if (-not $Resultado.Arquitetura_64bits) { $Resultado.Motivo += "Sistema é 32 bits" }
if (-not $Resultado.UEFI) { $Resultado.Motivo += "Não usa UEFI (Legacy-only)" }
if (-not $Resultado.TPM_Presente) { $Resultado.Motivo += "Sem TPM 2.0" }
if (-not $Resultado.BIOS_Recente) { $Resultado.Motivo += "BIOS muito antiga" }

if (
    $Resultado.CPU_Compativel -and
    $Resultado.Arquitetura_64bits -and
    $Resultado.UEFI -and
    $Resultado.TPM_Presente -and
    $Resultado.BIOS_Recente
) {
    $Resultado.Suporta_Windows_11 = $true
}

# ===============================
# EXIBIR RESULTADO
# ===============================

Write-Host "`nRESULTADO FINAL:"
$Resultado

Write-Host "`nSuporte Windows 11: " -NoNewline
if ($Resultado.Suporta_Windows_11) {
    Write-Host "SIM" -ForegroundColor Green
} else {
    Write-Host "NÃO" -ForegroundColor Red
    Write-Host "Motivos: " + ($Resultado.Motivo -join ", ")
}
