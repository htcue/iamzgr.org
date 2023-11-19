# Arch Gnome
## 卸载Gnome 多余的包
```bash
sudo pacman -R gnome-maps gnome-music gnome-tour gnome-weather gnome-software epiphany gnome-user-docs  totem simple-scan snapshot

sudo pacman -R yelp
```
## GNOME Shell browser integration Installation Guide
```
git clone https://aur.archlinux.org/gnome-browser-connector.git
cd gnome-browser-connector
makepkg -si
```
## Gnome Dash to dock
```
https://extensions.gnome.org/extension/307/dash-to-dock/
```
## Start Bluetooth Server
```bash
systemctl enable bluetooth
systemctl start bluetooth
systemctl status bluetooth
```
