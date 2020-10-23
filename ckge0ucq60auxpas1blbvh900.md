## Using Alacritty as a Neovim UI

I have been using [Alacritty](https://github.com/alacritty/alacritty) + [Tmux](https://github.com/tmux/tmux) as my default terminal for 2 years now and am very happy with it. 

[Vim](https://www.vim.org/) is one of my go to editors. I use [Neovim](https://neovim.io/), a Vim fork with additional features. For any development activities I find it inconvenient to use Vim inside terminal as I will be using it for various other things. I prefer a GUI. For Neovim there are a lot of GUIs available as mentioned in [their wiki page](https://github.com/neovim/neovim/wiki/Related-projects). I have used VimR, Oni, gnvim and neovide. Among them I chose VimR as I find it very stable. However, none of these GUIs were as fast as Neovim running inside terminals like Alacritty.

Recently I came to know that, any number of Alacritty instances can be started using the `alacritty` commad. It has an option to start Alacritty with a different configuration. Also, we can pass a command the will be executed on startup. It even has a way to set a custom title in title bar. On looking at these options I got an idea, which is using Alacritty like a Neovim GUI.

```
> alacritty -h
alacritty 0.5.0 (a6681e3)
Christian Duerr <contact@christianduerr.com>
Joe Wilm <joe@jwilm.com>
GPU-accelerated terminal emulator

USAGE:
    alacritty [FLAGS] [OPTIONS]

FLAGS:
...

OPTIONS:
...
    -e, --command <command>...                       Command and args to execute (must be last argument)
        --config-file <config-file>
            Specify alternative configuration file [default: $HOME/.config/alacritty/alacritty.yml]
    -t, --title <title>                              Defines the window title [default: Alacritty]
        --working-directory <working-directory>      Start the shell in the specified working directory
```

I started of with creating a new Alacritty configuration because my [default configuration](https://github.com/arunvelsriram/dotfiles/blob/master/config/alacritty/alacritty.yml#L247) makes Alacritty enter a tmux session on startup. Also, I wanted to keep my Alacritty Terminal configuration and Alacritty Neovim configuration separate.

Then I was able to launch Neovim like this:

```
> alacritty --config-file ~/.config/alacritty/anvim.yml --title nvim  -e $SHELL -lc "nvim"
```


![Screenshot 2020-10-17 at 11.46.36 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1602958643624/scAOKr-W4.png)

Nice. But this has few issues:

* The process we started will be in the foreground and occupies a terminal/tmux pane
* It might write some output to stdout which would be noisy
* On closing the terminal/tmux pane our alacritty process will be killed
* It opens Neovim with user's home directory as working directory. Opening a project directory or `$PWD` would be helpful

So, how do we solve these? I wrote a shell function called `anvim`. It starts the alacritty process in the background and will ignore any hungup signal. So it won't be killed even if we close the terminal/tmux pane. It is also capable of opening `$PWD` or a given directory or file.

```bash
# alacritty neovim
anvim() {
  if [ -n "${1}" ]; then
    local target=$(realpath $1)
  fi

  nohup alacritty --config-file ~/.config/alacritty/anvim.yml -t "nvim - ${target}" -e $SHELL -lc "nvim ${target}" >/dev/null &
}
```

* First argument is the file or directory to open. If not given, it opens home directory
* [`nohup`](https://en.wikipedia.org/wiki/Nohup) is used to ignore `hangup` signal from terminal
* Trailing `&` pushes the process to background
* `>/dev/null` redirects stdout to null

![Screenshot 2020-10-17 at 11.29.51 PM.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1602957619049/eJfyVfAVH.png)

As you can see in the screenshot, I did `anvim .` within my dotfiles directory and it opened it in Neovim inside a new Alacritty instance. Also, notice the title in the new instance.

Alacritty Neovim configuration I use and the `anvim` function is available in my dotfiles repo as well:

* [anvim.yml](https://github.com/arunvelsriram/dotfiles/blob/master/config/alacritty/anvim.yml)

* [`anvim` function](https://github.com/arunvelsriram/dotfiles/blob/master/oh-my-zsh-custom/plugins/personal/personal.plugin.zsh#L25)