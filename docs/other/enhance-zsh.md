# Enhance Zsh

On macOS, the default shell is **zsh**. There are many ways to enhance zsh to optimize our experience.

If you are using Ubuntu, then you first need to install zsh:

```shell
sudo apt install zsh -y
chsh -s /bin/zsh
```

## [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)

First of all, install **Oh My Zsh**:

```shell
sh -c "$(curl -fsSL https://install.ohmyz.sh/)"
```

Show command execution time in history command output:

```title="~/.zshrc" hl_lines="9"
...

# Uncomment the following line if you want to change the command execution time
# stamp shown in the history command output.
# You can set one of the optional three formats:
# "mm/dd/yyyy"|"dd.mm.yyyy"|"yyyy-mm-dd"
# or set a custom format using the strftime function format specifications,
# see 'man strftime' for details.
HIST_STAMPS="yyyy-mm-dd"

...
```

Enable plugins:

```title="~/.zshrc" hl_lines="8"
...

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(git copypath copyfile)

...
```


## [zsh-users](https://github.com/zsh-users)

Zsh community projects with zsh plugins, suggestions:
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting), 
[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions), 
[zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

```title="~/.zshrc" hl_lines="8"
...

# Which plugins would you like to load?
# Standard plugins can be found in $ZSH/plugins/
# Custom plugins may be added to $ZSH_CUSTOM/plugins/
# Example format: plugins=(rails git textmate ruby lighthouse)
# Add wisely, as too many plugins slow down shell startup.
plugins=(... zsh-syntax-highlighting zsh-autosuggestions zsh-history-substring-search)

...
```

!!! Note
    
    You can configure the shortcut key of **zsh-history-substring-search**:

    ```title="~/.zshrc" hl_lines="3-5"
    ... 

    # History-substring-search
    bindkey "$terminfo[kcuu1]" history-substring-search-up
    bindkey "$terminfo[kcud1]" history-substring-search-down
    ```
