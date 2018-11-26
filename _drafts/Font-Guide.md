---
author: Polybar Team
account: polybar
title: The Definitive Guide to Font Configuration in Polybar
excerpt: TODO
---

* TOC
{:toc}

## Font Basics
* Glyphs
* Codepoints
* Not all glyphs in all fonts
* Private Use Area

### Listing Installed Fonts
You can list all fonts installed on your system with `fc-list`.
On my system the output looks something like this:

```console
$ fc-list
...
/usr/share/fonts/noto/NotoSansMono-Regular.ttf: Noto Sans Mono:style=Regular
...
```

There are several hundreds of those lines.

Every line represents one font installed on your system
Everything before the first colon is the absolute path to the font file.
In the example that would be

```
/usr/share/fonts/noto/NotoSansMono-Regular.ttf
```

Everything else on the line is the font pattern, it's divided up as follows:

```
          separator
              |
Noto Sans Mono:style=Regular
|            | |           |
|------------| |-----------|
      |             |  
 font family      style
```

If you want to use a particular font, everything after the file path can be used as-is inside your polybar config:
```ini
font-0 = Noto Sans Mono:style=Regular
```

### Font Properties
A font pattern is generally composed of a font family as well as one or more properties, all separated by colons.

In its general form a pattern may look like this:
```
<font-family>(:<property>=<value>)*
```

* Properties can be added and removed from fonts
* Not all fonts support all properties

`antialias`

### Bitmap Fonts
`size` vs. `pixelsize`

## Finding Font Icons
`gucharmap`

## Adding a Font to Configuration
Polybar uses a font list in its config to get the font patterns to load. The font pattern are set as the values of the list keys `font-0, font-1, ...` inside the bar section.
Each bar defines its own font list, for example:
```ini
[bar/bar1]
...
font-0 = misc fixed:style=Regular
font-1 = Unifont:style=Medium
...

[bar/bar2]
...
font-0 = Noto Sans:style=Regular
...
```

After you have installed a font and want to add it to your polybar config, you need to determine what the font pattern for this font is.
To determine this, search through the output of `fc-list` (see [Listing Installed Fonts](#???) for detailed instructions).

This pattern can then be used to set the value for an element in the `font-*` list in your bar section.


### Font Configuration Options
`size`, `pixelsize` (which for scalabale which for bitmap)
`dpi`

### Verifying loaded Fonts
There are two main ways to confirm that the fonts you have specified in your config are going to be loaded correctly.

#### With `polybar`
Polybar can directly tell you what fonts it loaded for which font pattern. For that you will have to start polybar additonally with the `-l info` argument.
If, for example, you start your bar like this:
```console
$ polybar --config=~/.config/polybar/config bar
```

You can add the `-l info` parameter as follows:
```console
$ polybar -l info --config=~/.config/polybar/config bar
```

This will give you quite a lot of information. Look out for the lines that start with `* Loaded font "..."`, there will be one per font in your config:
```
...
* Loaded font "Font Awesome 5 Brands:style=Regular:size=10" (name=Font Awesome 5 Brands, offset=3, file=/usr/share/fonts/TTF/fa-brands-400.ttf)
...
```

Here `Font Awesome 5 Brands:style=Regular:size=10` is the exact font pattern you used in the config and 
```
name=Font Awesome 5 Brands
```
is the name of the font that was actually loaded. Check if those two names match up to make sure the correct font was loaded.
If they don't match, there is something wrong with your font patterns. 
See [Adding Font to Configuration ](#???) for how to add the correct font patterns.

#### With `fc-match`
Apart from directly using polybar to verify a font pattern will be correctly loaded is to use `fc-match`.

```ini
font-0 = Noto Sans Mono:style=Regular;-3
```

```console
$ fc-match "Noto Sans Mono:style=Regular"
```

This approach can be used for quick verification and should return the same result as with the approach above. However, this is not guaranteed, there may be input on which `fc-match` will match the fonts differently than polybar, we don't know of any such input though. If you really want to be sure, use the approach with directly using polybar.


### Glyph Resolution
When polybar has to figure out with which font to draw a glyph with it does so as follows:
It goes through the list of fonts (`font-0`, `font-1`, ...) in order and uses the first font that can render the given glyph.

If a formatting tag (`%{T}`) or a font property (e.g. `label-font`) was used, polybar first searches the font given there and subsequently searches through the font list as described above.

#### Changing Fonts

## Troubleshooting

### Glyph Conflicts
It is possible that two glyphs from different fonts map to the same codepoint. When this codepoint is then rendered on the bar, it may be rendered as any of the glyphs that correspond to that codepoint, depending on which order you have listed your fonts. See [Glyph Resolution](#???) for details on which glyph is chosen.

If the glyph from the wrong font is shown, you can fix this by explicity specifying which font should be used.

This can be done either with the [`%{T}` formatting tag](https://github.com/jaagr/polybar/wiki/Formatting#font-t):
```
The next icon is rendered with font-2: %{T3}😎%{T-}
```

Or setting the font index as a property of the label:
```ini
label = This entire label is rendered with font-4
label-font = 5
```

Both these approaches use a 1-based index. This means `label-font = 3` uses the third font specified, which is `font-2`.

### Finding Unmatched Characters
Font issues often occur as dropped character warnings that look like this in the polybar output:
```
warn: Dropping unmatched character  (U+e096)
```

This means that the dropped character is in none of the loaded fonts. 

1. Copy the dropped character from the polybar output. Make sure to only copy a single character.
1. Open `gucharmap`
1. Open the search dialog with `Ctrl-F` or `Search > Find`
1. Paste the dropped character into the search box and press `Search` 

### Emojis
Emoji support generally comes from the `Noto Emoji` font, but by default emojis often appear very large.
To fix this `Noto Emoji` supports the `scale` property, which is inversely proportional to the size of the emojis.
Here is a sample font pattern that has been shown to work:
```
Noto Emoji:style=Regular:scale=10:antialias=true:size=8
```

**Note:** The font name may be different on other machines. See [Finding Font Pattern](#???).

This approach may also work for other emoji fonts.
