---
id: journey
aliases: []
tags:
  - nix
  - linux
---

# My Linux Journey Up Until Now

At the start of March 2024, I decided to move to NixOS. I thought it would be worthwhile to write up
my overall journey to nix.

To start with why I chose Nix, we have to briefly cover my journey through Linux. In June 2023 I got 
a new SATA drive that I put into my laptop, since it seemed interesting I installed Kubuntu on top of
Windows. I figured I would try out Kubuntu, but ultimately continue to reside in Windows. However,
quite the contrary happened. After I installed Kubuntu, I never booted up Windows for multiple hours
again. 

## To start: Why I Chose Linux

I never considered completely abandoning Windows. I had a working system, my games ran well, I was able
to do homework, so why was Kubuntu so appealing to me? One word: flexibility. 

Throughout my programming journey I loved making hyper-extensible and complicated projects. My Minecraft
mods quickly exploded in scope. A simple block-outline mod quickly became a multi-week project. 
I rewrote a massive chat mod three times until I felt good about it. I really wanted to be able
to over-engineer all of my creations and create the perfect tool for me. Even outside of programming,
I would invest many hours into finding what I believed to be *the best* tool for whatever need 
I was trying to fulfill.

This thought process has consumed my process of learning Linux. I was amazed by the flexibility and
customization that is possible within Linux. [Kanata](https://github.com/jtroo/kanata) for keybinds,
neovim for coding. On Windows these were possible, but it was a hassle to get anything working.
Maybe this was due to my lack of trying, but the point stands that on Linux a lot of these system level
changes are significantly easier.

While I have had great experience, the journey hasn't been all sunshine and roses, and I definitely 
wouldn't recommend Linux to *everyone*. It is a very powerful tool, but depending on the job it does
require a bit manual intervention tweaking compared to Windows. But in exchange, you can tune *everything*
about your system. 

Up until October of 2023, Kubuntu served me well. After some deliberation I decided to switch to [EndeavourOS](https://endeavouros.com/),
an arch based distribution.
For me, that was easily one of the best decisions I've made in a long time. *My* problems (maybe not your problems)
with Kubuntu were:

- Fresh packages were hard to get a hold of.
    - New projects on GitHub were yet to be on the main APT repository.
- Not the most flexible system
    - Couldn't get the most recent version of KDE Plasma
- I found nvidia difficult and after it broke a few times after updating, I stopped wanting to update. (This was entirely my fault lol)

With my gained knowledge of my most recent Linux endeavours, I'm sure a lot of these problems
could have been circumvented in some way. The issue with not having a recent Plasma version
was mainly due to me installing Kubuntu LTS.

I *loved* EndeavourOS. It was significantly easier to get up and running than arch,
and had sane defaults that were really nice. A lot of things just worked. Nvidia
worked out of the box, there were *so* many packages, being able to make full use 
of the arch wiki taught me so much. To anyone who has experience with Linux,
I *highly* recommend EndeavourOS if you want to try a rolling release. In fact,
loved it so much I installed it again on my NVME (yes, I finally stopped using my SATA
drive for my operating system).

### TLDR so far

In short: 
- I started Kubuntu because Windows didn't fulfill my needs of customization
    - I switched to EndeavourOS because Kubuntu started with too much bloat.
     and didn't have the tools I wanted. I also wanted to start on a more bare install
     (yes yes, I know not as bare bones as raw arch) and fine-tune my system.

Choosing a Linux distribution is a very personal choice and is influenced by many factors.
Based on my comfortability with Linux I ended up choosing very different distribution. For
other people, they can make radically different decisions. I'll always recommend starting
out simpler for an initial install, but after that, the world is your oyster (is that the saying?).
 
Before I installed EndeavourOS I was very close to committing to NixOS. At the time, I
didn't think I needed the robustness of Nix. Nix felt like it would be slower and a bit
overkill for my system.

## So, Nix...

For those who don't know, [[linux/nix/index.md|Nix]] is a distribution where you can define everything
declaratively instead of imperatively. This essentially means you have to explicitly state everything about
your system, and then Nix will build it when you're ready. Check out the article for more information.

[[linux/nix/home_manager.md|Home Manager]] is what finally pushed me over. Having a centralized spot with nearly *all* of my 
configuration was exceedingly enticing. Neovim, firefox, NuShell, bat, yazi... being able to 
reproduce it whenever I wanted would be super nice. 

Having used it for about a month now, I can really see the appeal and have been enjoying the entire experience.
I have more thoughts on my [[linux/nix/onemonth.md|one month]] post.
