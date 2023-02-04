# Stylix

Stylix is a NixOS module which applies the same color scheme, font and
wallpaper to a wide range of applications and desktop environments.
In some cases, theming can be activated as early as the bootloader!

It also exports utilities for you to apply the theming to custom parts of your
configuration.

Stylix is built using [base16.nix](https://github.com/SenchoPens/base16.nix#readme),
a library which handles the generation of config files from templates provided by
the [base16](https://github.com/chriskempson/base16#readme) project.

## Installation

You can install Stylix using [Flakes](https://nixos.wiki/wiki/Flakes),
for example:

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    stylix.url = "github:danth/stylix";
  };

  outputs = { self, nixpkgs, home-manager, stylix }: {
    nixosConfigurations."<hostname>" = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        stylix.nixosModules.stylix
      ];
    };
  };
}
```

If [Home Manager](https://github.com/nix-community/home-manager) home-manager is available,
stylix with also import the home-manager module into it (can be disabled with
`stylix.homeManagerIntegration.enable = false`), and the system theme will be used
by default for each user theme.

If you want to use the home-manager module on its own, you need to import the
`homeManagerModules.stylix` output of the flake in your configuration.

## Wallpaper

To get started, you need to set a wallpaper image.

```nix
stylix.image = ./wallpaper.png;
```

The option accepts derivations as well as paths, so you can fetch a wallpaper
directly from the internet:

```nix
stylix.image = pkgs.fetchurl {
  url = "https://www.pixelstalk.net/wp-content/uploads/2016/05/Epic-Anime-Awesome-Wallpapers.jpg";
  sha256 = "enQo3wqhgf0FEPHj2coOCvo7DuZv+x5rL/WIo4qPI50=";
};
```

The wallpaper is the only option which is required! On home-manager used from a NixOS
configuration, the system image will be used by default, thus nothing is required at
all.

## Color scheme

### Automatic color schemes

If you only set a wallpaper, Stylix will use a
[genetic algorithm](https://en.wikipedia.org/wiki/Genetic_algorithm)
to choose a color scheme based on it. The quality of these automatically
generated schemes can vary, but more colorful images tend to have better
results.

You can force a light or dark theme using the polarity option:

```nix
stylix.polarity = "dark";
```

The generated scheme can be viewed in a web browser at
`file:///etc/stylix/palette.html`.

### Manual color schemes

Alternatively, you can choose a pre-made colorscheme from
[the Tinted Theming repository](https://github.com/tinted-theming/base16-schemes).
Either add the repository to your Flake inputs, or fetch it as follows:

```nix
let base16-schemes = pkgs.fetchFromGitHub {
  owner = "tinted-theming";
  repo = "base16-schemes";
  rev = "...";
  sha256 = "...";
};
```

Then you can choose which file you would like to use:

```nix
stylix.base16Scheme = "${base16-schemes}/gruvbox-dark-hard.yaml";
```

If you want to do anything more complex - such as running your own program to
generate the colour scheme - `base16Scheme` can accept any argument which
[`mkSchemeAttrs`](https://github.com/SenchoPens/base16.nix/blob/main/DOCUMENTATION.md#mkschemeattrs)
supports.

If the system `base16Scheme` is set this way, it will be used as default for the
corresponding `home-manager` option only if the user config has the same image. If
the user has changed the image, the default value will be the generated color scheme
from this picture.

## Fonts

The default combination of fonts is:

```nix
stylix.fonts = {
  serif = {
    package = pkgs.dejavu_fonts;
    name = "DejaVu Serif";
  };

  sansSerif = {
    package = pkgs.dejavu_fonts;
    name = "DejaVu Sans";
  };

  monospace = {
    package = pkgs.dejavu_fonts;
    name = "DejaVu Sans Mono";
  };

  emoji = {
    package = pkgs.noto-fonts-emoji;
    name = "Noto Color Emoji";
  };
};
```

These can be changed as you like.

To make things more uniform, you can replace the serif font with sans-serif:

```nix
stylix.fonts.serif = config.stylix.fonts.sansSerif;
```

Or even use monospace for everything:

```nix
stylix.fonts = {
  serif = config.stylix.fonts.monospace;
  sansSerif = config.stylix.fonts.monospace;
  emoji = config.stylix.fonts.monospace;
};
```

## Turning targets on and off

In Stylix terms, a target is anything which can have colors, fonts or a
wallpaper applied to it. Each module in this repository should correspond to a
target of the same name.

Each target has an option like `stylix.targets.«target».enable` to turn its
styling on or off. Normally, it's turned on automatically when the target is
installed. You can set `stylix.autoEnable = false` to opt out of this
behaviour, in which case you'll need to manually enable each target you want to
be styled.

Targets are different between home-manager and NixOS, and sometimes available
in both cases. If both are available, it is always correct to enable both.
