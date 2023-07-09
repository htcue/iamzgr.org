## Archlinux Install Chinese Input
### Install Fcitx5
```bash
sudo pacman -S fcitx5-im
```
### Edit /etc/environment file
```bash
nano /etc/environment

GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus

```
