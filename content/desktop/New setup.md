
```bash
sudo dnf -y install vim

### Make vim default sudoer editor
echo "Defaults editor=/usr/bin/vim" | sudo tee /etc/sudoers.d/99_custom_editor

### remove password prompts when using sudo
sudo sed -i 's/^#\s*%wheel\s\+ALL=(ALL)\s\+NOPASSWD: ALL/%wheel  ALL=(ALL) NOPASSWD: ALL/' /etc/sudoers
sudo sed -i 's/^%wheel\s\+ALL=(ALL)\s\+ALL/# %wheel  ALL=(ALL) ALL/' /etc/sudoers

sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf -y groupinstall \
      "Development Tools"

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

sudo dnf install @virtualization

sudo dnf -y install Ansible
```

 # Install Homebrew packages
    - name: Install Homebrew packages
      homebrew:
        name:
          - hugo
        state: present
      become_user: "davidt"

pip install --user gnome-extensions-cli
gext install "appindicatorsupport@rgcjonas.gmail.com"
gext enable "appindicatorsupport@rgcjonas.gmail.com"

gext install "legacyschemeautoswitcher@joshimukul29.gmail.com"
gext install "blur-my-shell@aunetx"
gext install "dash-to-dock@micxgx.gmail.com"
gext install "gsconnect@andyholmes.github.io"
gext install "logomenu@aryan_k"
gext install "search-light@icedman.github.com"

Backup gnome settings
dconf dump / > ~/Nextcloud/Documents/dconfsettings.bak

installe apps backup

`cp -r ~/.local/share/gnome-shell/extensions/ ~/Nextcloud/Documents`

Restore settings:
`dconf load / < ~/Nextcloud/Documents/dconfsettings.bak`

Restore extentions:
`cp -r ~/Nextcloud/Documents/extensions/* ~/.local/share/gnome-shell/extensions/`

Restore remmina connections
`cp ~/Nextcloud/remmina/* ~/.var/app/org.remmina.Remmina/data/remmina/`

Restore vimrc
`cat ~/Nextcloud/Documents/dotfiles/vimrc.bak > ~/.vimrc`

Restore ~/.bashrc: (if username is the same)
`cat ~/Nextcloud/Documents/dotfiles/bashrc.bak > ~/.bashrc`

Git
git config --global user.email "tdavetech@gmail.com"
git config --global user.name "linuxreader"
