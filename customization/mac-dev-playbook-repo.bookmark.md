# Mac Dev Ansible Playbook (github.com)

<https://github.com/geerlingguy/mac-dev-playbook>

## Description

Mac setup and configuration via Ansible. Contribute to geerlingguy/mac-dev-playbook development by creating an account on GitHub.

## Tags

#mac #ansible

## Content

# GitHub - geerlingguy/mac-dev-playbook: Mac setup and configuration via Ansible. {#github---geerlingguymac-dev-playbook-mac-setup-and-configuration-via-ansible. .reader-title}

geerlingguy

------------------------------------------------------------------------

[![Mac Dev Playbook Logo](https://raw.githubusercontent.com/geerlingguy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png){height="156" width="250"}](https://raw.githubusercontent.com/geerlingguy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png){rel="noopener noreferrer nofollow"}

## Mac Development Ansible Playbook {#mac-development-ansible-playbook dir="auto" tabindex="-1"}

[](#mac-development-ansible-playbook){#user-content-mac-development-ansible-playbook aria-label="Permalink: Mac Development Ansible Playbook"}

[![CI](https://github.com/geerlingguy/mac-dev-playbook/workflows/CI/badge.svg?event=push)](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI)

This playbook installs and configures most of the software I use on my Mac for web and software development. Some things in macOS are slightly difficult to automate, so I still have a few manual installation steps, but at least it\'s all documented here.

## Installation {#installation dir="auto" tabindex="-1"}

[](#installation){#user-content-installation aria-label="Permalink: Installation"}

1.  Ensure Apple\'s command line tools are installed (`xcode-select --install` to launch the installer).

2.  [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html){rel="nofollow"}:

    1.  Run the following command to add Python 3 to your \$PATH: `export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"`
    2.  Upgrade Pip: `sudo pip3 install --upgrade pip`
    3.  Install Ansible: `pip3 install ansible`

3.  Clone or download this repository to your local drive.

4.  Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.

5.  Run `ansible-playbook main.yml --ask-become-pass` inside this directory. Enter your macOS account password when prompted for the \'BECOME\' password.

> Note: If some Homebrew commands fail, you might need to agree to Xcode\'s license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

### Use with a remote Mac {#use-with-a-remote-mac dir="auto" tabindex="-1"}

[](#use-with-a-remote-mac){#user-content-use-with-a-remote-mac aria-label="Permalink: Use with a remote Mac"}

You can use this playbook to manage other Macs as well; the playbook doesn\'t even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com/){rel="nofollow"}, you just need to make sure you can connect to it with SSH:

1.  (On the Mac you want to connect to:) Go to System Preferences \> Sharing.
2.  Enable \'Remote Login\'.

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

    [ip address or hostname of mac]  ansible_user=[mac ssh username]

If you need to supply an SSH password (if you don\'t use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks {#running-a-specific-set-of-tagged-tasks dir="auto" tabindex="-1"}

[](#running-a-specific-set-of-tagged-tasks){#user-content-running-a-specific-set-of-tagged-tasks aria-label="Permalink: Running a specific set of tagged tasks"}

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`\'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

## Overriding Defaults {#overriding-defaults dir="auto" tabindex="-1"}

[](#overriding-defaults){#user-content-overriding-defaults aria-label="Permalink: Overriding Defaults"}

Not everyone\'s development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

    homebrew_installed_packages:
      - cowsay
      - git
      - go

    mas_installed_apps:
      - { id: 443987910, name: "1Password" }
      - { id: 498486288, name: "Quick Resizer" }
      - { id: 557168941, name: "Tweetbot" }
      - { id: 497799835, name: "Xcode" }

    composer_packages:
      - name: hirak/prestissimo
      - name: drush/drush
        version: '^8.1'

    gem_packages:
      - name: bundler
        state: latest

    npm_packages:
      - name: webpack

    pip_packages:
      - name: mkdocs

    configure_dock: true
    dockitems_remove:
      - Launchpad
      - TV
    dockitems_persist:
      - name: "Sublime Text"
        path: "/Applications/Sublime Text.app/"
        pos: 5

Any variable can be overridden in `config.yml`; see the supporting roles\' documentation for a complete list of available variables.

## Included Applications / Configuration (Default) {#included-applications-configuration-default dir="auto" tabindex="-1"}

[](#included-applications--configuration-default){#user-content-included-applications--configuration-default aria-label="Permalink: Included Applications / Configuration (Default)"}

Applications (installed with Homebrew Cask):

-   [ChromeDriver](https://sites.google.com/chromium.org/driver/){rel="nofollow"}
-   [Docker](https://www.docker.com/){rel="nofollow"}
-   [Dropbox](https://www.dropbox.com/){rel="nofollow"}
-   [Firefox](https://www.mozilla.org/en-US/firefox/new/){rel="nofollow"}
-   [Google Chrome](https://www.google.com/chrome/){rel="nofollow"}
-   [Handbrake](https://handbrake.fr/){rel="nofollow"}
-   [Homebrew](http://brew.sh/){rel="nofollow"}
-   [LICEcap](http://www.cockos.com/licecap/){rel="nofollow"}
-   [nvALT](http://brettterpstra.com/projects/nvalt/){rel="nofollow"}
-   [Sequel Ace](https://sequel-ace.com/){rel="nofollow"} (MySQL client)
-   [Slack](https://slack.com/){rel="nofollow"}
-   [Sublime Text](https://www.sublimetext.com/){rel="nofollow"}
-   [Transmit](https://panic.com/transmit/){rel="nofollow"} (S/FTP client)

Packages (installed with Homebrew):

-   autoconf
-   bash-completion
-   doxygen
-   gettext
-   gifsicle
-   git
-   gh
-   go
-   gpg
-   httpie
-   iperf
-   libevent
-   sqlite
-   nmap
-   node
-   nvm
-   php
-   ssh-copy-id
-   cowsay
-   readline
-   openssl
-   pv
-   wget
-   wrk
-   zsh-history-substring-search

My [dotfiles](https://github.com/geerlingguy/dotfiles) are also installed into the current user\'s home directory, including the `.osx` dotfile for configuring many aspects of macOS for better performance and ease of use. You can disable dotfiles management by setting `configure_dotfiles: no` in your configuration.

Finally, there are a few other preferences and settings added on for various apps and services.

## Full / From-scratch setup guide {#full-from-scratch-setup-guide dir="auto" tabindex="-1"}

[](#full--from-scratch-setup-guide){#user-content-full--from-scratch-setup-guide aria-label="Permalink: Full / From-scratch setup guide"}

Since I\'ve used this playbook to set up something like 20 different Macs, I decided to write up a full 100% from-scratch install for my own reference (everyone\'s particular install will be slightly different).

You can see my full from-scratch setup document here: [full-mac-setup.md](https://github.com/geerlingguy/mac-dev-playbook/blob/master/full-mac-setup.md).

## Testing the Playbook {#testing-the-playbook dir="auto" tabindex="-1"}

[](#testing-the-playbook){#user-content-testing-the-playbook aria-label="Permalink: Testing the Playbook"}

Many people have asked me if I often wipe my entire workstation and start from scratch just to test changes to the playbook. Nope! This project is [continuously tested on GitHub Actions\' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).

You can also run macOS itself inside a VM, for at least some of the required testing (App Store apps and some proprietary software might not install properly). I currently recommend:

-   [UTM](https://mac.getutm.app/){rel="nofollow"}
-   [Tart](https://github.com/cirruslabs/tart)

## Ansible for DevOps {#ansible-for-devops dir="auto" tabindex="-1"}

[](#ansible-for-devops){#user-content-ansible-for-devops aria-label="Permalink: Ansible for DevOps"}

Check out [Ansible for DevOps](https://www.ansiblefordevops.com/){rel="nofollow"}, which teaches you how to automate almost anything with Ansible.

## Author {#author dir="auto" tabindex="-1"}

[](#author){#user-content-author aria-label="Permalink: Author"}

This project was created by [Jeff Geerling](https://www.jeffgeerling.com/){rel="nofollow"} (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)).
