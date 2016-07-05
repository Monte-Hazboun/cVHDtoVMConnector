function Get-TargetResource {
	[CmdletBinding()]
	[OutputType([System.Collections.Hashtable])]
	param (
        
        [parameter(Mandatory = $true)]
        [String]$VMName,
        
        [Parameter(Mandatory = $true)]
        [ValidateSet("IDE","SCSI")]
        [String]$ControllerType,

        [Parameter(Mandatory = $true)]
        [uint32]$ControllerNumber,

        [Parameter(Mandatory = $true)]
        [Uint32]$ControllerLocation,
        
        [Parameter(Mandatory = $true)]
        [String]$VHDPath
                    
	)
        

    #$VHD = Get-VMHardDiskDrive -VMName $VMname -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation
    $VHD = Get-vm | Get-VMHardDiskDrive | where {$_.path -eq $VHDpath}

    @{
		VMName               = $VMName
		VhdPath              = $VHD.path
        ControllerType       = $VHD.ControllerType
        ControllerNumber     = $VHD.ControllerNumber
        ControllerLocation   = $VHD.ControllerLocation
        Ensure               = If ($vhd) {"Present"} else {"Absent"}
	}
    
}

function Set-TargetResource {
	[CmdletBinding()]
	param (
        [parameter(Mandatory)]
        [String]$VMName,

        [Parameter(Mandatory)]
        [String]$VHDPath,

        [Parameter(Mandatory)]
        [ValidateSet("IDE","SCSI")]
        [String]$ControllerType,

        [Parameter(Mandatory)]
        [uint32]$ControllerNumber,

        [Parameter(Mandatory)]
        [uint32]$ControllerLocation,
        
		[ValidateSet("Present","Absent")]
		[String]$Ensure     
	)
    
    $ConfirmPreference = "None"
    If (!(Test-Path $VHDPath)) {
        throw "Could not find VHD at $VHDPath . Please ensure it exists in the right location"
    }

    $VM = Get-VM $VMName

    If (-not $VM) {
        throw "Could not find $VMname on Hyper-V host"
    }

    $VMState = $VM.State
    If ($VMState -eq "Running") {
        Stop-VM -VM $VM -confirm:$false 
    }
    
    Switch ($ensure) {
        'Present' {
            $VHD = Get-VMHardDiskDrive -VMName $VMname -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation
            If (-not $VHD) {
                Add-VMHardDiskDrive -VMName $VMName -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation -Path $VHDPath    
            } elseif ($VHD.Path -ne $VHDPath) {
                $VHD | Remove-VMHardDiskDrive
                Add-VMHardDiskDrive -VMName $VMName -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation -Path $VHDPath
            }    
        }
        'Absent' {
            $VHD = Get-VMHardDiskDrive -VMName $VMname -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation
            If ($VHD.Path -eq $VHDPAth) {
                $VHD | Remove-VMHardDiskDrive
            }
        }
    }
    
    If ($VMState -eq "Running") {
        Start-VM -VM $VM -confirm:$false
    }   
}

function Test-TargetResource
{
	[CmdletBinding()]
	[OutputType([System.Boolean])]
	param (
        [parameter(Mandatory = $true)]
        [String]$VMName,

        [Parameter(Mandatory = $true)]
        [String]$VHDPath,

        [Parameter(Mandatory = $true)]
        [ValidateSet("IDE","SCSI")]
        [String]$ControllerType,

        [Parameter(Mandatory = $true)]
        [uint32]$ControllerNumber,

        [Parameter(Mandatory = $true)]
        [uint32]$ControllerLocation,

		[ValidateSet("Present","Absent")]
		[String]$Ensure    

	)
    $ConfirmPreference = "None"
    Switch ($ensure) {
        'Present' {
            $VHD = Get-VMHardDiskDrive -VMName $VMname -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation
            If (-not $VHD) {
                return $false
            } elseif ($VHD.Path -ne $VHDPath) {
                return $false
            } else {
                return $true
            }
        }
        'Absent' {
             $VHD = Get-VMHardDiskDrive -VMName $VMname -ControllerType $ControllerType -ControllerNumber $ControllerNumber -ControllerLocation $ControllerLocation
            If (-not $VHD) {
                return $true
            } elseif ($VHD.Path -ne $VHDPath) {
                return $true
            } else {
                return $false
            }           
        }   
    } 
}


Export-ModuleMember -Function *-TargetResource
