# openSUSE Leap 15.6 KDE X11 minimal installation

X11 is the Leap default, hence the stay in X11.  
Autologin with wayland gives my amd laptop a crash and return to cli (works as intended on Intel)(seems to be a known bug in those versions).

## Base system installation

* boot from the openSUSE Leap iso (check installation media if it's a freshly formatted medium)
* Connect to wifi if applicable
* activate online repositories
* choose the "Server" role
* choose your partitionning  (I use encryption without LVM, and no swap file (16GB RAM), works good so far)
* choose your country / city
* create your user
* choose last options
  * booting - go and change bootloader options if the value of 8 seconds is too much at startup. 5 is a good middle ground
  * security - I disable the firewall and ssh service and policykit default privileges : easy
  * network configuration - click on "switch to NetworkManager"
* clic install, computer will reboot at the end
* login into the command line interface

## KDE installation

`sudo zypper in patterns-kde-kde_plasma konsole`  
To me, it's the best ratio minimalistic/usability. If you want do to your own experiment with a more minimalistic approach, use the package plasma-session, but you'll have to add a lot of things by yourself afterwards.

I'm a noob with patterns and don't want to deal with it (package reinstallation in my back) so for the sake of simplicity I uninstall the pattern immediately after.  

`sudo zypper remove patterns-kde-kde_plasma`

`sudo zypper remove opensuse-welcome filelight tigervnc plasma5-desktop-emojier khelpcenter5 xterm`  
I do not like/use those applications but it's mostly a matter of taste!

`sudo systemctl set-default graphical.target`  
`sudo reboot`

At first boot remove broken shortcuts like the one for discover. If you need global scaling (ex for your 32" 4K) add PLASMA_USE_QT_SCALING=1 in /etc/environment then change for ie 150% and reboot.

Now you have a minimal but functional KDE.

## Software

### A word about it

* some software is available as a flatpak only (ex missionCenter) and not in the repos
* some are available at the same time in both. There are tradeoffs choosing one or the other. Some people say everything should be installed as flatpak... On the contrary, some software might need a more complete integration in the system... or the distribution packager can tweak it (ex fedora activate hardware video acceleration on AMD in their firefox rpm)
* you'll have to make your own opinion about what you prefer
* kde discover is a mixed bag, some people say it interacts badly with zypper and can't install system packages anyway... I choose not to install it and do everything manually in the CLI

### Add more software sources options with opi and flatpak

`sudo zypper in -y opi`  
[OPI](https://github.com/openSUSE/opi) can install software from OBS or Packman without leaving it enabled, allowing quick and clean installs of software not in the standard repos.

`sudo zypper in flatpak`  
`flatpak remote-add --if-not-exists --user flathub <https://dl.flathub.org/repo/flathub.flatpakrepo>`  
[Flatpak](https://www.flatpak.org/) is an alternative way of installing apps on linux. Apps are packaged with their dependencies, hence portables and more or less isolated from the rest of the system. Use [flatseal](https://flathub.org/apps/com.github.tchx84.Flatseal) if you need to change permissions of an app.  

### Some multimedia codecs and hardware acceleration enabled mesa

`sudo opi -n install codecs`

### Virtualization

You can use Yast to install KVM server and tools. Some prefer it over Virtualbox.

`sudo zypper in virtualbox`  
`usermod -aG vboxusers yourUserName`  
`reboot`  
[Virtualbox](https://www.virtualbox.org/) is a popular type2-hypervisor.  

Oddly, after I install virtualbox I can't start a podman/distrobox container because of /dev/vboxusb permissions... I don't understand fully the implications but here's a bad fix... unfortunately at each reboot /dev/vboxusb is recreated with wrong permissions. Idk how to solve it right now hence I use kvm over virtualbox...  
`chmod -R 0755 /dev/vboxusb`

### Containers

`sudo zypper in podman distrobox`  
[Distrobox](https://distrobox.it/) creates integrated containers.

`flatpak install boxbuddy`  
[Boxbuddy](https://github.com/Dvlv/BoxBuddy) is a nice gui interface for distrobox.

### Office

`sudo opi install vscode`  
[Visual Studio Code](https://code.visualstudio.com/) is the trendy open source code editor.
Alternatively, you may prefer `flatpak install VSCodium` (less telemetry but less integration). Or you could install vscode in a distrobox container which will serves as a development platform where you'll install all your programming stuff.

`sudo zypper remove MozillaFirefox && flatpak install firefox` (select when asked the "stable" one)  
[Firefox](https://www.mozilla.org/fr/firefox/new/) is a popular browser from Mozilla.  
You can keep the opensuse version from zypper (firefox extended support, opensuse theming) or you can use the flatpak version directly from mozilla with hardware decoding enabled. It's a matter of taste and as always there are pros and cons.

`sudo zypper in MozillaThunderbird`  
[Thunderbird](https://www.thunderbird.net/en-US/) is a very popular mail client and the "sibling" of Firefox.

`flatpak install chrome` (select when asked the "stable" one) or `opi install chrome` if you prefer a RPM installation  
[Chrome](https://www.google.com/chrome/index.html) is the most used browser nowadays.

`sudo zypper in okular`  
[Okular](https://okular.kde.org/) is the kde document viewer and can open a lot of file formats.

`sudo zypper in nextcloud-desktop-dolphin`  
[Nextcloud](https://nextcloud.com/) desktop client with dolphin integration, if you have a Nextcloud server instance somewhere of course!

`sudo zypper in gutenprint`  
[Gutenprint](https://sourceforge.net/projects/gimp-print/) is a collection of printer drivers. I personnaly have better results using yast to configure printers than with kde printer setup.

[Rustdesk](https://rustdesk.com/)  
Rustdesk is a better alternative to Teamviewer. [Download](https://github.com/rustdesk/rustdesk/releases/tag/1.2.3-2
) the "rustdesk...suse.rpm" and `sudo zypper in` the rpm to install it, say (i)gnore to the error (rpm isn't signed).

### Multimedia

`flatpak install spotube`  
[Spotube](https://github.com/KRTirtho/spotube) is a spotify player using youtube music as a back-end. A bit slow to start songs but free and legal.

`flatpak install webcord`  
[Webcord](https://github.com/SpacingBat3/WebCord) is a Discord client.

`flatpak install vlc`  
[Video player](https://www.videolan.org/vlc/) with codecs included, is the standard video player

`flatpak install missioncenter`  
[Mission Center](https://gitlab.com/mission-center-devs/mission-center/) is a clear and good system monitor (with %gpu used in videos, to test hardware decoding in firefox for example)

#### Firefox hardware acceleration

To see if hardware acceleration is available, launch a youtube video in 4K, launch mission center and expand it to see the gpu video encode/decode, a 0% means software decoding by the CPU. To confirm that, launch in a new tab about:support and check the HARDWARE_VIDEO_DECODING, it should be "default - available" if you use intel.  
If it isn't, you can change in about:config media.hardware-video-decoding.force-enabled to true, and media.ffmpeg.vaapi.enabled to true. Restart firefox and test again. If it still doesn't work, try one way of installing firefox or the other (zypper in MozillaFirefox or flatpak install firefox)

### To update the system

`sudo zypper dup`  
`flatpak update`  
`sudo reboot`
