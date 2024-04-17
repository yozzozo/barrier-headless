# barrier-headless
This is a write up on how to run `barrier` (the synergy-core fork) on a headless raspberry pi. It's based on the [work](https://github.com/hishamk/headlesssynergysetup) of [hishamk](https://github.com/hishamk/), describing how to do the same thing for the original synergy.

## Why is this useful?
If you're running multiple machines, none of them being the primary one, and you don't want to have the server machine running all the time just to share mouse and keyboard, this guide might be for you.

## Limitations
The keyboard still sends keys to the host system. I have to look into how to disable that.

## Security considerations
I've read on some forums, that running barrier as service with root privileges is dangerous. So use at own risk.

## More Info
I'm no linux expert. I just customized what I found over at [hishamk](https://github.com/hishamk/)'s repo. For more technical details, please visit his repository.

## Tested Configuration
Raspberry Pi 3B, 16GB SD card, Raspberry Pi OS Lite (Debian 12 / bookworm) 32-bit.  "Lite" OS is no desktop environment, command line only.

# Setup steps
1. Boot up and allow to configure itself for the first time; login.
2. `sudo apt update`
3. `sudo apt upgrade`
4. Compile barrier. See their [wiki](https://github.com/debauchee/barrier/wiki/Building-on-Linux) for instructions.
   * Note, the "libqt4-dev" package it references is not available anymore, and can be omitted from the package list when installing -- it's not used.
   * It's important that you compile using the Release build configuration. Edit clean_build.sh, change line 12 from -Debug to -Release.
5. `sudo apt install xorg xserver-xorg-video-dummy`
6. `git clone https://github.com/yozzozo/barrier-headless.git ~/barrier-headless`
7. Ensure /etc/hostname reflects the name of the server appropriately.
8. Create barrier config (i.e. on another machine) and copy it to `/etc/barrier.conf`
   * Ensure you convert the line endings to LF if you used a Windows machine to create the file. For some reason barrier didn't like CRLF.
   * Ensure the config file has the new server hostname in it, in the screens and links sections
9. Following [barrier's steps](https://github.com/debauchee/barrier/wiki/Command-Line#ssl_config), start creating an SSL configuration.  First create the directory hierarchy: `mkdir -p ~/.local/share/barrier/SSL/Fingerprints`
10. Then create the SSL config PEM (key pair): `openssl req -x509 -nodes -days 365 -subj /CN=Barrier -newkey rsa:2048 -keyout /home/<USER>/.local/share/barrier/SSL/Barrier.pem -out /home/<USER>/.local/share/barrier/SSL/Barrier.pem`
11. Generate an SSL config fingerprint for the server: `openssl x509 -fingerprint -sha1 -noout -in ~/.local/share/barrier/SSL/Barrier.pem`
12. Copy the output fingerprint (long hex string separated by colons) into ~/.local/share/barrier/SSL/Fingerprints/Local.txt
13. Copy the ~/.local/share/barrier dir hierarchy to /root (ensuring owner is root for all copied files/dirs)
14. Copy X config file: `sudo cp barrier-headless/xorg_dummy.config /etc/X11/xorg.conf.d/`
15. Copy X service file: `sudo cp barrier-headless/x_dummy.service /etc/systemd/system/`
16. Copy barrier service file: `sudo cp barrier-headless/barrier.service /etc/systemd/system/`
17. Change owner: `sudo chown root:root /etc/systemd/system/barrier.service /etc/systemd/system/x_dummy.service`
18. Add execute flag (not sure if needed): `sudo chmod uog+x /etc/systemd/system/barrier.service /etc/systemd/system/x_dummy.service`
19. Reload systemd: `systemctl daemon-reload`
20. Start X dummy service: `systemctl start x_dummy`
21. Check its status: `systemctl status x_dummy`
22. Start barrier server: `systemctl start barrier`
23. Check its status: `systemctl status barrier`
24. Enable both services to start persistently: `systemctl enable x_dummy.service; systemctl enable barrier.service`
25. `reboot`

Note: it may be necessary to modify connecting clients' SSL trusted hosts file, if the server is being reused with a new certificate.
