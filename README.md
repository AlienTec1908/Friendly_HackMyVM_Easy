# Friendly - HackMyVM (Easy) - Writeup

![Friendly Icon](Friendly.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Friendly" (Schwierigkeitsgrad: Easy).

## Metadaten

*   **Maschine:** Friendly (HackMyVM - Easy)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Friendly](https://hackmyvm.eu/machines/machine.php?vm=Friendly)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 2023-04-04
*   **Original Writeup:** [https://alientec1908.github.io/Friendly_HackMyVM_Easy/](https://alientec1908.github.io/Friendly_HackMyVM_Easy/)

## Zusammenfassung des Angriffspfads

Der initiale Nmap-Scan identifizierte einen FTP-Dienst (Port 21, ProFTPD) mit erlaubtem anonymen Login und einen Apache-Webserver (Port 80). Es wurde festgestellt, dass das Wurzelverzeichnis des FTP-Servers beschreibbar war und mit dem Web-Root übereinstimmte (da eine `index.html` sowohl über FTP als auch HTTP erreichbar war).

Eine einfache PHP-Webshell (`test.php`) wurde vorbereitet und über den anonymen FTP-Zugang (`lftp`) in das Web-Root hochgeladen. Der Aufruf dieser Webshell über HTTP (`http://freund.hmv/test.php?cmd=...`) ermöglichte Remote Code Execution (RCE). Mittels einer Bash-Reverse-Shell-Payload wurde eine Verbindung zu einem `nc`-Listener aufgebaut und initialer Zugriff als Benutzer `www-data` erlangt. Die Shell wurde anschließend stabilisiert (`python pty`, `stty`).

Als `www-data` wurde das Home-Verzeichnis des Benutzers `RiJaba1` gefunden und die User-Flag aus `user.txt` gelesen. Die Überprüfung der `sudo`-Rechte (`sudo -l`) ergab, dass `www-data` den Befehl `/usr/bin/vim` als `root` ohne Passwort ausführen durfte (`(ALL : ALL) NOPASSWD: /usr/bin/vim`).

Diese unsichere `sudo`-Regel wurde ausgenutzt, indem `sudo vim` gestartet und innerhalb des Editors der Befehl `:!/bin/sh` ausgeführt wurde. Dies gewährte eine Root-Shell. Die Root-Flag wurde anschließend nicht im Standardverzeichnis `/root/root.txt` gefunden (dort war eine "Fake"-Flag), sondern mittels `find` in `/var/log/apache2/root.txt` lokalisiert und ausgelesen.

## Verwendete Tools (Auswahl)

*   `nmap`
*   `echo`
*   `vi` / Editor
*   `mv`
*   `lftp`
*   Web Browser / `curl`
*   `cat`
*   `grep`
*   `nc` (netcat)
*   `which`
*   `python3` (`pty` module)
*   `stty`
*   `cd`
*   `ls`
*   `sudo`
*   `vim`
*   `find`
*   `id`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** `nmap` Scan -> Port 21 (FTP, ProFTPD, Anonymous Login, Writable Root), Port 80 (HTTP, Apache). Hostname `freund.hmv` ermittelt.
2.  **FTP Enumeration & Upload:** Mit `lftp` anonym verbinden. Bestätigen, dass Upload ins FTP-Root (Web-Root) möglich ist. PHP-Webshell (`test.php` mit ``) vorbereiten. Webshell via `put test.php` hochladen.
3.  **Initial Access (RCE):** `nc`-Listener starten. Webshell über `http://freund.hmv/test.php?cmd=...bash_reverse_shell_payload...` triggern -> Shell als `www-data`.
4.  **Shell Stabilization:** `python3 -c 'import pty;...'`, `export TERM=xterm`, `stty raw -echo; fg`.
5.  **User Flag:** Navigieren nach `/home/RiJaba1`, `cat user.txt`.
6.  **Privilege Escalation Vector:** `sudo -l` für `www-data` -> `(ALL : ALL) NOPASSWD: /usr/bin/vim`.
7.  **Root Access:** `sudo vim` ausführen. Innerhalb von Vim `:!/bin/sh` eingeben -> Root-Shell.
8.  **Root Flag Discovery:** `cat /root/root.txt` (Fake Flag). `find / -name root.txt` -> Echte Flag in `/var/log/apache2/root.txt`. `cat /var/log/apache2/root.txt`.

## Flags

*   **User Flag (`/home/RiJaba1/user.txt`):** `b8cff8c9008e1c98a1f2937b4475acd6`
*   **Root Flag (`/var/log/apache2/root.txt`):** `66b5c58f3e83aff307441714d3e28d2f`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
