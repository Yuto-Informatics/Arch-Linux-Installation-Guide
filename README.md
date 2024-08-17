Dies ist eine Zusammenfassung der Befehle für eine kompakte Installation von Arch Linux meines Installations-Videos auf Youtube.

Ausführliches Installations-Video: https://youtu.be/820Kez7wS0A

#### Tastaturlayout und Boot-Überprüfung
```bash
loadkeys de-latin1 # Deutsches Tastaturlayout
cat /sys/firmware/efi/fw_platform_size # Überprüfung, ob über UEFI gebooted worden ist (Falls nein, kommt ein Fehler zurück)
```
#### WLAN (nur wenn kein LAN-Kabel verfügbar)
```bash
iwctl
	device list # Verfügbare WLAN-Karten anzeigen
	station <wlan0> connect <SSID> # Mit WLAN verbinden
ping archlinux.org # WLAN-Verbindung prüfen
```
#### Partitionen erstellen und formatieren (Backup nicht vergessen!)
```bash
lsblk # Datenträger anzeigen, um Formatierung zu planen
gdisk /dev/sdX # Partionstabelle und Partionen anlegen
	n 1G ef00 # Bootpartition mit 1 GB Platz (Geht auch kleiner)
	n 8G # Swap mit 8 GB Platz (Faustregel: 1,5 * RAM)
	n # Root-Partition
	p # Ergebnis anzeigen
    w # Partitionstabelle schreiben (ACHTUNG: DATENVERLUST)
mkfs.fat -F 32 /dev/sdX1 # Fat32 auf Bootpartition
mkswap /dev/sdX2 # Swap formatieren
mkfs.ext4 /dev/sdX3 # ext4 auf Root-Partition

mount /dev/sdX3 /mnt # Root-Partition mounten
mkdir /mnt/boot # boot-Ordner erstellen
mkdir /mnt/boot/efi # efi-Ordner erstellen für Boot-Partition (optional genügt auch /boot)
mount /dev/sdX1 /mnt/boot/efi # Boot-Partition mounten
swapon /dev/sdX2 # Swap aktivieren
lsblk # Ergebnis anzeigen
```
#### Grundsystem installieren
```bash
reflector --verbose --latest 10 --country "Germany" --protocol https --sort rate --save /etc/pacman.d/mirrorlist # Mirrorlist sortieren
pacstrap /mnt base base-devel linux linux-firmware dhcpcd nano iwd lvm2 dbus acpid avahi cups cronie sudo less most intel-ucode amd-ucode dosfstools gptfdisk # Grundsystem unter /mnt installieren

genfstab -U /mnt >> /mnt/etc/fstab # fstab generieren
```
#### System konfigurieren
```bash
arch-chroot /mnt # Alle folgenden Befehle werden direkt auf dem neuen System ausgeführt

# Systemdienste aktivieren
systemctl enable dhcpcd cups cronie avahi-daemon
systemctl enable systemd-timesyncd # Beim Systemstart Zeit über Atomuhr abgleichen (problematisch bei dual boot)

# Alternativ zu 'echo' kann mit 'nano' die Datei bearbeitet werden
echo <myhost> > /etc/hostname # Hostname des Systems setzen (wie soll der PC heißen?)
echo LANG=de_DE.UTF-8 > /etc/locale.conf # Spracheinstellung auf Deutsch setzen
Alternativ: echo LANG=en_US.UTF-8 > /etc/locale.conf # Spracheinstellung auf Englisch setzen

nano /etc/locale.gen # Auswählen, welche Sprachen generiert werden sollen (Jeweilige Sprache darf nicht auskommentiert sein)
locale-gen # Sprachen generieren

echo KEYMAP=de-latin1 > /etc/vconsole.conf # Tastaturlayout für die Konsole setzen
echo FONT=lat9w-16 >> /etc/vconsole.conf # Schriftart für die Konsole setzen

ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime # Zeitzone setzen

# Host-Datei bearbeiten
nano /etc/hosts
    #<ip-address>	<hostname.domain.org>	<hostname>
    127.0.0.1	    localhost.localdomain	localhost
    ::1		        localhost.localdomain	localhost
```
#### Bootloader installieren
```bash
pacman -S efibootmgr grub # Tools für den Bootloader

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB # Bootloader installieren und ins UEFI eintragen
grub-mkconfig -o /boot/grub/grub.cfg # Grub-Konfigurationsdatei erstellen
```
#### Nutzer erstellen
```bash
passwd # root-Passwort setzen
useradd -m -g users -s /bin/bash <username> # Neuen Benutzer anlegen
passwd <username> # Passwort des neuen Benutzers setzen

# Root-Rechte für den neuen Benutzer aktivieren
EDITOR=nano visudo # Zeile "#wheel ALL=(ALL:ALL) ALL" darf nicht auskommentiert sein. Dadurch bekommen alle Nutzer, die in dieser Gruppe sind, root-Rechte durch sudo)
gpasswd -a <username> wheel # Neuen Benutzer zur Gruppe wheel hinzufügen 
```
#### Neues System zum ersten mal starten (kann auch erst zum Schluss gemacht werden)
```bash
reboot # System neustarten und ins neue System booten
```
#### Desktop-Umgebung installieren (in diesem Fall KDE Plasma)
```bash
pacman -S xorg-server xorg-xinit # Xorg installieren (auch wenn KDE Plasma mitlerweile standardmäßig auf Wayland setzt, würde ich Xorg immer mitinstallieren)
lspci | grep VGA # Grafikkartenmodell herausfinden
pacman -Ss xf86-video # Grafikkarten-Open-Source-Treiber listen
pacman -S xorg-drivers # Alle Open-Source-Treiber installieren
Alternativ für Nvidia-Nutzer: pacman -S nvidia-dkms # Proprietärer Treiber (empfohlen)
pacman -S xf86-input-synaptics # Treiber für Touch (ist in xorg-drivers enthalten)

localectl set-x11-keymap de pc105 deadgraveacute # Tastaturlayout für grafische Oberfläche setzen

# Desktopumgebung, Display Manager und Terminal können selbstverständlich individuell ausgesucht werden. Diese Auswahl dient nur als Beispiel.
pacman -S plasma-meta # Plasma als Desktopumgebung installieren
pacman -S sddm # sddm als Display Manager installieren
pacman -S terminator # Terminator als Terminal installieren
systemctl enable sddm # sddm-Systemdienst aktivieren
reboot # Fertiges System neustarten.
```
#### Wenn ihr alles richtig gemacht habt, dann habt ihr jetzt ein funktionierendes Arch Linux installiert :)
#### Kontaktiert mich gerne, wenn irgendetwas outdated sein sollte oder ihr andere Verbesserungsvorschläge oder Fragen habt (am besten über Youtube oder E-Mail)
Youtube: https://www.youtube.com/@Yuto-Informatics  
E-Mail-Adresse: questions@yuto-informatics.de
