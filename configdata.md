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





Write-Host "Inventário COMPLETO gerado em $Out"
