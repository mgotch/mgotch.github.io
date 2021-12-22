#Start Maintenance
$MyServer = $env:COMPUTERNAME
$HUB = SERV
$MyServer
Set-ServerComponentState $MyServer -Component HubTransport -State Draining -Requester Maintenance
Restart-Service MSExchangeTransport
Restart-Service MSExchangeFrontEndTransport
Redirect-Message -Server $MyServer -Target $HUB
Suspend-ClusterNode $MyServer
Set-MailboxServer $MyServer -DatabaseCopyActivationDisabledAndMoveNow $True
Set-MailboxServer $MyServer -DatabaseCopyAutoActivationPolicy Blocked

Set-ServerComponentState $MyServer -Component ServerWideOffline -State Inactive -Requester Maintenance


Setup.exe /Mode:Upgrade /IAcceptExchangeServerLicenseTerms

#Stop Maintenance
$MyServer = $env:COMPUTERNAME
$MyServer
Set-ServerComponentState $MyServer -Component ServerWideOffline -State Active -Requester Maintenance
Resume-ClusterNode $MyServer

Set-MailboxServer $MyServer -DatabaseCopyActivationDisabledAndMoveNow $False
Set-MailboxServer $MyServer -DatabaseCopyAutoActivationPolicy Unrestricted

Set-ServerComponentState $MyServer -Component HubTransport -State Active -Requester Maintenance
Restart-Service MSExchangeFrontEndTransport
Restart-Service MSExchangeTransport
Get-ServerComponentState $Myserver
