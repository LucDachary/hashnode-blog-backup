## My Basic Custom Keyboard Layout

I'm a software engineer working on a french AZERTY layout keyboard. While it is well suited to write French, it's just as flawed as the QWERTY layout keyboard when it comes to coding. Earlier this year I grew tired of contorting my right hand to reach keys such as \\(\texttt /\\) (slash), \\(\texttt{\\\\}\\) (backslash) or the infuriating \\(\texttt\`\\) (backtick), and I learned how to customise __X11 keymaps__ using `xmodmap`.

This article 's goal is to rush you through your own basic configuration. Feel free to open the provided links for a deeper understanding.

I'm using Archlinux and I used its wiki's [Xmodmap](https://wiki.archlinux.org/title/Xmodmap) page.

# Xmodmap
## Your current keymap
Your keymap is made of __keycodes__ and __keysyms__  ([Xmodmap's introduction](https://wiki.archlinux.org/title/Xmodmap#Introduction)).

To browse the current keymap on your computer, run `xmodmap -pke`. You get one line of output per _keycode_, and all the associated _keysyms_. Here is an extract of mine:

```
…
keycode  24 = a A a A ae AE …
keycode  25 = z Z z Z guillemotleft less …
keycode  26 = e E e E EuroSign cent …
keycode  27 = r R r R paragraph registered …
keycode  28 = t T t T tslash Tslash …
keycode  29 = y Y y Y leftarrow yen …
…
```

The first _keysym_ is the __bare value__. It means that if I press the `a` key on my keyboard (associated with keycode 24) I get a lower-case `a`. The next ones on the line are the values __with modifiers__. For instance the second _keysym_ is obtained pressing my key with `Shift` (so `Shift + key`),  and it produces an upper-case `A` (see [keymap table](https://wiki.archlinux.org/title/Xmodmap#Keymap_table) for details about the other modifiers).

## Save your keymap
Make a copy of your current, default keymap, into your home directory.
```shell
xmodmap -pke > ~/.Xmodmap
```

## Customise it!
You can do many things but to stay safe I'm only __swapping__ some symbols, so I can still type the same number of symbols as before, but in an easier way. For instance I swapped `ù` and `/` so writing a slash is now as easy as pressing `ù` (between `m` and `*` on AZERTY keyboards). If I want a `ù` I can still press `Shift + :`, where the slash used to be.

Open `~/.Xmodmap` and copy-paste all the _keycode_ lines you want to twist __to the end of the file__. This way keeping track of changes if very easy: you can comment a line out (using `!`) if you don't want a mapping anymore. The previous default declaration will still exist.

On the following screenshot you can see the original bindings followed by the lines I duplicated. To perform the swap explained above I switched the values in __blue__, and it resulted in the values in __cyan__.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1660506566360/ahjht21WS.png align="center")

To tell your computer to use this new keymap, run `xmodmap ~/.Xmodmap`.

There you go! You can now fix the “issues” of your keyboard layout. For information here is the complete list of my alterations:

```xmodmap
! Swaps ù and /
keycode  48 = slash percent ugrave percent twosuperior Ugrave twosuperior
keycode  60 = colon ugrave colon slash division division division
! Swaps ~ and é
keycode  11 = asciitilde 2 asciitilde 2 eacute Eacute asciitilde
! Swaps ç and `
! Swaps è and \
keycode  16 = backslash 7 backslash 7 ccedilla Egrave dead_grave
keycode  17 = underscore 8 underscore 8 egrave macron backslash
keycode  18 = dead_grave 9 ccedilla 9 asciicircum Ccedilla asciicircum
! Swaps @ and à
keycode  19 = at 0 at 0 agrave Agrave at
! Swaps # and ² (sometimes ² original key is seen as œ).
! oe → numbersign
keycode  49 = numbersign OE oe OE leftdoublequotemark rightdoublequotemark leftdoublequotemark
keycode  12 = quotedbl 3 quotedbl 3 oe cedilla numbersign
```

## Load your own keymap on startup using `xinit`
To have your new keymap automatically loaded on session startup, add the following line into your `~/.xinitrc` file. It stipulates that if the path `~/.Xmodmap` is a file then you run `xmodmap ~/.Xmodmap`.
```shell
[[ -f ~/.Xmodmap ]] && xmodmap ~/.Xmodmap
```
