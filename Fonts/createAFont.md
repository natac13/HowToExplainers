# How to Create a Font

This is about combining two fonts together to make one. The idea for this is from a Github comment where people were looking for a free alternative to [Operator Mono](https://www.typography.com/fonts/operator/styles); which is a beautiful font. However quite expensive.

### Combining FiraCode and Script12 BT

First [Font Forge]() needs to be installed.

```
sudo add-apt-repository ppa:fontforge/fontforge;
sudo apt update
sudo apt install fortforge
```

**Keep in mind** that the fonts folder `/usr/share/fonts` is sudo protected and therefore to open the program, use:

```
sudo fontforge
```

- Then open up FiraCode-Regular.otf. Ignore any warnings that may pop up.
- Select at the top left:
  - Element -> Font Info -> PS Names
- Change:
  - Fontname: `NewFontName-Regular`
  - Family Name: `NewFontName`
  - Name for Humans: `NewFontName Regular`
- On the left navigate to **TTF Names**
- Change:
  - Fullname: `NewFontName-Regular` (or other suffix if differntt style)

**Keep in mind**

Each font name needs to have its own suffix but the FontFamily must be the same for each variant.

- Click OK and when the Dialog about changing the UniqueID comes up select the **Change** button.
- Next is to generate the new font!
- Go: File -> Cenerate Fonts
- Change:
  - the font type to **OpenType (CFF)**
  - deselect the **Validate Before Saving** box
  - Make sure the new font name is correct.
  - Then generate.
  - **Upon a failure error; it is usually do to the fact that you do not have sudo permission for the directory.**

Now repeat the above steps to create the Italic variant of the new font but using Script12 BT as the font.

Therefore the Font Info:

```
Fontname: NewFontName-Italic
Family Name: NewFontName
Name for Humans: NewFontName Italic
Weight: Regular
```

TTF Names info changes to:

```
Family: NewFontName
UniqueId: NewFontName-Italic
FullName: NewFontName-Italic
```

**If any field is red** right click and select `Detach from PostScript Names.`

## Set the Font in VSCode

Open the setting JSON file with `ctrl+shift+p` and search for _settings_.

```
"editor.fontFamily": "NewFontName", backups...,
```

Credit goes to [frenchtoast747](https://github.com/frenchtoast747) who wrote the original comment on Github that I am just recording for my future use.
