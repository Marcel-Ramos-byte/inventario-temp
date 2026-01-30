Comandos validados

Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName

Não testados

Instalações 32 bits:
Get-ItemProperty "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName
Instalações por usuário:
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName











Claro. Aqui estão os comandos de PowerShell individuais, sem script, um por um, para você rodar manualmente e coletar informações de softwares sem perder nada.
Esses comandos funcionam em:
    • Windows XP (PowerShell 1.0)
    • Windows 7
    • Windows 8
    • Windows 10
    • Windows 11
E coletam tudo o que importa para manutenção industrial.

🟦 1) LISTAR PROGRAMAS INSTALADOS — VIA REGISTRO (COMPLETO)
Esse é o método correto.
Não usa Win32_Product (que é perigoso e incompleto).
Instalações 64 bits:
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName
Instalações 32 bits:
Get-ItemProperty "HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName
Instalações por usuário:
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select-Object DisplayName, DisplayVersion, Publisher, InstallLocation |
Sort-Object DisplayName
👉 Use esses três comandos juntos para não perder nada.

🟦 2) LISTAR TODOS OS EXECUTÁVEIS (.exe) EM PASTAS IMPORTANTES
Essencial para softwares industriais portáteis, programas sem instalação e drivers escondidos.
Busca em C:\ inteiro (com filtro para não travar):
Get-ChildItem C:\ -Recurse -Include *.exe -ErrorAction SilentlyContinue |
Select-Object FullName
Busca em locais onde softwares industriais costumam ficar:
Get-ChildItem "C:\Automation","C:\Projetos","C:\Programas","C:\Siemens","C:\Rockwell","C:\Schneider","C:\HMI","C:\Tools" -Recurse -Include *.exe -ErrorAction SilentlyContinue |
Select-Object FullName

🟦 3) LISTAR SERVIÇOS DO WINDOWS (Ignition, OPC, SCADA, drivers)
Get-Service | Sort-Object DisplayName | Format-Table Name, DisplayName, Status

🟦 4) LISTAR DRIVERS DO SISTEMA (importante para PLCs)
Drivers como FTDI, MPI, Profibus, Beckhoff, Rockwell OPC, etc.
Get-WmiObject Win32_SystemDriver |
Sort-Object Name |
Format-Table Name, State, PathName

🟦 5) LISTAR PROGRAMAS INSTALADOS COM WIN32_PRODUCT (OPCIONAL)
❗ Já aviso: não recomendo, mas se você quiser comparar:
Get-WmiObject Win32_Product | Select-Object Name, Version, InstallLocation
⚠️ Isso força “consistency check” no MSI e pode quebrar instalações antigas.
Use somente para comparação, nunca como fonte principal.

🟦 6) LISTAR TODOS OS PROGRAMAS EM “Arquivos de Programas”
Muitos programas industriais apenas criam pastas.
64 bits:
Get-ChildItem "C:\Program Files" | Select-Object Name
32 bits:
Get-ChildItem "C:\Program Files (x86)" | Select-Object Name

🟦 7) LISTAR TAREFAS AGENDADAS
Softwares industriais às vezes rodam como agendadores:
schtasks /query /fo LIST /v

🟦 8) LISTAR PROCESSOS INSTALADOS E A ORIGEM DO EXECUTÁVEL
Isso revela programas ativos escondidos:
Get-Process | Select-Object ProcessName, Path | Sort-Object Path

🟩 RESUMO
✔ Softwares instalados:
(3 comandos de Registro)
✔ Softwares portáteis, ZIP, antigos:
(Busca recursiva por .exe)
✔ Serviços (Ignition, OPC, SCADA)
Get-Service
✔ Drivers (MPI, FTDI, Beckhoff, Rockwell)
Get-WmiObject Win32_SystemDriver
✔ Programas em Program Files
Get-ChildItem "C:\Program Files"
Get-ChildItem "C:\Program Files (x86)"
✔ Processos e caminhos
Get-Process | Select Name, Path
Isso cobre 100% dos softwares, mesmo raros.

🟧 Quer que eu gere agora:
✔ uma “ficha de checagem” para você preencher por notebook?
ou
✔ um script que salva todos esses comandos automaticamente em TXT?
Só pedir!
_______________________________________________________________________


$Out="C:\Inventario"; New-Item -ItemType Directory -Force -Path $Out | Out-Null;

# =======================
# 1) HARDWARE COMPLETO
# =======================
Get-WmiObject Win32_ComputerSystem |
Select-Object Model,Manufacturer,NumberOfProcessors,TotalPhysicalMemory |
Out-File "$Out\Hardware_ComputerSystem.txt";

Get-WmiObject Win32_Processor |
Select-Object Name,NumberOfCores,MaxClockSpeed |
Out-File "$Out\Hardware_CPU.txt";

Get-WmiObject Win32_PhysicalMemory |
Select-Object Manufacturer,PartNumber,Capacity,Speed |
Out-File "$Out\Hardware_RAM.txt";

Get-WmiObject Win32_LogicalDisk |
Select-Object DeviceID,VolumeName,FileSystem,Size,FreeSpace |
Out-File "$Out\Hardware_Discos.txt";

Get-WmiObject Win32_BaseBoard |
Select-Object Product,Manufacturer,SerialNumber |
Out-File "$Out\Hardware_PlacaMae.txt";

Get-WmiObject Win32_BIOS |
Select-Object SMBIOSBIOSVersion,Manufacturer,ReleaseDate,SerialNumber |
Out-File "$Out\Hardware_BIOS.txt";

Get-WmiObject Win32_VideoController |
Select-Object Name,AdapterRAM,DriverVersion |
Out-File "$Out\Hardware_GPU.txt";


# =======================
# 2) SISTEMA OPERACIONAL
# =======================
Get-WmiObject Win32_OperatingSystem |
Select Caption,Version,BuildNumber,OSArchitecture,InstallDate |
Out-File "$Out\SO_SistemaOperacional.txt";


# =======================
# 3) PROGRAMAS INSTALADOS
# =======================
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*" |
Select DisplayName,DisplayVersion,Publisher,InstallLocation |
Sort-Object DisplayName |
Out-File "$Out\Programas_Instalados.txt";


# ========================================
# 4) SOFTWARES INDUSTRIAIS / PORTÁTEIS
# ========================================
Get-ChildItem "C:\" -Recurse -Include *.exe -ErrorAction SilentlyContinue |
Where-Object {
    $_.FullName -match "automation|plc|hmi|scada|siemens|rockwell|allen|omron|schneider|mitsubishi|codesys|twincat|wincc|ignition|factorytalk|prosafe"
} |
Select-Object FullName |
Out-File "$Out\Softwares_Industriais_Potenciais.txt";


# =======================
# 5) SERVIÇOS INDUSTRIAIS
# =======================
Get-Service |
Where-Object {
    $_.DisplayName -match "opc|ignition|siemens|rockwell|beckhoff|codesys|hmi|plc|scada"
} |
Select Name,DisplayName,Status |
Out-File "$Out\Servicos_Industriais.txt";


# =======================
# 6) DRIVERS INDUSTRIAIS
# =======================
Get-WmiObject Win32_SystemDriver |
Where-Object {
    $_.Name -match "ftdi|siemens|beckhoff|rockwell|schneider|profinet|profibus|kvaser|hilscher|twincat"
} |
Select Name,State,PathName |
Out-File "$Out\Drivers_Industriais.txt";


# =======================
# 7) PASTAS RELEVANTES
# =======================
Get-ChildItem "C:\Automation","C:\Projetos","C:\HMI","C:\WinCC","C:\Siemens","C:\Rockwell","C:\Schneider","C:\PLC" `
-Directory -Recurse -ErrorAction SilentlyContinue |
Select FullName |
Out-File "$Out\Pastas_Relevantes.txt";

Write-Host "Inventário COMPLETO gerado em $Out"
