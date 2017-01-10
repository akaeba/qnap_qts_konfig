# QNAP QTS 4.2 - Automatischer Skriptaufruf nach Systemhochlauf


## Problemstellung

Viele nützliche Funktionen lassen sich mittels anwendungsspezifischen Skripten realisieren. Für die erfolgreiche Nutzung,
ist es unter umständen erforderlich solch ein Skript nach dem Systemhochlauf automatisiert zu starten.


## Vorraussetzung

* QNAP NAS


## _init.d_ Autorun Skript _runInitd.sh_

Einige optionale Dienste (_Optware_) nutzen das System _init.d_, im Verzeichnis _/opt/etc/init.d/_, zum initialisieren ihrer Anwendungen. Um nach jedem Systemstart die optionalen Dienste 
zu starten, sind die Skripte des Ordners _init.d_ auszuführen. Dieses Skript selbst ist ebenfalls wieder zu starten. Hierfür findet die 
[QPKG](http://techlightup.blogspot.de/2013/08/qnap-automatically-run-script-at-startup.html "QNAP: Automatically run a script at startup") basierte Methode Anwendung.

Zunächst Anlegen des QPKG Verezeichnisses und des Skriptes zum Laden sämtlicher Folgeskripte:
```sh
mkdir -p /share/MD0_DATA/.qpkg/runInitd
touch /share/MD0_DATA/.qpkg/runInitd/runInitd.sh
chmod +x /share/MD0_DATA/.qpkg/runInitd/runInitd.sh
nano /share/MD0_DATA/.qpkg/runInitd/runInitd.sh
``` 

Nach diesem [Beispiel](http://www.welzels.de/blog/projekte/qnap-mods-tricks-und-projekte/optware-init-skripte-nach-booten-starten "Optware Init-Skripte nach Booten starten") 
folgt für _runInitd.sh_:
```sh
#!/bin/sh
# Purpose: Executes all scripts in init.d directory
# SRC: http://www.welzels.de/blog/projekte/qnap-mods-tricks-und-projekte/optware-init-skripte-nach-booten-starten/
# SRC: http://techlightup.blogspot.de/2013/08/qnap-automatically-run-script-at-startup.html


# Start init.d Scripts, if directory /opt/etc/init.d exists
if [ -d /opt/etc/init.d ]; then
  # For the content in /opt/etc/init.d
  for INIT_SRC in `ls /opt/etc/init.d/ | grep -v '~' | sort`
  do
    # Do not execute RC-Scripts
    if [ ${INIT_SRC:0:1} != 'S' ]; then
      # Execute only if script is executable
      if [ -x /opt/etc/init.d/${INIT_SRC} ]; then
        # Start script
        /opt/etc/init.d/${INIT_SRC} start
      fi
    fi
  done
fi
``` 

Diese Skript startet alle nicht RC-Skripte des Ordner _init.d_. Als Beispiel würden die letzten beiden Skripte ausgeführt, das Erste hingegen nicht:
```sh
-rwxr-xr-x    1 admin    administ       140 Feb 14  2012 S70net-snmp*
-rwxr-xr-x    1 admin    administ      1032 Jan  9 12:42 sane-backends*
-rwxr-xr-x    1 admin    administ       905 Jan  9 12:40 ssh-port40*
``` 


## Ausführen _runInitd.sh_

Den Start des Skriptes _runInitd.sh_ übernimmt nun das 
[QPKG](http://techlightup.blogspot.de/2013/08/qnap-automatically-run-script-at-startup.html "QNAP: Automatically run a script at startup")
System von QNAP. 

Dazu die Datei _/etc/config/qpkg.conf_ öffnen:
```sh
nano /etc/config/qpkg.conf
``` 

Dort folgenden Eintrag am Ende hinzufügen:
```sh
[runInitd]
Name = runInitd
Version = 0.1
Author = akae
Date = 2017-01-10
Shell = /share/MD0_DATA/.qpkg/runInitd/runInitd.sh
Install_Path = /share/MD0_DATA/.qpkg/runInitd
Enable = TRUE
``` 

Damit läuft der Systemstart nun wie folgt ab:
* QPKG führt das Skript _runInitd.sh_ aus
* _runInitd.sh_ führt nacheinander die nicht RC Skripte im Ordner _/opt/etc/init.d/_ aus
