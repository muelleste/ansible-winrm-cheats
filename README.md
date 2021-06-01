# Ansible WinRM Cheat Sheet

## Ansible-Developer Server

Before starting with all windows componentes, there a few requirments on our ansible server itself.
This is a additional installation as normal.

> sudo apt install python-dev libkrb5-dev krb5-user

While installation progress, there are a few questions, if you don't answer that correctly you can correct it manually.

*Ubuntu:*
> admin $ vim /etc/krb5.conf

```
[libdefaults]
        default_realm = LAB.EDU

# The following krb5.conf variables are only for MIT Kerberos.
        kdc_timesync = 1
        ccache_type = 4
        forwardable = true
        proxiable = true
[realms]
        LAB.EDU = {
                kdc = <DC1.DOMAINCONTROLLER.DE>
                admin_server = <DC1.DOMAINCONTROLLER.DE>
        }

[domain_realm]
        .lab.edu = LAB.EDU

```

## Windows Management

### WinRM - HTTP Connection

_Advice:_ HTTP is only useful in a lab environment, when no CA is there. Please do not implement this in production!

#### Firewall

*Check Windows Firewall, if WinRM is enabled and ports are open:*

```powershell
Get-NetFirewallRule | Where-Object Name -eq "WINRM-HTTP-In-TCP"
```

*Create a Firewall Rule:*

```powershell
if (-Not(Get-NetFirewallRule | Where-Object Name -eq "WINRM-HTTP-In-TCP")) {
    New-NetFirewallRule -Name "WINRM-HTTP-In-TCP" `
        -DisplayName "Windows Remote Management (HTTP-In)" `
        -Description "Inbound rule for Windows Remote Management via WS-Management. [TCP 5985]" `
        -Group "Windows Remote Management" `
        -Program "System" `
        -Protocol TCP `
        -LocalPort "5985" `
        -Action Allow `
        -Profile Domain, Private
}

```

#### Service-Check

*Check Windows Service:*

```powershell
Get-Service -Name WinRM
```

*Start Windows Service:*

```powershell
Start-Service -Name WinRM
```

#### Permssion

If you don't use the local administrator for administrate, you have to give your User needed rights.

### WinRM Service

While using http protocol unencrypted protocol must be allowed in the service.

*Admin-Powershellsession:*
> Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value true

## WinRM Commandlets

**Create WinRM Listener Quickconfig:**
> winrm quickconfig

**Get WinRM Listeners:** 
> winrm get winrm/config/Listener?Address=*+Transport=HTTPS

**DELETE des WinRM Listeners:**
> winrm delete winrm/config/Listener?Address=*+Transport=HTTPS 
