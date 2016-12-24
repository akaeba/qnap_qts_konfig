# QNAP QTS 4.2 - SSH Login für beliebige Nutzer


## Problemstellung

Die Standardeinstellungen des QTS erlauben nicht die SSH Anmeldung für beliebige Nutzer. Vor dem Hintergrund des Aufbaus eines 
Backupservers, dass das _Rsync_ oder _Rsnapshoot_ Protokoll nutzt wäre ein SSH Zugang höchst wünschenswert.


## Vorraussetzung

* QNAP NAS
* Putty

## QTS (Server)
### Erweiterte Ordnerzugriffsrechte

Standardmäßig ist beim dem QTS Betriebssystem die vereinfachte Ordnerfreigabe aktiviert. Die führt während des Betriebes immer wieder zu dem 
überschreiben der mittles _chmod_ gesetzten Berechtigungen. Dies ist dahingehend ärgerlich, da der SSH Deamon bestimmte Ordnerberechtigungen
benötigt.

_QTS -> Systemsteuerung -> Privelegieneinstellungen -> Freigabe-Ordner -> Erweiterte Ordnerzugrifssrechte aktivieren_ 

<br/>
<center><img src=" /inc/advancedDirectoryPermissions.png" height="100%" width="100%" alt="Screenshot" title="Aktivierung erweiterter Ordnerzugriffsrechte" /></center>
<br/>



### Erzeugung RSA Schlüssel

Zunächst ist ein RSA Schlüsselpaar zu erzeugen. Nach _[SSH How To Set Up Authorized Keys][]_ folgt für den Befehl:
```sh
ssh-keygen -t rsa -C "My Amazing SSH Server"
``` 

Der Parameter _-C_ erlaubt die Vergabe eines Kommentars der später in der _authorized\_keys_ auftaucht. Nach dem Aufruf des Kommandos erfolgt die Ausgabe:
```sh
# Generating public/private rsa key pair. 
# Enter file in which to save the key (/Users/UserName/.ssh/id_rsa):
``` 

Zusätzlich kann der Schlüssel noch mit einem Passwort gesichert werden. Bei einfachem Bestätigen erfolgt keine Passwortvergabe:
```sh
# Enter passphrase (empty for no passphrase):
# Enter same passphrase again:
``` 

Nun sollten folgende beide Dateien vorhanden sein:
* _id\_rsa_ 		(privater Schlüssel)
* _id\_rsa.pub_		(öffentlicher Schlüssel)



### Schlüssel dem Nutzer (_myUser_) hinzufügen

Als nächstes in das _Home_ Verzeichnes des Nutzer mit zukünftigen SSH Zugang navigieren (Bei QTS in der Regel):
```sh
cd /share/homes/<myUser>
``` 

Falls noch nicht vorhanden das Verzeichnis _.ssh_ anlegen:
```sh
mkdir .ssh
``` 

Dort der _authorized\_keys_ den Inhalt des Schlüssels _id\_rsa.pub_ hinzufügen:
```sh
cat id_rsa.pub >> authorized_keys
``` 

Nun sind die Berechtigungen wie folgt anzupassen:
```sh
chmod 0711 /share/homes/<myUser>
chmod 0700 /share/homes/<myUser>/.ssh
chmod 0600 /share/homes/<myUser>/.ssh/authorized_keys
``` 

Die Nutzung des _Admin_ SSH Zuganges bdeingt das sämtliche angelegte Dateien nun dem Benutzer _Admin_ gehören. Damit der SSH login funktionieren kann,
müssen die Besitzrechte und die Gruppenzugehörigkeit bei dem über SSH zu authentifizierenden Nutzer liegen:
```sh
chown myUser /share/homes/<myUser>/.ssh
cd /share/homes/<myUser>/.ssh
chown myUser ./
chown myUser authorized_keys
``` 

Ändern der Gruppenzugehörigkeit:
```sh
chgrp myUserGrp /share/homes/<myUser>/.ssh
cd /share/homes/<myUser>/.ssh
chgrp myUserGrp ./
chgrp myUserGrp authorized_keys
``` 

Die Ausgabe von _ls -la_ sollte wie folgt aussehen:
```sh
drwx------    2 myUser myUserGrp      4096 Dec 23 21:18 ./
drwx------    4 myUser myUserGrp      4096 Dec 23 21:32 ../
-rw-------    1 myUser myUserGrp       417 Dec 23 21:18 authorized_keys
``` 


### Einrichten SSH Deamon



## Putty (Client)






## Referenzen

<!--- Internetlinks -->
[SSH How To Set Up Authorized Keys]:												https://wiki.qnap.com/wiki/SSH:_How_To_Set_Up_Authorized_Keys   														"Anleitung zur Erzeugung eines RSA-Schlüssels und Einstellung des SSH Zuganges"
[Advanced Folder Permissions on QNAP NAS]:											https://www.qnap.com/en/tutorial/con_show.php?op=showone&cid=6															"Deaktivierung der vereinfachten Dateiberechtigungen unter Linux"
[SSH Server refused our key]:														https://forum.qnap.com/viewtopic.php?t=33942																			"Server lehnt RSA Authentifizierung ab"
[SSH broken after homedir permissions and hostname change on EC2-hosted Ubuntu]:	http://serverfault.com/questions/452034/ssh-broken-after-homedir-permissions-and-hostname-change-on-ec2-hosted-ubuntu	"Konfigurationsanpassung des SSH deamons"