# Configure Azure ARC in een test omgeving

### ==> Disclaimer work in progress. <==

## Voorwoord

Soms is het handig om even snel een test omgeving op te zetten, Zodat je dingen kunt testen. Echter is het niet altijd mogelijk om een hele Hyper-V omgeving op te tuigen.
Hoe ga je dat snel doen? Wat heb je daar voor nodig? en kan dit in Azure? Meerdere vragen die ik nog wel eens gesteld krijg.
Dus in deze blog ga ik jullie uitleggen hoe je in Azure een Hyper-V omgeving kunt bouwen, en hoe je daarna Azure ARC erop installeert zodat je de machines kunt beheren via de Azure Portal.


## Bouw een nieuwe Server die geschikt is voor Hyper-V (by Portal and Powershell)

We gaan nu via de portal een nieuwe machine bouwen die voldoet aan de eisen om Hyper-V te mogen installeren.(heel veel machines in Azure hebben namelijk niet de juiste specs om een Hyper-V omgeving te kunnen optuigen.)

Ga naar http://portal.azure.com en log in met de credentials om een machine te kunnen bouwen.

Ga naar **Create a Resources.**

![Image](./../Images/AzureArc/Resource.JPG)

Zoek daarna op **Windows Server**, en klik daarna op **Create**

![Image](./../Images/AzureArc/WindowsServer.jpg)

Kies daarna voor **Windows Server 2019 Datacenter** en druk op **Create**


![Image](./../Images/AzureArc/WindowsServer1.jpg)


-----
Vervolgens moet je de onderstaande gegevens invullen..


~~~
Subscription = "de subscription die je gebruikt om machines te installeren"
Resource Group = Selecteer een bestaande Resource group of maak een nieuwe aan.
Virtual Machine name = HV01
Region = Kies de regio die het dichtst bij je zit. in ons geval is dat West-Europe
Availablility Options = No Infrastructure redundancy required ( Kies niet hiervoor als je een live omgeving ervan gaat maken)
Security Type = Trusted launch virtual machines
Image = Windows Server 2019 Datacenter - x64 Gen2
Size = Dit moet een V3 verie zijn om Hyper-V te kunnen installeren. Dus de Stansdard_D8s_v3

Username = Zelf in te vullen
Password = Zelf in te vullen

de rest mag default blijven
~~~
![Image](./../Images/AzureArc/WindowsServer2.jpg)

En klik op **Review & Create** 

Na de Review kun je op **Create** klikken

![Image](./../Images/AzureArc/Deploy.JPG)

Na een tijdje is de machine klaar en gaan we connecten en inloggen op de machine.

Ga naar de resource die we net hebben aangemaakt en klik op **Connect**.

![Image](./../Images/AzureArc/connect.JPG)


Mocht je een andere manier dan Native RDP hebben geconfigureerd connect via die manier anders klik op **select** bij Native RDP en er word een rdp file gedownload.


![Image](./../Images/AzureArc/RDP.jpg)


Open de RDP file en login op de nieuwe Server met de Admin credentials die je toen straks hebt opgegeven.


Mocht je dit willen uitvoeren via Powershell dan is hieronder het Powershell Script hiervoor.


~~~~
# Variables
$resourceGroupName = "YourResourceGroupName"
$vmName = "YourVMName"
$location = "East US" # Change this to the desired Azure region
$adminUsername = "YourAdminUsername"
$adminPassword = "YourAdminPassword"
$vmSize = "Standard_D8s_v3"
$imagePublisher = "MicrosoftWindowsServer"
$imageOffer = "WindowsServer"
$imageSku = "2019-Datacenter"
$nicName = "YourNICName"
$vnetName = "YourVNetName"
$subNetName = "YourSubnetName"

# Create a new resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a new virtual network
$subnet = New-AzVirtualNetworkSubnetConfig -Name $subNetName -AddressPrefix "10.0.1.0/24"
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vnetName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $subnet

# Create a public IP address
$publicIp = New-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name "YourPublicIP" -AllocationMethod Dynamic -Location $location

# Create a network security group (optional)
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroupName -Name "YourNSG" -Location $location

# Allow RDP traffic (optional)
$rdpRule = New-AzNetworkSecurityRuleConfig -Name "Allow-RDP" -Description "Allow RDP" -Access Allow -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange 3389
$nsg | Add-AzNetworkSecurityRuleConfig -NetworkSecurityRule $rdpRule
$nsg | Set-AzNetworkSecurityGroup

# Create a network interface
$nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName $resourceGroupName -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $publicIp.Id -NetworkSecurityGroupId $nsg.Id

# Create the virtual machine configuration
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize
$vmConfig = Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential (New-Object PSCredential -ArgumentList $adminUsername, (ConvertTo-SecureString -AsPlainText -Force $adminPassword))


$vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName $imagePublisher -Offer $imageOffer -Skus $imageSku -Version "latest"
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Create the VM
New-AzVM -ResourceGroupName $resourceGroupName -Location $location -VM $vmConfig

~~~~





## Failover Azure VM naar een andere regio.

Nadat we de Disaster recovery hebben ingesteld en de replicatie heeft gelopen kunnen we nu wat grafische bronnen bekijken die duidelijk aangeven wat er gebeurt en hoe de resources met elkaar verbonden zijn.
![Image](./../Images/DisasterRecovery/replication1.jpg)

Je kunt een failover uitvoeren waarmee je een productie failover uitvoert van de vm. Als de regio waar de source op staat nog beschikbaar is kun je ASR de machine laten stoppen en de laatste wijzigingen laten syncen, zodoende hebben we geen gegevens verlies.
Dit is uiteraard alleen mogelijk als de source regio nog bestaat. Anders neemt ASR de laatst bestaande restore point.
![Image](./../Images/DisasterRecovery/restorepoint.jpg)

Voorbereid zijn op disaster recovery is uitstekend. We willen er echter zeker van zijn dat het werkt als je het nodig hebt. Je wilt niet wachten tot een echte ramp toeslaat om erachter te komen of alles correct is ingesteld. Dit is waar Testfailover voor is. Testfailover is een mogelijkheid voor ons om failover van de virtuele machine naar een geïsoleerd virtueel netwerk in de doelregio te maken om de virtuele machine en toepassing te kunnen testen zonder enige impact op de productie-implementatie.‎

‎Als je meerdere vm's hebt die je in een specifieke volgorde wilt failoveren en misschien zelfs enkele extra scripts wilt laten uitvoeren om volledig failover uit een regio uit te voeren, biedt ASR ons ook [Recoveryplans](https://docs.microsoft.com/en-us/azure/site-recovery/recovery-plan-overview?WT.mc_id=itopstalk-blog-thmaure). Recoveryplans zijn idealer voor complexere scenario's dan slechts één virtuele machine.‎


## Azure Site Recovery via Powershell

Mocht je nu veel machines hebben die toegevoegd moeten worden dan zou het makkelijk kunnen zijn om dit via Powershell uit te voeren.
Hieronder een link van Microsoft learn waarbij je via Powershell ASR kunt enable en replicaties kunt starten.

~~~
https://learn.microsoft.com/en-us/azure/site-recovery/azure-to-azure-powershell
~~~
