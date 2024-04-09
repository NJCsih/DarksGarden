---
id: mpv
aliases: []
tags: []
title: Configuring mpv
---

I recently moved my entire mpv config into a Nix tracked one. Home manager has some really 
useful utilities for configuring mpv, but there were some weird things that I hadn't seen
before.

# What the frick is wrapping?

If you look up mpv in Nix pacakges, you'll be greeted with these two (main) packages:

```
mpv
mpv-unwrapped
```

"Um ok, mpv seems more generic? Guess I'll use that one..."

To help explain what is happening here we need to take a look at a little 
example configuration.

```nix
wrapMpv (pkgs.mpv-unwrapped.override { ffmpeg = pkgs.ffmpeg-full })  {
  youtubeSupport = true;
  scripts = with pkgs.mpvScripts; [
    uosc
    thumbfast
    mpris
  ];
}
```

What the heck is this??? Lets start by looking in the parenthesis. We have the line:

```nix
pkgs.mpv-unwrapped.override { ffmpeg = pkgs.ffmpeg-full }
```

Ah, this looks familiar! This is just some simple attribute overriding. Lets see
if we can find the line... [here it is](https://github.com/NixOS/nixpkgs/blob/ff0dbd94265ac470dda06a657d5fe49de93b4599/pkgs/applications/video/mpv/default.nix#L13).

Ok so we can give this a temporary name to make it easier to read, `my-unwrapped-mpv`. Cool. Lets look back at what we got.

```nix
wrapMpv my-unwrapped-mpv {
  youtubeSupport = true;
  scripts = with pkgs.mpvScripts; [
    uosc
    thumbfast
    mpris
  ];
}
```

This is just a function! [Here's the declaration](https://github.com/NixOS/nixpkgs/blob/ff0dbd94265ac470dda06a657d5fe49de93b4599/pkgs/applications/video/mpv/wrapper.nix).
Look! There's the `youtubeSupport` and `scripts` attribute. So what does wrapping actually do?

## Explaining Wrapping

Some resources:

- [manual](https://ryantm.github.io/nixpkgs/stdenv/stdenv/#fun-makeWrapper)
- [gist](https://gist.github.com/CMCDragonkai/9b65cbb1989913555c203f4fa9c23374)

Looking into the definition for `wrapMpv` we see that in the `postBuild` script it calls `makeWrapper`.
`makeWrapper` essentially just makes it easier to run a package with certain arguments and environment variables.

For mpv this specifically adds the executables to the `PATH` that are needed for certain features (youtube, python, lua ...).
On top of that, it specifies arguments to run the executable with. In this case it bundles up all the scripts we specified and
adds them as arguments.

So that's what the different packages do, and that's what wrapping does as well! We're now ready to package our own scripts.

# Scripts

Looking at the declaration of the wrapper, we see this:
> Scripts: a set of derivations (probably from `mpvScripts`) where each is
>  expected to have a `scriptName` passthru attribute that points to the
>  name of the script that would reside in the script's derivation's
>  `$out/share/mpv/scripts/`. A script can optionally also provide an `extraWrapperArgs` passthru attribute.

So we have need a `$out/share/mpv/scripts/<name>.lua` file and we also have `extraWrapperArgs` that add arguments for the wrapper 
(those attributes and environment variables we were talking about!).

So we could manually do this, but nixpkgs has a [buildLua](https://github.com/NixOS/nixpkgs/blob/ff0dbd94265ac470dda06a657d5fe49de93b4599/pkgs/applications/video/mpv/scripts/buildLua.nix) function that does most of this for us.

Here's what it does: 

- Extracts the script name from the package name (mpv-uosc -> uosc).
- Finds a `main.lua` within the `src` and copies that to the `$out/share/mpv/scripts/<name>.lua` (`uosc.lua`)
- Sets some meta attributes
- The package is done

## Packaging our own script

Lets put this to the test by packaging [skipsilence](https://codeberg.org/ferreum/mpv-skipsilence). 

Giving this link a quick [nurl](https://github.com/nix-community/nurl) we get this source:

```nix
fetchFromGitea {
  domain = "codeberg.org";
  owner = "ferreum";
  repo = "mpv-skipsilence";
  rev = "2c9b50cc492ee517a41d5e8555c6e491f0b3998c";
  hash = "sha256-J06+gP/ND0wGQQPx1oTZuo6xhja2Ix2vK9xPtvhpJ8w=";
}
```

Checking this repository out we see that there exists a `main.lua` so we don't have to set `passthru.scriptName`
to the main file, but it should be really easy if you need to.

Put it together and we have:

```nix
{
  pkgs,
  fetchFromGitea,
  lib,
  unstableGitUpdater,
}:

pkgs.mpvScripts.buildLua {

  pname = "mpv-skipsilence";
  version = "unstable-2024-04-06";
  passthru.updateScript = unstableGitUpdater {};

  src = fetchFromGitea {
    domain = "codeberg.org";
    owner = "ferreum";
    repo = "mpv-skipsilence";
    rev = "2c9b50cc492ee517a41d5e8555c6e491f0b3998c";
    hash = "sha256-J06+gP/ND0wGQQPx1oTZuo6xhja2Ix2vK9xPtvhpJ8w=";
  };

  meta = {
    description = "Increase playback speed during silence";
    homepage = "https://codeberg.org/ferreum/mpv-skipsilence";
    license = lib.licenses.unfree; # At the time of writing this is unlicensed
  };
}
```

I put this in `skipsilence.nix`, so now in our `scripts` option for the wrap, we can add 

```nix
(pkgs.callPacakage ./skipsilence.nix { })
```

to our script list.

# Shaders

Shaders are actually easier than scripts (if you're using home-manager).

For shaders I like to set them up in a respective profile and then have an input
or another script handle loading the profiles.

In home manager you have the option `programs.mpv.profiles`. Here's how I would 
go about setting up [CFL Prediction](https://github.com/Artoriuz/glsl-chroma-from-luma-prediction) profile.

```nix
let
  cflPrediction = pkgs.fetchFromGitHub {
    owner = "Artoriuz";
    repo = "glsl-chroma-from-luma-prediction";
    rev = "37a449a94f8c532b4ed06822ea8b0398f4209737";
    hash = "sha256-3Q8aDgdsKnJhd3fmEoJMTDoAkA5s2g1ve9SB1HKAB1U=";
  };
in
config.programs.mpv.profiles = {
  "interpolate-shaders" = {
    glsl-shaders-toggle = [ "${cflPrediction}/CfL_Prediction.glsl" ];
  };
};
```

Then when using `apply-profile interpolate-shaders` it will toggle the shader. With home-manager,
I recommend using the `-toggle`, `-clr`, and `-append` options for lists. `-set` may work for one,
but with list home-manager just duplicates the config option. For set you can use `-clr` and then
`-append`. Clear doesn't need any option, it can just be `glsl-shaders-clr = ""`.
