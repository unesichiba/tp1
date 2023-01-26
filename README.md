#Script Name: ServerHealthReport.ps1
$ComputerName = $($env:COMPUTERNAME)
$Global:Alerts = 0
$current_date= (Get-Date -Format D)
$ScriptPath = Get-Location
$ServerList = Import-CSV "$ScriptPath\ServerList.csv" -Delimiter ';'
$ReportFileName = "$ScriptPath\ServerHealthReport.html"
$ReportTitle = "Server Health Report"
$UptimeDayMax = 45
$RAMFree = 15
$CPUCritical = 75
$DrvWarning = 15

Write-Host "Creating report..." -Foreground Yellow
# Create output files and nullify display output
New-Item -ItemType file $ReportFileName -Force > $null

 #EXPLAIN 
 Add-Content $ReportFileName "<html>"
Add-Content $ReportFileName "<head>"

Add-Content $ReportFileName "<meta http-equiv=Content-Type content=text/html; charset=UTF-8 />"

Add-Content $ReportFileName "</head>"

Add-Content $ReportFileName "<body>"

Add-Content $ReportFileName "<p>'Bonjour'</p>"

Add-Content $ReportFileName "<p>Vous trouverez ci-dessous la météo du <b>$current_date<b>. </p>"

Add-Content $ReportFileName "<h1><u>PRODUCTION:</u></h1>"

Add-Content $ReportFileName "<table style= width:15%; border=1>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center>&#10060;</center></td>"
Add-Content $ReportFileName "<td><center><b>ERREUR</b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center> &#9888;</center></td>"
Add-Content $ReportFileName "<td><center><b><b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center>&#9989;</center></td>"
Add-Content $ReportFileName "<td><center><b>RAS<b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "</table>"
Add-Content $ReportFileName "<h2>Infra Azure :</h2>"




 





	#--------------------
    #table with disk spec
    #--------------------
    <#Add-Content $FileName1 "<tr>"
	Add-Content $FileName1 "<td align='center'>$devid</td>"
	Add-Content $FileName1 "<td align='center'>$volName</td>"
	Add-Content $FileName1 "<td align='right'>$totSpace</td>"
	Add-Content $FileName1 "<td align='right'>$usedSpace</td>"
	Add-Content $FileName1 "<td align='right'>$frSpace</td>"#>

#------
# Main
#------
Write-Host "Collecting data for servers in list..."
ForEach ($Server in $Serverlist)
{
$ServerName = $($Server.Server)
$ServerDesc = $($Server.Description)

Add-Content $ReportFileName "<table style=width:100% border=1>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<th colspan=6  style='background-color:#97BDF2;width:100px;'>$ServerName</th>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<th   style='background-color:#97BDF2;width:100px;'>Services</th>"
Add-Content $ReportFileName " <th style='background-color:#97BDF2 ;width:84px'>Statut</th>"
Add-Content $ReportFileName " <th style='background-color:#97BDF2 ;width:84px'>AVR</th>"
Add-Content $ReportFileName " <th style='background-color:#97BDF2;width:400px'>Détails</th>"

Add-Content $ReportFileName "<tr>"
#Add-Content $ReportFileName "<td style='writing-mode: vertical-rl;' rowspan=6><center><b>$ServerName</b></center></td>"
Add-Content $ReportFileName "<td rowspan=6>"
Add-Content $ReportFileName "<table border=1 style=width:100%>"
Add-Content $ReportFileName "<td style='writing-mode: vertical-rl;'rowspan=6><center><b>Check Etat des Vms</b></center></td>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td ><center><b>Consommation CPU</b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center><b>Consommation RAM</b></center></td>"
Add-Content $ReportFileName "</tr>"
<#Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center><b>Consommation Disk IOs</b></center></td>"
Add-Content $ReportFileName "</tr>"#>
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center><b>Consommation Network </b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center><b>Percent Disk Space</b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center><b>Uptime</b></center></td>"
Add-Content $ReportFileName "</tr>"
Add-Content $ReportFileName "<th colspan=2  >"
Add-Content $ReportFileName "<center><b> IIS </b></center>"
Add-Content $ReportFileName "</th>"
Add-Content $ReportFileName "</table>"
Add-Content $ReportFileName "</td>"
	try {
		Write-Host "Total Alerts: $Global:Alerts"
		Write-Host "Server Name: $ServerName, $ServerDesc" -Foreground Green
		$CPUs = (Get-WMIObject Win32_ComputerSystem -Computername $ServerName -ErrorAction Stop).numberofprocessors
                $CPUavg = Get-WmiObject win32_processor -computername $ServerName | Measure-Object -property LoadPercentage -Average
                $CPUavg=$CPUavg.Average
		Get-WMIObject -computername $ServerName -class win32_processor -ErrorAction Stop | ForEach {$TotalCores = $TotalCores + $_.numberofcores}
		$ComputerSystem = Get-WmiObject -ComputerName $ServerName -Class Win32_operatingsystem -Property CSName, TotalVisibleMemorySize, FreePhysicalMemory -ErrorAction Stop
		$BootTime = (Get-WmiObject win32_operatingSystem -computer $ServerName -ErrorAction Stop).lastbootuptime
		}
    catch {
		Write-Host "ERROR collecting data for $ServerName " -ForegroundColor Yellow
		$_.Exception
		"Continuing..."
    }
 
 
#----------
# CPU status
#----------

$TotalCores = 0 
Get-WMIObject -computername $ServerName -class win32_processor | ForEach {$TotalCores = $TotalCores + $_.numberofcores}
If ($TotalCores -eq 1)
	{$CPUSpecs = "CPU: $CPUs with 1 core, Avg Load %: $CPUavg"}
else
	{$CPUSpecs = "CPU: $CPUs with $TotalCores cores, Avg Load %: $CPUavg"}

IF ($CPUavg -ge $CPUCritical)
	{
		Add-Content $ReportFileName "<td><center>&#10060;</center></td>"
	}
	elseif($CPUavg -le $CPUCritical)
	{
		Add-Content $ReportFileName "<td><center>&#9989;</center></td>"
        }
        else {
            Add-Content $ReportFileName "<td><center><b></b></center></td>"
        }
		
Add-Content $ReportFileName "<td><center>$CPUavg%</center></td>"
Add-Content $ReportFileName "<td><center></center></td>"



#----------
# RAM status
#----------
$MachineName = $ComputerSystem.CSName
$FreePhysicalMemory = ($ComputerSystem.FreePhysicalMemory) / (1mb)
$TotalVisibleMemorySize = ($ComputerSystem.TotalVisibleMemorySize) / (1mb)
$TotalVisibleMemorySizeR = $TotalVisibleMemorySize
$TotalFreeMemPerc = [math]::round(($FreePhysicalMemory/$TotalVisibleMemorySize)*100)
$TotalFreeMemPercR = $TotalFreeMemPerc
$RAMSpecs = "RAM: $TotalVisibleMemorySizeR GB with $TotalFreeMemPercR% free"
$UsedRam = 100 -$TotalFreeMemPerc
Add-Content $ReportFileName "<tr>"
IF ($TotalFreeMemPerc -le $RAMFree)
	{
		Add-Content $ReportFileName "<td><center>&#10060;</center></td>"
        
	}
	elseif ($TotalFreeMemPerc -gt $RAMFree) {
 
		Add-Content $ReportFileName "<td><center>&#9989;</center></td>"
	}
    else {
        Add-Content $ReportFileName "<td><center><b></b></center></td>"
    }
	Add-Content $ReportFileName "<td><center>$UsedRam % </center></td>"
	Add-Content $ReportFileName "<td><center></center></td>"
	Add-Content $ReportFileName "</tr>"


#--------------------------
# Begin Server Disk tables
#--------------------------

 <#Disk IOs
Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center>&#9989</center></td>"
Add-Content $ReportFileName "<td><center><b></b></center></td>"
Add-Content $ReportFileName "</tr>"#>

#---------------------
# Network Interface
#---------------------

$networkAdapter = Get-WmiObject -Class Win32_PerfFormattedData_Tcpip_NetworkInterface
foreach ($adapter in $networkAdapter) {
	#if ($adapter.Name -like "*Ethernet*") {
    $bytesSent = $adapter.BytesSentPersec
    $bytesReceived = $adapter.BytesReceivedPersec
    $totalBytes = $bytesSent + $bytesReceived

    if (( $totalBytes -gt 0 ) -and ($totalBytes -lt 1000000)) {
		Add-Content $ReportFileName "<tr>"
		Add-Content $ReportFileName "<td><center>&#9989</center></td>"
		Add-Content $ReportFileName "<td><center>$totalBytes<b></b></center></td>"
        #$body += "<p>Network status for $($adapter.Name) is OK. Total bytes sent and received: $totalBytes.</p>"
		Add-Content $ReportFileName "<td><center></center></td>"
		
		
    } elseif ($totalBytes -eq 0) {
		Add-Content $ReportFileName "<tr>"
		Add-Content $ReportFileName "<td><center>&#10060</center></td>"
		Add-Content $ReportFileName "<td><center>$totalBytes<b></b></center></td>"
		Add-Content $ReportFileName "<td><center><p style='color:red'>ERREUR: Network interface  is DOWN. </p></center></td>"
		
    }else {
		Add-Content $ReportFileName "<tr>"
		Add-Content $ReportFileName "<td><center> &#9888;</center></td>"
		Add-Content $ReportFileName "<td><center>ax$totalBytes<b></b></center></td>"
		Add-Content $ReportFileName "<td><center>ac</center></td>"
		
		
	}
	

	Add-Content $ReportFileName "</tr>"
#}
}
<#Add-Content $ReportFileName "<tr>"
Add-Content $ReportFileName "<td><center>&#9989</center></td>"
Add-Content $ReportFileName "<td><center><b></b></center></td>"
Add-Content $ReportFileName "</tr>"#>


# Disk Space
$disks = Get-WmiObject -Class Win32_LogicalDisk -Filter "DeviceID='C:'"
foreach ($disk in $disks) {
    if ($disk.DriveType -eq 3) {
        $freeSpace = $disk.FreeSpace
        $size = $disk.Size
        $percentFree = [Math]::Round(($freeSpace / $size) * 100, 2)
		if ($percentFree -gt $DrvWarning) {
		
			Add-Content $ReportFileName "<tr><td><center>&#9989</center></td><td><center>$($disk.DeviceID)$percentFree%</center></td><td><b></b></td></tr>"
			#Add-Content $ReportFileName "<td><center>  ok &#9989</center></td>"
            #Add-Content $ReportFileName"<p>Disk status for $($disk.DeviceID) is OK. Free space: $percentFree%.</p>"
        } else {
			Add-Content $ReportFileName "<tr>"
			#Add-Content $ReportFileName "<td><center> Free space low </center></td>"
			Add-Content $ReportFileName "<tr><td>Free space low </td><td>$($disk.DeviceID)$percentFree%</td><td><b></b></td></tr>"
            
			#Add-Content $ReportFileName"<p style='color:red'>WARNING: Disk status for $($disk.DeviceID) is low. Free space: $percentFree%.</p>"
        }
		
    }

	}
	Add-Content $ReportFileName "</tr>"
	Add-Content $ReportFileName "</tr>"
        
	
	
    
	

 
#--------
# Uptime
#--------
$BootTime = [System.Management.ManagementDateTimeconverter]::ToDateTime($BootTime)
$Now = Get-Date
$span = New-TimeSpan $BootTime $Now 
	$Days	 = $span.days
	$Hours   = $span.hours
	$Minutes = $span.minutes 
	$Seconds = $span.seconds
	
	
#Remove plurals if the value = 1
	If ($Days -eq 1)
		{$Day = "1 day "}
	else
		{$Day = "$Days days "}

	If ($Hours -eq 1)
		{$Hr = "1 hr "}
	else
		{$Hr = "$Hours hrs "}

	If ($Minutes -eq 1)
		{$Min = "1 min "}
	else
		{$Min = "$Minutes mins "}

	If ($Seconds -eq 1)
		{$Sec = "1 sec"}
	else
		{$Sec = "$Seconds secs"}
$lastboot=(Get-CimInstance -ClassName Win32_OperatingSystem).LastBootUpTime
$Uptime = $Day + $Hr 
$sds= $uptime.TotalDays
Write-Host "Total Alerts: $Days"
IF ($Days -lt 1 )
{
	Add-Content $ReportFileName "<td><center>&#10060;</center></td>"
	Add-Content $ReportFileName "<td><center>$Uptime</center></td>"
	Add-Content $ReportFileName "<td><center><b>le dernier redémarrage était le : $lastboot </b></center></td>"
}else {
		Add-Content $ReportFileName "<td><center><b>&#9989;</b></center></td>"
		Add-Content $ReportFileName "<td><center>$Uptime</center></td>"
		Add-Content $ReportFileName "<td><center><b>le dernier redémarrage était le : $lastboot </b></center></td>"
	}

	
		
		
#------------------
# IIS
#------------------
$IIS = (Get-Service -Name w3svc)
if($IIS.State -eq "Running")
{
    Add-Content $ReportFileName "<tr>"
	Add-Content $ReportFileName "<td><center>&#9989</center></td>"
	Add-Content $ReportFileName "<td><center>Running</center></td>"
	
}
else
{
    Add-Content $ReportFileName "<tr>"
	Add-Content $ReportFileName "<td><center>&#10060;</center></td>"
	Add-Content $ReportFileName "<td><center>Is Not Running</center></td>"
}


Add-Content $ReportFileName "<td><center><b> </b></center></td>"
Add-Content $ReportFileName "</tr>"
}
