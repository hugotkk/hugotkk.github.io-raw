---
title: "Move Forward Backward in Chrome"
date: 2021-08-20T10:18:29+01:00
---

I used the emacs keybind to maniplicate the string on command line but it does not work at chrome on macos.

I am unhappy with that until I find the solution on this post
- https://stackoverflow.com/questions/20146972/is-there-a-way-to-make-alt-f-and-alt-b-jump-word-forward-and-backward-instead-of

``` bash
sudo mkdir -p ~/Library/Keybindings/
sudo vi ~/Library/Keybindings/DefaultKeyBinding.dict
```

```
{
    "~d" = "deleteWordForward:";
    "^w" = "deleteWordBackward:";
    "~f" = "moveWordForward:";
    "~b" = "moveWordBackward:";
}
```


