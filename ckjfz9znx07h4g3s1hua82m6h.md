## Preventing Firefox From Entering Safe Mode

Recently I started using [yabai](https://github.com/koekeishiya/yabai/) + [skhd](https://github.com/koekeishiya/skhd) for window management in OS X. 

I have set up `Caps Lock` as a `Hyper` key using [Karabiner elements](https://github.com/pqrs-org/Karabiner-Elements) and dedicated it for yabai + skhd. 

`Hyper` key simply produces the effect of `Shift + Alt + Command + Control`. Basically, if I press `Caps Lock + <something>` it will be converted to `Shift + Alt + Command + Control + <something>`.

I wanted to set up some hotkeys to open applications that I frquently use like Terminal and Browser. I am a Firefox user so I wanted to open Firefox whenever I press `Hyper + B`. I added an skhd configuration like this:

```
hyper - b : open "/Applications/Firefox Developer Edition.app"
```

But it didn't work as expected. All I got was this:

![Firefox Safe Mode prompt screenshot](https://cdn.hashnode.com/res/hashnode/image/upload/v1609608514081/2iFGJa518.png)

Firefox was trying to start in Safe Mode. After some research, I came to know that **pressing `Alt` while opening Firefox signals it to open in Safe Mode**.

You can reproduce this by executing `open "/Applications/Firefox Developer Edition.app"` and pressing `Alt` immediately before Firefox launches.

`Hyper` key is the problem here. As I mentioned before `Hyper = Shift + Alt + Command + Control` and it contains `Alt` as one of the keys so Firefox was signalled to open in Safe Mode.

Luckily, there is an environment variable `MOZ_DISABLE_SAFE_MODE_KEY` using which we can prevent Firefox from entering Safe Mode even if `Alt` key is pressed.

Execute `MOZ_DISABLE_SAFE_MODE_KEY=1 open "/Applications/Firefox Developer Edition.app"` and press `Alt` immediately, you would see that Firefox opening normally.

I fixed my skhd configuration using the same:

```
hyper - b : export MOZ_DISABLE_SAFE_MODE_KEY=1; open "/Applications/Firefox Developer Edition.app"
```

### References:

* [https://github.com/mcandre/dotfiles/blob/master/setenv.MOZ_DISABLE_SAFE_MODE_KEY.plist#L16](https://github.com/mcandre/dotfiles/blob/master/setenv.MOZ_DISABLE_SAFE_MODE_KEY.plist#L16)
* [https://github.com/arunvelsriram/dotfiles/blob/master/skhdrc#L5](https://github.com/arunvelsriram/dotfiles/blob/master/skhdrc#L5)
