---
layout: post
title: Keeping My Setup Portable
---

At some point, you'll probably be forced to reinstall the operating system on your computer and/or move your data over to a new computer. It can be a daunting prospect. My time with the [Korora project](https://web.archive.org/web/20180819040707/https://kororaproject.org/) came to a close when the project died, leaving me with an unsupported distro. I decided to move to stock Fedora at that point, and although there was a "migration" path where I could simply upgrade from Korora 28 to Fedora 29, the result left my laptop with some pretty hairy stability issues. At that point, I decided a fresh install would be my only way forward.

Having done some distro hopping in the past, I had some rudimentary notes on how to get my critical files and programs onto a new install so that I could get up and running. But this would be the first time completely blowing away everything on my primary laptop since I made the jump from Windows to Linux some years back. The prospect of losing something key during the move made me more than a little nervous.

I decided to tackle this in the same way I usually tackle a big project:  by breaking it up into a bunch of little ones that I could throw checkboxes next to. If nothing else, it would let me maintain the illusion of progress to myself as I dragged out a task I was somewhat afraid to complete. Nonetheless, the process was a good learning experience, so I thought I would document it here.

## A Rough Category of Migrations

Broadly speaking, everything critical on my laptop fell into one of three categories
 - Data 
 - Programs
 - Configs

 That might be simplifying it a bit, but for any given computer, that more or less describes the stuff that varies between any two installations of the same OS:  your personal files, the programs you have installed, and any tweaks you make to those programs once you have them installed. Much of the learning process was in figuring out what could be automated and what I would have to do by hand after the reinstall. Along the way, I found some methods for making my environment more "portable", so that I could more easily reinstall, migrate, or survive catastrophic data failure.

### Data

 Data encompasses all of your personal files that cannot be reinstalled or retrieved elsewhere: documents, music, pictures, and so forth. There's no way getting around having to manually copy those between machines. Luckily, I regularly make backups of my data files, so this was the *least* stressful part of getting my laptop ready for reinstall. Everything important that wasn't already backed up either went onto a flash drive for safekeeping or onto Google Drive for easy retrieval later.

 One area that I had nearly forgotten about (until I reviewed my ssh_config file further down the list) was backing up some of the SSH and private keys I had generated to get into my various boxes. That would have been rather unfortunate. 

### Programs

 Anything you have installed that is critical to your workflow would fall under the "Programs" category. I began at first by simply opening up my applications list and making a note of which ones I needed. Then I realized that I could make Linux do this *for me*, and greatly decrease the amount of time I spend manually reinstalling stuff.

 First I made a list of everything I had installed at the time:

`dnf list installed | cut -d "." -f 1 > pkg-list.txt`

The syntax was tailored towards a Fedora system, so the commands may be different depending on whether your distro uses `apt`, `dpkg`, `pacman`, etc. I use the cut command to truncate the architecture and version from the package list, since I don't care about downloading a specific version of `vim` so long as I get the latest one my distro's repos will offer. The output changes from this:

    vim-common.x86_64           2:8.1.998-1.fc29            @updates               
    vim-enhanced.x86_64         2:8.1.998-1.fc29            @updates               
    vim-filesystem.noarch       2:8.1.998-1.fc29            @updates               
    vim-minimal.x86_64          2:8.1.998-1.fc29            @updates               
    vim-powerline.noarch        2.7-1.fc29                  @anaconda              

To this:

    vim-common
    vim-enhanced
    vim-filesystem
    vim-minimal
    vim-powerline

Much easier to read, and it allows me to then pass into my new system as an install list via the following command:

`dnf install $(cat pkg-list.txt()`

I highly suggest reviewing the list first, as you may find some programs you don't really care to reinstall on the new system. Additionally, a good number of packages will be preinstalled on your new distro, so you don't necessarily need to put `firefox` in your list if it comes preinstalled on the distro. DNF will helpfully skip anything you pass via the list that is already installed. DNF will *not* skip past packages that it can't install, however:  any package you have in the last that it cannot find in the enabled repos will cause the whole operation to bomb out. I ended up having to edit the list and re-run it a couple of times until I had weeded out the handful of programs I had installed from COPRs or other non-standard repos (`vlc` in particular had me scratching my head for awhile until I remembered that it was hosted through RPMFusion due to some of the nonfree codecs it bundles).

Finally, this process won't capture anything you had installed via `flatpak`. I had about a dozen, so that required falling back on writing them down by hand and reinstalling them via GNOME Software later. I've been pretty impressed with Flatpak so far, but as we can see, it's still got some ways to go before it can completely replace yum/dnf.

### Configs

Once all of your desired programs have been reinstalled, it's time to get them the way you like them. Apply your favorite themes, change the font size, tweak the layout until it the app looks and feels just right. Some applications save all of these changes into binary or otherwise un-readable files that are difficult to get at, particularly if you want to migrate those changes over; you're left with re-entering the changes in manually, or copying the entire directory over, and all of its unforeseen consequences. A well-written application, however, preserves all of its configuration changes into a tidy human-readable file, like a [config file](https://www.freedesktop.org/wiki/Specifications/config-spec/).

Fortunately, most popular programs support such a file, and it's likely your favorite ones do as well, provided you know where to look. On a standard Linux system, per the handy dandy [XDG Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html), the $XDG_CONFIG directory will usually be in `$HOME/.config`. Some examples of the programs I use and what I'm able to control in their config files:

 - The `config` file for audacious tracks  my selected theme, my Last.fm session key, and the all-important `close_to_tray=TRUE` setting for minimizing to the system tray when the program is closed.
 - A `redshift.conf` file can hold manual GPS coordinates for the program so that redshift doesn't need to reach out to the internet using my laptop's geolocation in order to determine sunrise/sunset time.
 - My preferred code editor, SublimeText, carries not only user preferences in config files, but installed packages as well. Rather than copying over the packages themselves, all I needed to do was migrate the package list over when I reinstalled Sublime--the program cleverly looked to see what was needed and downloaded any packages on the list that weren't already installed.

 In the category of "configs" I would also include the venerable [dotfile](https://dotfiles.github.io/). However, at the moment [mine](https://github.com/AdmiralAsshat/dotfiles) still kinda sucked, and are mostly cobbled together from others I've found on the web, such that I don't feel terribly qualified to discuss them just yet. Perhaps in a future blog post.

## Wrapping it all up

Once I had everything ready to go, I was able to test the migration with a couple of dry-runs. I installed a fresh copy of Fedora 29 onto a spare laptop, transferred everything I needed from my flash drive over, and measured how long it took to get up and running. The dry-run helped bring to light some kinks in my migration plan:  the package install list was missing some packages I had from non-default repos; a good number of my dotfile functions didn't work; some programs required just copying the entire application folder over (looking at you, Thunderbird). But once I had fixed those and was happy with the new plan, the actual reinstall/migration only took a few hours.

The system still isn't perfect. Repos and package names differ, such that my package list would likely fail by a pretty wide margin if I tried to hop from one distro to another. But it gives me a pretty solid list to work with, such that I imagine a few minutes plugging missing package names into [Repology](https://repology.org/) would be enough to get up and running if I suddenly found myself stuck on a Debian or Ubuntu system. 