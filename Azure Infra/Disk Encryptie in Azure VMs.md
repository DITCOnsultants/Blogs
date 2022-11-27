# Disk Encryptie in Azure VMs

### ==> Disclaimer work in progress <==

## Voorwoord
In het verleden vereiste het proces van het encrypten van een virtuele machine in Microsoft Azure een AD App account om het werk uit te kunnen voeren.
Tegenwoordig hebben we een eenvoudigere manier om onze VM’s in Microsoft Azure te encrypten en deze nieuwe methode is geïntegreerd in de Azure Portal.
In dit artikel gaan we de basisprincipes behandelen om encryptie in te schakelen met behulp van de nieuwe methode, inclusief het maken van een Azure Key Vault.
We nemen ook de stappen door die kunnen worden uitgevoerd in de Azure Portal en de Powershell functies om het encryptieproces te ondersteunen.

![Image](./Images/Encryptie/Diagram.png)

## Inrichten van de Key Vault
De eerste stap is het creëren van een Key Vault.
Log in op de Azure portal en klik op **Create a resource**.

![Image](./Images/Encryptie/CreateResource.png)

Vul in **Key vault** en klik op **Create**.

![Image](./Images/Encryptie/CreateKeyvault.png)

Een nieuwe wizard zal naar voren komen. En hier kunnen we alle basisinstellingen configureren die we nodig hebben om de nieuwe Key Vault met nieuwe schijf encryption te gaan gebruiken.

**Vul in; Naam Region Resource**

![Image](./Images/Encryptie/KeyvaultSettings.png)

Klik op **Next** en we gaan naar Access Policy. 
Zorg ervoor dat het vinkje staat bij **Azure Disk encryption for volume encryption**. 

![Image](./Images/Encryptie/ResourceAccess.png)

Klik op **review+ create** om de inrichting van de KeyVault te starten.

**Notitie:** *Als je een hub- en spoke- topologie gebruikt, is het gebruik van een Key Vault op je shared infrastructure bij de hub de juiste manier.
Op deze manier maken al je encrypted VM’s gebruik van de centralized keys om hun schijven op te slaan en te encrypten.*


Mocht je het vinkje bij Azure Disk encryption for volume encryption vergeten zijn. Kun je altijd deze nog aanzetten bij Access Policies in de Key Vault properties.

Na het maken van de Key Vault is onze volgende stap het maken van onze eerste set van Keys. Open de net aangemaakte Key Vault en klik onder objects op **Keys**.
Klik op **Generate/Import**.


## Encrypt een VM via de Azure Portal.

## Schijven en encryptie scenario’s beheren.

## Disk Encryptie gebruiken met PowerShell

Met al die informatie zou de PowerShell cmdlet (**Set-AzVMDiskEncryptionExentsion** ) vergelijkbaar zijn met de onderstaande cmdlet.

```
Set-AzVmDiskEncryptionExtension -ResourceGroupName <ResourceGroupName> -VMName <VMName> -DiskEncryptionKeyVaultId  <Key Vault Resource ID> -DiskEncryptionKeyVaultUrl <Key Vault URL> -KeyEncryptionKeyVaultId <Key Vault Resource ID> -KeyEncryptionKeyUrl <Key URL> -VolumeType "All"
```


Meer informatie over schijf encryptie vind je [hier](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/disk-encryption-faq#can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption)


