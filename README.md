Deutsch | [English](README_en.md) | [HowTo's: Inhaltsverzeichnis](https://github.com/toafez/Tutorials)

# Analysieren und Bereinigen einer überfüllten Systempartition auf einem Synology NAS
[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Ftoafez%2FHowTo_Syno.SystemPartition%2Ftree%2Fmain&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

## Worum geht es
Die folgende Anleitung beschreibt, wie man eine überfüllte **Systempartition** auf einem **Synology NAS** analysieren und von Ballast befreien kann, um den ordnungsgemäßen Betrieb des DSM wiederherzustellen.

## Einleitung
Physikalische Datenspeicher wie Festplatten (HDD), Solid State Drives (SSD), NVM Express (NVMe) usw., die allgemein als Laufwerke bezeichnet werden, verfügen in der Regel über eine oder mehrere Partitionen für die Bereitstellung und Speicherung von Daten. 

Bei einem Synology NAS sind alle internen Laufwerke grundsätzlich in drei Partitionen unterteilt und zwar in eine **Systempartition**, eine **SWAP-Partition** und eine **Datenpartition**. Die Systempartition und die SWAP-Partition werden dabei als RAID 1 auf jedem internen Laufwerk gespiegelt, während die Datenpartition aus Speicherpools und Volumes besteht, auf denen alle Benutzerdaten und installierte Pakete gespeichert werden. Weitere Informationen siehe ![Was sind Laufwerkspartitionen?](https://kb.synology.com/de-de/DSM/tutorial/What_are_drive_partitions)

## Die Systempartition
Die **Systempartition** verfügt in der Regel über eine **Speicherkapazität von 2,3 GiByte (bis DSM 6) oder 7,9 GiByte (ab DSM 7)** und enthält das eigentliche Betriebssystem, also den DiskStation Manager, alle Benutzer-, System- und Netzwerkeinstellungen sowie die Systemprotokolle. Unmittelbar nach der Erst- oder Neuinstallation sind je nach Gesamtspeicherkapazität **ca. 25 bis ca. 65 Prozent des verfügbaren Speicherplatzes belegt**. Der verbleibende freie Speicherplatz ist in der Regel völlig ausreichend, um einen störungsfreien Betrieb des DSM zu gewährleisten. 

## Auslöser einer überfüllten Systempartition
Manchmal landen jedoch Daten auf der Systempartition, die dort nicht hingehören. Dazu zählen sowohl bewusste als auch unbewusste Handlungen, wenn z.B. Benutzerdaten und/oder Programmeinstellungen im Home-Verzeichnis des Superusers root abgelegt werden, das sich ebenfalls auf der Systempartition befindet, oder wenn z.B. fehlgeleitete Sicherungsaufträge oder Downloads dort gespeichert werden. Auch nicht sauber getrennte USB- oder SATA-Datenträger, unvorhersehbare Systemabstürze oder ein plötzlicher Stromausfall können dazu führen, dass Datenfragmente auf der Systempartition landen.

Bis zu einem gewissen Punkt hat das keinen Einfluss auf die Systemstabilität des DSM. Sobald jedoch der belegte Speicherplatz eine gewisse kritische Masse erreicht, die irgendwo zwischen 80 und 100% der verfügbaren Speicherkapazität liegt, beginnt der DSM langsam zu straucheln und es treten erste Störungen auf. Man kann sich z.B. nicht mehr am DSM anmelden, es können keine System- oder Paketupdates mehr durchgeführt werden oder Pakete können nicht mehr ausgeführt werden. Häufig erhält man dann Fehlermeldungen wie diese:

_**Dieser Vorgang kann nicht ausgeführt werden. Möglicherweise ist die Netzwerkverbindung instabil, oder das System ausgelastet. Bitte versuchen Sie es später erneut.**_

## Analysieren der Systempartition 
Da es leider nicht immer offensichtlich ist, was die Ursache für eine überfüllte Systempartition ist, kann ein Teil der Lösung solcher Probleme bereits darin liegen, nachvollziehen zu können, welche Aktionen zuvor durchgeführt wurden, welche Aufgaben erledigt und welche Änderungen oder Anpassungen vorgenommen wurden. Dies erleichtert bereits die Suche nach möglichen Übeltätern innerhalb der Systempartition.

Ein klassisches Beispiel wäre ein überfülltes Homeverzeichnis des Superuser root, daher sollte man auf jeden Fall einen Blick in dieses Verteichnis werfen, sollte einem der belegte Speicherplatz unverhältnismäßig hoch vorkommen. Da sich auf der Systempartition keine Speicherpools oder Volumes befinden, sollte ein eventuell vorhandenes Verzeichnis mit der Bezeichnung /volumeUSB... ebenfalls aufhorchen lassen. Auch Verzeichnisse oder Dateien mit bekannten Paketnamen oder auffälligen Dateinamen von zuvor heruntergeladenen Programmen oder Archiven sollten kritisch hinterfragt werden. 

Wie so oft gibt es auch hier keine allgemeingültige Lösung, sondern nur Tipps und Hinweise, die die Suche erleichtern können. Der erste Schritt sollte daher immer sein, sich den Inhalt der Systempartition anzeigen zu lassen und sich dann von oben nach unten, beginnend mit dem Verzeichnis, das den meisten Speicherplatz belegt, durch den Verzeichnisdschungel zu arbeiten, bis man den oder die Übeltäter gefunden hat.

Für alle weiteren Schritte ist es notwendig, sich über ein Terminalprogramm per SSH als root auf der Konsole des Synology NAS einzuloggen. Wie das geht, beschreibt Synology u.a. in der Anleitung ![Wie kann ich mich über SSH mit Root-Berechtigung bei DSM/SRM anmelden?](https://kb.synology.com/de-de/DSM/tutorial/How_to_login_to_DSM_with_root_permission_via_SSH_Telnet) 

#### _Hinweis: Text in Großbuchstaben innerhalb eckiger Klammern dient als Platzhalter und muss durch eigene Angaben ersetzt werden, während gemischter Text aus Groß- und Kleinbuchstaben, Ziffern und Sonderzeichen innerhalb eckiger Klammern optional verwendet werden kann. In jedem Fall müssen die eckigen Klammern beim Ersetzen von Platzhaltern oder bei der Verwendung einer Option entfernt werden._

## Das Programm `df` (**d**isk **f**ree)
Das Programm `df` zeigt die Größe, den belegten und den freien Speicherplatz aller eingehängter Partitionen an. Um die angezeigte Speicherkapazität in einem für den Menschen leichter lesbaren Format auszugeben, wird die Option `-h` oder `-H` an den `df` Befehl angehängt, je nachdem, ob die Ausgabe in metrischer oder binärer Form erfolgen soll.

- _**Syntax:** df [OPTION] [DATEI]_

   **_Verwendete Optionen für das Programm df (weitere Optionen ![siehe Manpage](https://manpages.debian.org/bookworm/manpages-de/df.1.de.html))_**

   - **-h, --human-readable** (Gibt die Speicherkapazität in einem binären Format in Potenzen von 1024 an, das für den Menschen leichter zu lesen ist.)
      ```
      "K" ≙ "Kibibyte (KiB)" ≙ 1,024¹ ≙ 1,024 Bytes
      "M" ≙ "Mebibyte (MiB)" ≙ 1,024² ≙ 1,048,576 Bytes
      "G" ≙ "Gibibyte (GiB)" ≙ 1,024³ ≙ 1,073,741,824 Bytes
      "T" ≙ "Tebibyte (TiB)" ≙ 1,024⁴ ≙ 1,099,511,627,776 Bytes
      "P" ≙ "Pebibyte (PiB)" ≙ 1,024⁵ ≙ 1,125,899,906,842,620 Bytes
      ```
   - **-H, --si** (Gibt die Speicherkapazität in einem metrischen Format in Potenzen von 1000 an, das für den Menschen leichter zu lesen ist.)
      ```
      "K" ≙ "Kilobyte (kB)" ≙ 1,000¹ ≙ 1,000 Bytes
      "M" ≙ "Megabyte (MB)" ≙ 1,000² ≙ 1,000,000 Bytes
      "G" ≙ "Gigabyte (GB)" ≙ 1,000³ ≙ 1,000,000,000 Bytes
      "T" ≙ "Terabyte (TB)" ≙ 1,000⁴ ≙ 1,000,000,000,000 Bytes
      "P" ≙ "Petabyte (PB)" ≙ 1,000⁵ ≙ 1,000,000,000,000,000 Bytes
      ```
  #### Führe den folgenden Befehl aus, um dir den Status der Systempartition anzeigen zu lassen.
  ```
  df -h
  ```
     So würde die Ausgabe einer "gesunden" Systempartition aussehen...

  ```
  root@SynologyNAS:~# df -h
  Filesystem              Size  Used Avail Use% Mounted on
  /dev/md0                2.3G  1.5G  773M  66% /
  devtmpfs                3.8G     0  3.8G   0% /dev
  tmpfs                   3.9G  240K  3.9G   1% /dev/shm
  tmpfs                   3.9G   46M  3.8G   2% /run
  tmpfs                   3.9G     0  3.9G   0% /sys/fs/cgroup
  tmpfs                   3.9G   30M  3.8G   1% /tmp
  /dev/loop0               27M  770K   24M   4% /tmp/SynologyAuthService
  Weitere Speicherpools oder Volumes...
  ...
  ..
  .
  ```
   Interessant ist hier vor allem die erste Zeile, die die Gesamtspeicherkapazität, den belegten und freien Speicherplatz sowie den belegten Speicherplatz in Prozent der Systempartition (Mounted on `/`) anzeigt. Der vorangestellte Pfadname der Systempartition (/dev/md0) kann dabei durchaus variieren.

  ```
  /dev/md0                2.3G  1.5G  773M  66% /
  ```
    Mit der Einführung des DiskStation Manager 7 (kurz DSM) wurde die Systempartition bei einer Neuinstallation des DSM ab einem bestimmten Zeitpunkt auf 7,9 G (GiByte) vergrößert. Das Ergebnis einer "gesunden" Systempartition würde dann in etwa so aussehen.

  ```
  root@SynologyNAS:~# df -h
  Filesystem         Size  Used Avail Use% Mounted on
  /dev/md0           7.9G  2.0G  5.8G  26% /
  devtmpfs           8.7G     0  8.7G   0% /dev
  tmpfs              8.8G  244K  8.8G   1% /dev/shm
  tmpfs              8.8G   38M  8.7G   1% /run
  tmpfs              8.8G     0  8.8G   0% /sys/fs/cgroup
  tmpfs              8.8G   29M  8.7G   1% /tmp
  /dev/loop0          27M  767K   24M   4% /tmp/SynologyAuthService
  Weitere Speicherpools oder Volumes...
  ...
  ..
  .
  ```

## Das Programm `du` (**d**isk **u**sage) 
Das Programm `du` zeigt den belegten Speicherplatz von Dateien, oder bei Angabe eines Verzeichnisses, rekursiv den belegten Speicherplatz aller darin enthaltenen Dateien an. Auch hier stehen weitere Optionen zur Verfügung, um die Ausgabe an die jeweiligen Bedürfnisse anzupassen und um die Lesbarkeit zu verbessern.

- _**Syntax:** du [OPTION] [DATEI]_

   _**Verwendete Optionen für das Programm `du` (weitere Optionen ![siehe Manpage](https://manpages.debian.org/bookworm/manpages-de/du.1.de.html))**_
   - **-x, --one-file-system** (Verzeichnisse auf anderen Dateisystemen werden übersprungen.)
   - **-h, --human-readable** (Gibt die Speicherkapazität in einem binären Format in Potenzen von 1024 an, das für den Menschen leichter zu lesen ist.)
   - **--si** (Gibt die Speicherkapazität in einem metrischen Format in Potenzen von 1000 an, das für den Menschen leichter zu lesen ist.)
   - **-d, --max-depth=N** (Gibt die Gesamtgröße eines Verzeichnisses bis zur Ebene N des übergebenen Arguments zurück.)

  #### Der Befehl lautet (bitte noch nicht ausführen, sondern erst weiterlesen)  
  ```
  du -x -h -d 1 /
  ```
  
## Das Programm `sort`
Vor der eigentlichen Ausführung des `du` Befehs sollte zur besseren Übersicht die Speicherbelegung der ausgegebenen Verzeichnisse nach ihrer Größe absteigend sortiert werden. Dies wird durch das Programm `sort` erreicht.

- _**Syntax:** sort [OPTION] [DATEI]_

    _**Verwendete Optionen für das Programm sort (weitere Optionen ![siehe Manpage](https://manpages.debian.org/bookworm/manpages-de/sort.1.de.html))**_
    - **-h, --human-numeric-sort** (Gibt die Dateigröße in einem besser lesbaren Format aus)
    - **-r, --reverse** (Sortierung in umgekehrter Reihenfolge)

   #### Der Befehl lautet (bitte noch nicht ausführen, sondern erst weiterlesen)  
   ```
   sort -r -h
   ```

## Kombinierte Ausgabe von `du` und `sort` 
Die Ausgabe des oben beschriebenen Befehls `du -x -h -d 1 /` kann nun über eine sogenannte Pipe `|` an den Befehl `sort -r -h` übergeben werden, um die Speicherbelegung absteigend zu sortieren. Die Befehle werden also miteinander kombiniert und zusammen in einer Zeile ausgeführt.

- _**Syntax:** du [OPTION] [DATEI] **|** sort [OPTION] [DATEI]_

  Führe den folgenden Befehl aus, um den belegten Speicherplatz aller Verzeichnisse direkt unterhalb des Wurzelverzeichnisses der Systempartition rekursiv und nach Größe sortiert anzuzeigen.
  ```
  du -x -h -d 1 / | sort -r -h
  ```
    So könnte z.B. der Verzeichnisinhalt einer bereits leicht angeschlagenen Systempartition mit einer maximalen Speicherkapazität von 2.3 GiByte aussehen...

      ```
      root@SynologyNAS:~# du -x -h -d 1 / | sort -r -h
      1.9G    /
      1.1G    /usr
      605M    /root
      161M    /var
      43M     /.syno
      11M     /var.defaults
      3.7M    /etc
      2.5M    /etc.defaults
      1.3M    /.log.junior
      28K     /.old_patch_info
      20K     /volumeUSB1
      16K     /.system_info
      16K     /opt
      4.0K    /mnt
      4.0K    /lost+found
      4.0K    /initrd
      ```

  Hier fallen zwei Dinge auf. Zum einen ist das Homeverzeichnis `/root` des Superusers root mit einem Speicherplatzbedarf von 605M (MiByte) deutlich größer als erwartet. Zum anderen wird das Verzeichnis /volumeUSB1 angezeigt. Obwohl es scheinbar nur 20K (KiByte) des verfügbaren Speicherplatzes belegt, gehört dieses Verzeichnis nicht hierher, da sich, wie eingangs erwähnt, alle Volumes auf der Datenpartition befinden. Eine fehlerhafte Anzeige kann ausgeschlossen werden, da die Option `-x` des Befehls `du` besagt, dass Verzeichnisse auf anderen Partitionen übersprungen werden.

## Die kombinierte Ausgabe weiter eingrenzen
Zunächst wollen wir uns dem Verzeichnis `/root` zuwenden, da der hier belegte Speicherplatz vermutlich der Auslöser für ein bereits strauchelnden DSM sein kann. Um mehr über den Inhalt des Verzeichnisses zu erfahren, passen wir die bereits oben verwendete Befehlskette etwas an, und geben dem Befehl `du` das Verzeichnis `/root` mit.

```
du -x -h -d 1 /root | sort -r -h
```

  Das Ergebnis sieht in diesem Beispiel folgendermaßen aus.

  ```
  root@SynologyNAS:~# du -x -h -d 1 /root | sort -r -h
  605M    /root/.vscode-server
  605M    /root
  176K    /root/.dotnet
  48K     /root/.ssh
  28K     /root/.cache
  24K     /root/.local
  16K     /root/.config
  ```

Um noch tiefer in die Verzeichnisstruktur einzudringen, könnte man die Befehlskette wiederum um das Verzeichnis `/root/.vscode-server` erweitern und erneut ausführen, bis man irgendwann lokalisiert hat, wo sich der Übeltäter verbirgt. Da mir in diesem Fall bereits der Name des Verzeichnisses verrät, dass es sich um gespeicherte Anwendungsdaten des Programms Visual Studio Code handelt, das sich über eine SSH-Verbindung als root auf meiner Synology NAS anmeldet, beende ich die Suche an dieser Stelle und werde den Ordner im nächsten Schritt komplett löschen, um meinen DSM wieder funktionsfähig zu bekommen. Alternativ könnte ich einzelne Dateien  oder das gesamte Verzeichnis auch auf ein internes oder externes Volume verschieben, wenn ich nicht sicher bin, ob ich die Daten noch einmal benötige.

## Das Programm `mv` (move)
Das Programm `mv` wird normalerweise verwendet, um Dateien oder ganze Verzeichnisse zu verschieben. Mit `mv` können aber auch Dateien und Verzeichnisse umbenannt werden, worauf hier nicht weiter eingegangen wird. Außerdem werden in diesem Beispiel keine weiteren Optionen benötigt, so dass an dieser Stelle nur auf die entsprechende ![Manpage](https://manpages.debian.org/bookworm/manpages-de/mv.1.de.html) von `mv` verwiesen wird. 

- _**Syntax:** mv [QUELLE] [ZIEL]_

  Im Folgenden wird der Befehl `mv` verwendet, um das zuletzt ermittelte Verzeichnis `/root/.vscode-server` auf ein internes Volume des DSM nach `/volume1/Backup/Systempartitionsdaten` zu verschieben. Die hier verwendeten Verzeichnisse dienen nur als Beispiel und müssen in jedem Fall durch eigene ersetzt werden.

   ```
   mv /root/.vscode-server /volume1/Backup/Systempartitionsdaten/
   ```
   
  **Wichtiger Hinweis:** Es muss darauf geachtet werden, dass das Zielverzeichnis mit einem `/` endet, sonst versucht `mv` beim Verschieben das Verzeichnis `/volume1/Backup/Systempartitionsdaten` in `/volume1/Backup/.vscode-server` umzubenennen, anstatt das Quellverzeichnis `/root/.vscode-server` in das Verzeichnis `/volume1/Backup/Systempartitionsdaten` zu verschieben.

## Das Programm rm (remove)
Das Programm `rm` löscht auf der Konsole Dateien oder ganze Verzeichnisse ohne den Umweg über den Papierkorb. **Gelöschte Daten können nicht wiederhergestellt werden.** Aus diesem Grund sollten alle Löschbefehle gut überlegt sein, vor allem, wenn z.B. die Option `-r` für rekursives Löschen (löscht also ganze Verzeichnisstrukturen) mit der Option `-f` kombiniert wird, um den Befehl ohne Rückfrage ausführen zu lassen. Auf der anderen Seite kann eine permanente Rückfrage auf Dauer auch recht nervig sein, weshalb diese Option trotzdem sehr gerne verwendet wird.  

- _**Syntax:** rm [OPTION] [DATEI ODER VERZEICHNIS]_

     _**Verwendete Optionen für das Programm rm (weitere Optionen ![siehe Manpage](https://manpages.debian.org/bookworm/manpages-de/rm.1.de.html))**_
     - **-r, --recursive** (Verzeichnisse und deren Inhalte rekursiv löschen)
     - **-f , --force** (Keine Nachfrage beim Löschen) 

  Obwohl der Befehl `rm` immer mit Vorsicht zu genießen ist, kommt man oft nicht umhin, ihn zu benutzen. In diesem Beispiel wird das Unterverzeichnis `.vscode-server` rekursiv aus dem Verzeichnis `/root` gelöscht, d.h. mit allem, was sich in diesem Verzeichnis befindet.

   ```
   rm /root/.vscode-server
   ```

## Abschließende Worte
Theoretisch sollte die Systempartition dem DSM nun wieder genügend Speicherplatz zur Verfügung stellen, um einen reibungslosen Betrieb zu ermöglichen. Verbleibt in diesem Beispiel noch das Verzeichnis /volumeUSB1, das auf die gleiche Weise analysiert, verschoben oder gelöscht werden kann, wie es zuvor am Beispiel des Verzeichnisses `/root/.vscode-server` beschrieben wurde. 
