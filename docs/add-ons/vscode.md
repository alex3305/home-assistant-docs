# Visual Studio Code for Home Assistant

With Visual Studio Code for Home Assistant it is really easy to edit your YAML configuration on your Home Assistant machine. I prefer to have my configuration within a version control system, but this add-on provides me with the ability to also edit on server. And I can push my changed back into version control.

## Installation

Installation of this add-on is quite straight forward. The add-on is available as a community add-on within the Home Assistant add-on store.

## Configuration

I just used the stock configuration, which worked fine for me.

## Extensions

Personally, I really prefer the Subliminal color scheme. Which I have installed as an color theme extension and applied to vscode. I also tend to prefer the IntelliJ IDEA keybindings.

## Git configuration

Configuring Git was a bit trickier than I've expected. First of all you'll need your Git private key set up on your Home Assistant server. I had it located within my user's `.ssh` directory. To get the key into this add-on I've just used this command:

```bash
sudo cp ~/.ssh/* addons/data/a0d7b954_vscode/.ssh/
```

After that I had to set up my user and e-mail within the vscode terminal. Something like this:

```bash
git config --global user.name "Alex"
git config --global user.email "alex@example.com"
```

Finally I was able to effectively use Git from within VSCode.
