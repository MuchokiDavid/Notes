[patorjk ASCII Text Generator](https://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type+Something+&x=none&v=4&h=4&w=80&we=false)

# Ubuntu Zsh Development Environment Setup

This guide reproduces the Zsh environment with:

* Oh My Zsh
* Powerlevel10k theme
* Random `figlet` banner
* Random `lolcat` colors
* Random `figlet` fonts
* Banner displayed on terminal startup and after `clear`
* `zsh-autosuggestions`
* `zsh-syntax-highlighting`
* `you-should-use`

---

## 1. Install the required packages

```bash
sudo apt update

sudo apt install -y \
    zsh \
    git \
    curl \
    wget \
    figlet \
    lolcat
```

If `lolcat` isn't available from your repositories:

```bash
sudo apt install ruby
sudo gem install lolcat
```

---

## 2. Make Zsh your default shell

```bash
chsh -s $(which zsh)
```

Log out and back in.

---

## 3. Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

---

## 4. Install Powerlevel10k

```bash
git clone --depth=1 \
https://github.com/romkatv/powerlevel10k.git \
${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

Open `~/.zshrc` and set:

```zsh
ZSH_THEME="powerlevel10k/powerlevel10k"
```

---

## 5. Install useful plugins

### zsh-autosuggestions

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

### zsh-syntax-highlighting

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

### you-should-use

```bash
git clone https://github.com/MichaelAquilina/zsh-you-should-use \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/you-should-use
```

---

## 6. Enable the plugins

Edit `~/.zshrc`.

Use:

```zsh
plugins=(
    git
    zsh-autosuggestions
    zsh-syntax-highlighting
    you-should-use
)
```

---

## 7. Install NVM (optional but recommended)

```bash
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Then add to `~/.zshrc`:

```zsh
export NVM_DIR="$HOME/.nvm"

[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"
```

Install Node:

```bash
nvm install --lts
```

---

## 8. Configure the random startup banner

Append this to the bottom of `~/.zshrc`:

[patorjk ASCII Text Generator](https://patorjk.com/software/taag/#p=display&f=Avatar&t=LEONE+LAMBORGHINI&x=none&v=4&h=4&w=80&we=false)

```zsh
FONTS=(
    slant
    standard
    big
    small
    banner
    digital
    doom
    chunky
    script
    bubble
    smslant
)

show_banner() {
    FONT=${FONTS[$((RANDOM % ${#FONTS[@]} + 1))]}
    figlet -f "$FONT" "Leone Lamborghini" | lolcat --seed=$RANDOM
}

clear() {
    command clear
    show_banner
}

show_banner
```

This gives you:

* a random ASCII font
* different rainbow colors each time
* a banner every time you open the terminal
* a banner every time you type `clear`

---

## 9. Disable Powerlevel10k Instant Prompt

Since the startup banner writes to the terminal, Instant Prompt must be disabled.

Open:

```bash
nano ~/.p10k.zsh
```

Find:

```zsh
typeset -g POWERLEVEL9K_INSTANT_PROMPT=verbose
```

Replace it with:

```zsh
typeset -g POWERLEVEL9K_INSTANT_PROMPT=off
```

---

## 10. Reload Zsh

```bash
source ~/.zshrc
```

or simply open a new terminal.

---

## Result

Each new terminal session will have:

* Powerlevel10k prompt
* Git integration
* Autosuggestions from history
* Syntax highlighting
* Alias recommendations from `you-should-use`
* Random `figlet` font
* Random rainbow colors using `lolcat`
* Banner redrawn after every `clear`

This setup is suitable for a modern Ubuntu development environment using Git, Node.js, Django, and Vue.



