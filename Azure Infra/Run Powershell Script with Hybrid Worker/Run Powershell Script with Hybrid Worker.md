# Disaster Recovery

### ==> Disclaimer work in progress. <==

## Voorwoord

Als u uw on-premises omgeving wilt automatiseren, is Azure Arc Server een geweldige aanbieding voor het onboarden van Azure-beheerservices zoals Azure Monitor,Sentinel maar ook Defender en IAM.
Een van de andere voordelen die we nog hebben is het gebruik van Hybrid Workers. Hierdoor kun je RUnbooks,Logic Apps of Power Automate gebruiken om het beheer of deploy van dingen op je on-premise vm.

In deze blog laat ik je zien hoe je een Automation account maakt,Connect een server met Azure ARC,Creeren van een Hybrid Workers group en daarna het creeren en deployen van een Runbook op basis van Powershell.

![Image](./../Images/RunPowershellHybrid/Praatplaat.JPG)

## Maken van een Azure Automation Account.

Naar mijn idee kun je het Automation Account het beste in een eigen RG stoppen. Hierdoor hou je overzichtelijk wat hoort bij dit Automation account. Hou er netwerk technisch rekening mee dat dit account en alles wat er onder staat bij de omgevingen kan komen.

In dit document beschrijf ik niet hoe je een Resource group moet maken omdat ik er vanuit ga dat dit al bekend is.

Zoek in de zoekbalk naar Automation account en klik hierop.

![Image](./../Images/RunPowershellHybrid/zoekbalk.jpg)

Klik daarna op **Create**

![Image](./../Images/RunPowershellHybrid/create.jpg)

Vul in: 
~~~

Resource Group = Aangemaakte Resource group
Automation Account name= Maak een logische naam aan voor dit account
Region = Zelfde als de RG

~~~

![Image](./../Images/RunPowershellHybrid/CreateAA1.jpg)

Klik **Next**

Nu kom je in het Advanced tabblad, hier kun je aangeven van welke identity dit account gebruik moet maken in dit geval kiezen wij voor een System Managed account.

![Image](./../Images/RunPowershellHybrid/CreateAA2.jpg)

De rest hoeven we voor nu niet aan te passen. Mocht je alleen private access toe willen staan (dus de toegang tot dit account ontsluiten van toegang ergens anders vandaan) dan moet je bij networking Private Access aanklikken.

Voor nu klik op **Review + Create**
![Image](./../Images/RunPowershellHybrid/CreateAA3.jpg)



Mocht je meer info willen over een Automation Account en hoe te creeren kijk dan hier

-[Create a standalone Azure Automation account](https://learn.microsoft.com/en-us/azure/automation/automation-create-standalone-account?WT.mc_id=modinfra-0000-thmaure&tabs=azureportal)


## Connect Server in Azure via Azure Arc

Als je een server die on-premises of bij een andere cloudprovider staat, wilt verbinden met Azure via Azure Arc, ga je in de Azure Portal naar Azure Arc Center en selecteert je Azure Arc-servers. Hier kun je op de knop **Toevoegen** klikken en kun je de onboarding-wizard doorlopen.  Over een paar weken zal ik ook nog een blog online brengen hoe je AzureArc configureert voor je on-premise omgeving.

![Image](./../Images/RunPowershellHybrid/ArcOnboarding.jpg)


## Creeren van Hybrid Worker Group.

Nu kun je Hybrid Worker Group  maken en onderhouden voor het uitvoeren van taken die je on-premise ook wilt uitvoeren , wat veerkracht biedt om taken uit te voeren voor meerdere Hybrid Workers. Met op extensies based Hybrid Workers (preview) kunnen zowel Azure-machines als niet-Azure-machines (via Arc-server) worden beheerd via ARM-sjablonen en -beleid.

Als je een hybrid worker wilt maken (die Windows- en Linux-servers kan zijn) Open het Automation account en ga naar Hybrid Worker groups
![Image](./../Images/RunPowershellHybrid/HWG1.jpg

Klik hierna op **Create Hybrid Worker Group**

![Image](./../Images/RunPowershellHybrid/HWG3.jpg)

Vul in: 
~~~

Name = Logische naam voor HWG
Use Hybrid Worker Credentials = Custom
Choose Hybrid Worker Credentials = Add Credentials

~~~

![Image](./../Images/RunPowershellHybrid/HWG2.jpg)

Gebruik een service account met de juiste rechten die op de machines het script mag uitvoeren.
Ga nu het naar het tabblad  Hybrid Workers en klik op **Add Machines**

![Image](./../Images/RunPowershellHybrid/HWG4.jpg)

Je krijgt nu je machines te zien..

Selecteer de machine waarop je Hybrid Worker wil configureren en klik op **Add**

![Image](./../Images/RunPowershellHybrid/HWG5.jpg)



