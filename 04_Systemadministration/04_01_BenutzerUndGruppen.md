
`/etc/passwd`, 

`/etc/group` 

`/etc/shadow`

# 1 Benuzter

Jeden Benuzter hat ein User ID (UID)


1. _normal users_ sind Personen, die sich am System anmelden und dieses verwenden. Für sie ist jeweils ein Passwort und ein Home Directory definiert.
    
2. _system users_ entsprechen keiner Person; sie werden systemintern zur Ausführung von Systemdiensten verwendet. Beispielsweise läuft ein Web-Server-Prozess meist unter einem eigenen User.


## 1.1 `/etc/passwd`

bekannten Benutzer mit Namen, UID, Home Directory und weiteren Informationen

## 1.2 Benuzter 的密码保存在另外的一个地方  `/etc/shadow`

Passwort-Hashwerte und andere sensible Informationen (z.B. die Gültigkeitsdauer des Benutzer-Accounts)

In Linux gibt es dazu die nur für `root` lesbare Datei `/etc/shadow`; in BSD-Systemen `/etc/master.passwd`.



# 2 Gruupe 

Gruppen sind in der Datei `/etc/group` definiert.


Benuzter 属于 那个 Gruppe 在  /etc/passwd 给出 
Diese primäre Gruppe eines Benutzers ist in seinem Eintrag in /etc/passwd angegeben. Oft definiert man für normale Benutzer eine gleichnamige Gruppe als deren Default-Gruppe: der User xyz gehört zur Gruppe xyz.


# 3 Benutzer- und Gruppenverwaltung

adduser, deluser   User anlegen/löschen

addgroup, delgroup Gruppe anlegen/löschen

gpasswd   User zu Gruppen hinzufügen / daraus entfernen

chage  Account-Ablaufdatum ändern

passwd  Passwort ändern

who  Aktuell eingeloggte User anzeigen

last  Liste der letzten Logins anzeigen

# 4 Benuzter als owner des Prozesses 


一个 process 属于一个 user

一个 Process 属于一个 primären Gruppe sowie optional weiteren supplementary groups 


Mit dem Kommando` id` kann man Benutzer und Gruppen der aktuellen Shell anzeigen lassen. 
`newgrp `öffnet eine neue Shell mit einer anderen primären Gruppe (die eine der Gruppen des Benutzers sein muss).

