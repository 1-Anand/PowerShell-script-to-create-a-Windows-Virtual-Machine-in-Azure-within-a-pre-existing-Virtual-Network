# PowerShell-script-to-create-a-Windows-Virtual-Machine-in-Azure-within-a-pre-existing-Virtual-Network
PowerShell script to create a Windows Virtual Machine (VM) in Azure within a pre-existing Virtual Network (VNet), along with explanations and best practices:

# Define Variables
$resourceGroupName = "<your-resource-group-name>"
$location = "<your-azure-region>"  # e.g., "East US", "West Europe"
$vmName = "<your-vm-name>"
$vnetName = "<your-vnet-name>"
$subnetName = "<your-subnet-name>"
$adminUsername = "<your-admin-username>"
$adminPassword = "<your-admin-password>" | ConvertTo-SecureString -AsPlainText -Force

# Connect to Azure
Connect-AzAccount 

# Get Existing VNet and Subnet
$vnet = Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name $subnetName

# Create Network Interface
$nic = New-AzNetworkInterface -Name "$vmName-nic" -ResourceGroupName $resourceGroupName `
    -Location $location -SubnetId $subnet.Id

# Create VM Configuration
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize "Standard_D2s_v3"

# Set VM Operating System and Source Image
$vmConfig = Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName `
    -Credential (New-Object PSCredential -ArgumentList $adminUsername, $adminPassword) -EnableAutoUpdate
$vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" -Skus "2022-Datacenter-AzureEdition-Core" -Version "latest"

# Attach Network Interface
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Create VM
New-AzVM -ResourceGroupName $resourceGroupName -Location $location -VM $vmConfig
