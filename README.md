# Setting up a new computer

My personal installation script for a new computer.

Note, this sets up an extremely opinionated and highly personalized installation, with my preferences and dotfiles. Please adjust as needed!

Some resources borrowed from:

- https://github.com/ruyadorno/installme-osx/
- https://gist.github.com/millermedeiros/6615994
- https://gist.github.com/brandonb927/3195465/

## Install from script:

Open the terminal, then:

```sh
bash -c "`curl -L https://git.io/fjdb1`"
```

This command runs the following script:

```shell
#!/bin/sh

echo "
░░░░░░░░░▄░░░░░░░░░░░░░░▄
░░░░░░░░▌▒█░░░░░░░░░░░▄▀▒▌
░░░░░░░░▌▒▒█░░░░░░░░▄▀▒▒▒▐
░░░░░░░▐▄▀▒▒▀▀▀▀▄▄▄▀▒▒▒▒▒▐
░░░░░▄▄▀▒░▒▒▒▒▒▒▒▒▒█▒▒▄█▒▐
░░░▄▀▒▒▒░░░▒▒▒░░░▒▒▒▀██▀▒▌
░░▐▒▒▒▄▄▒▒▒▒░░░▒▒▒▒▒▒▒▀▄▒▒▌
░░▌░░▌█▀▒▒▒▒▒▄▀█▄▒▒▒▒▒▒▒█▒▐
░▐░░░▒▒▒▒▒▒▒▒▌██▀▒▒░░░▒▒▒▀▄▌
░▌░▒▄██▄▒▒▒▒▒▒▒▒▒░░░░░░▒▒▒▒▌
▌▒▀▐▄█▄█▌▄░▀▒▒░░░░░░░░░░▒▒▒▐
▐▒▒▐▀▐▀▒░▄▄▒▄▒▒▒▒▒▒░▒░▒░▒▒▒▒▌
▐▒▒▒▀▀▄▄▒▒▒▄▒▒▒▒▒▒▒▒░▒░▒░▒▒▐
░▌▒▒▒▒▒▒▀▀▀▒▒▒▒▒▒░▒░▒░▒░▒▒▒▌
░▐▒▒▒▒▒▒▒▒▒▒▒▒▒▒░▒░▒░▒▒▄▒▒▐
░░▀▄▒▒▒▒▒▒▒▒▒▒▒░▒░▒░▒▄▒▒▒▒▌
░░░░▀▄▒▒▒▒▒▒▒▒▒▒▄▄▄▀▒▒▒▒▄▀
░░░░░░▀▄▄▄▄▄▄▀▀▀▒▒▒▒▒▄▄▀
░░░░░░░░░▒▒▒▒▒▒▒▒▒▒▀▀

You are blessed by Mac Doge!
"

# Colorize

# Set the colours you can use
black=$(tput setaf 0)
red=$(tput setaf 1)
green=$(tput setaf 2)
yellow=$(tput setaf 3)
blue=$(tput setaf 4)
magenta=$(tput setaf 5)
cyan=$(tput setaf 6)
white=$(tput setaf 7)

# Resets the style
reset=`tput sgr0`

# Color-echo. Improved. [Thanks @joaocunha]
# arg $1 = message
# arg $2 = Color
cecho() {
  echo "${2}${1}${reset}"
  return
}

echo ""
cecho "###############################################" $red
cecho "#        DO NOT RUN THIS SCRIPT BLINDLY       #" $red
cecho "#         YOU'LL PROBABLY REGRET IT...        #" $red
cecho "#                                             #" $red
cecho "#              READ IT THOROUGHLY             #" $red
cecho "#         AND EDIT TO SUIT YOUR NEEDS         #" $red
cecho "###############################################" $red
echo ""

# Set continue to false by default.
CONTINUE=false

echo ""
cecho "Have you read through the script you're about to run and " $red
cecho "understood that it will make changes to your computer? (y/n)" $red
read -r response
if [[ $response =~ ^([yY][eE][sS]|[yY])$ ]]; then
  CONTINUE=true
fi

if ! $CONTINUE; then
  # Check if we're continuing and output a message if not
  cecho "Please go read the script, it only takes a few minutes" $red
  exit
fi

# Here we go.. ask for the administrator password upfront and run a
# keep-alive to update existing `sudo` time stamp until script has finished
sudo -v
while true; do sudo -n true; sleep 60; kill -0 "$$" || exit; done 2>/dev/null &

##############################
# Root											 
##############################

cd ~
mkdir -p repos

##############################
# Prerequisite: Install Brew #
##############################

echo "Installing brew..."

if test ! $(which brew)
then
	## Don't prompt for confirmation when installing homebrew
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null
fi

# Latest brew, install brew cask
brew upgrade
brew update

homebrew_taps=(
	"caskroom/cask"
  "caskroom/fonts"
)
for homebrew_tap in "${homebrew_taps[@]}"; do
	brew tap "$homebrew_tap"
done



#############################################
### Generate ssh keys & add to ssh-agent
### See: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
#############################################

echo "Generating ssh keys, adding to ssh-agent..."
read -p 'Input email for ssh key: ' useremail

echo "Use default ssh file location, enter a passphrase: "
ssh-keygen -t rsa -b 4096 -C "$useremail"  # will prompt for password
eval "$(ssh-agent -s)"

# Now that sshconfig is synced add key to ssh-agent and
# store passphrase in keychain
ssh-add -K ~/.ssh/id_rsa

# If you're using macOS Sierra 10.12.2 or later, you will need to modify your ~/.ssh/config file to automatically load keys into the ssh-agent and store passphrases in your keychain.

if [ -e ~/.ssh/config ]
then
    echo "ssh config already exists. Skipping adding osx specific settings... "
else
	echo "Writing osx specific settings to ssh config... "
   cat <<EOT >> ~/.ssh/config
	Host *
		AddKeysToAgent yes
		UseKeychain yes
		IdentityFile ~/.ssh/id_rsa
EOT
fi

##############################
# Node and nvm
##############################


echo "Installing NVM"
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash

echo "Installing Node"
. ~/.nvm/nvm.sh
nvm install --lts 
nvm use --lts

echo "Upgrading npm"
npm install -g npm

##############################
# Install via Brew           #
##############################

echo "Starting brew app install..."

homebrew_packages=(
  "git"
  "mas"
  "rclone"
  "thefuck"
  "tig"
  "yarn"
  "zsh-completions"
  "zsh"
)

for homebrew_package in "${homebrew_packages[@]}"; do
  brew install "$homebrew_package"
done

homebrew_cask_packages=(
  "1password"
  "alfred"
  "calibre"
  "docker"
  "evernote"
  "firefox"
  "font-fira-code"
  "font-meslo-for-powerline"
  "google-chrome"
  "gpg-suite"
  "iterm2"
  "karabiner-elements"
  "notion"
  "postman"
  "scroll-reverser"
  "spotify"
  "timing"
  "virtualbox"
  "visual-studio-code"
  "vlc"
  "whatsapp"
)

for homebrew_cask_package in "${homebrew_cask_packages[@]}"; do
  brew cask install "$homebrew_cask_package"
done

### Run Brew Cleanup
brew cleanup

#############################################
### Installs from Mac App Store
#############################################

echo "Installing apps from the App Store..."

### find app ids with: mas search "app name"
brew install mas

mac_app_store_apps=(
  "568494494" # Pocket
  "585829637" # Todoist
  "425424353" # The Unarchiver
  "405399194" # Kindle (1.26.1)
)

### Mas login is currently broken on mojave. See:
### Login manually for now.

cecho "Need to log in to App Store manually to install apps with mas...." $red
echo "Opening App Store. Please login."
open "/Applications/App Store.app"
echo "Is app store login complete.(y/n)? "
read response
if [ "$response" != "${response#[Yy]}" ]
then
	for mac_app_store_app in "${mac_app_store_apps[@]}"; do
		mas install "$mac_app_store_app"
	done
else
	cecho "App Store login not complete. Skipping installing App Store Apps" $red
fi

#############################################
### VS Code
#############################################

echo "Setting up VS Code"

cat << EOF >> ~/.bash_profile
# Add Visual Studio Code (code)
export PATH="$PATH:/Applications/Visual Studio Code.app/Contents/Resources/app/bin"
EOF

vscode_extensions=(
  "capaj.vscode-exports-autocomplete"
  "christian-kohler.path-intellisense"
  "Compulim.vscode-clock"
  "dbaeumer.vscode-eslint"
  "eamodio.gitlens"
  "EditorConfig.EditorConfig"
  "esbenp.prettier-vscode"
  "Gruntfuggly.todo-tree"
  "ms-azuretools.vscode-docker"
  "ms-vsliveshare.vsliveshare"
  "naumovs.color-highlight"
  "octref.vetur"
  "streetsidesoftware.code-spell-checker"
  "vscode-icons-team.vscode-icons"
  "waderyan.gitblame"
  "wix.vscode-import-cost"
)

for vscode_extension in "${vscode_extensions[@]}"; do
  code --install-extension "$vscode_extension"
done

# ==================================================================
# Repos
# ==================================================================


cd ~/repos

echo "Cloning scripts"
git clone https://github.com/fhavrlent/scripts.git

echo "Cloning dotfiles"
git clone https://github.com/fhavrlent/dotfiles.git


# ==================================================================
# Dotfiles
# ==================================================================


cd ~/repos/dotfiles

echo "VS Code dotfiles"
rm -f ~/Library/Application\ Support/Code/User/settings.json
ln ./vscode.json ~/Library/Application\ Support/Code/User/settings.json


#############################################
### Set OSX Preferences - Borrowed from https://github.com/mathiasbynens/dotfiles/blob/master/.macos
#############################################

# Close any open System Preferences panes, to prevent them from overriding
# settings we’re about to change
osascript -e 'tell application "System Preferences" to quit'


##################
### Finder, Dock, & Menu Items
##################

# Keep folders on top when sorting by name
defaults write com.apple.finder _FXSortFoldersFirst -bool true

# Expand save panel by default
defaults write NSGlobalDomain NSNavPanelExpandedStateForSaveMode -bool true
defaults write NSGlobalDomain NSNavPanelExpandedStateForSaveMode2 -bool true

# Save to disk (not to iCloud) by default
defaults write NSGlobalDomain NSDocumentSaveNewDocumentsToCloud -bool false

# Finder: show all filename extensions
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# Automatically quit printer app once the print jobs complete
defaults write com.apple.print.PrintingPrefs "Quit When Finished" -bool true

# Avoid creating .DS_Store files on network or USB volumes
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

# Use list view in all Finder windows by default
# Four-letter codes for the other view modes: `icnv`, `clmv`, `Flwv`
defaults write com.apple.finder FXPreferredViewStyle -string "Nlsv"

# Minimize windows into their application’s icon
defaults write com.apple.dock minimize-to-application -bool true

# Don’t show recent applications in Dock
defaults write com.apple.dock show-recents -bool false

# "Disable the “Are you sure you want to open this application?” dialog"
defaults write com.apple.LaunchServices LSQuarantine -bool false

##################
### Text Editing / Keyboards
##################

# Disable smart quotes and smart dashes
defaults write NSGlobalDomain NSAutomaticQuoteSubstitutionEnabled -bool false
defaults write NSGlobalDomain NSAutomaticDashSubstitutionEnabled -bool false

# Disable auto-correct
defaults write NSGlobalDomain NSAutomaticSpellingCorrectionEnabled -bool false

###############################################################################
# Screenshots / Screen                                                        #
###############################################################################

# Require password immediately after sleep or screen saver begins"
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

# Save screenshots to the desktop
defaults write com.apple.screencapture location -string "$HOME/Downloads"

# Save screenshots in PNG format (other options: BMP, GIF, JPG, PDF, TIFF)
defaults write com.apple.screencapture type -string "png"

###############################################################################
# Trackpad, mouse, keyboard, Bluetooth accessories, and input                 #
###############################################################################

# Stop iTunes from responding to the keyboard media keys
launchctl unload -w /System/Library/LaunchAgents/com.apple.rcd.plist 2> /dev/null

###############################################################################
# Mac App Store                                                               #
###############################################################################

# Enable the automatic update check
defaults write com.apple.SoftwareUpdate AutomaticCheckEnabled -bool true

# Download newly available updates in background
defaults write com.apple.SoftwareUpdate AutomaticDownload -int 1

# Install System data files & security updates
defaults write com.apple.SoftwareUpdate CriticalUpdateInstall -int 1

###############################################################################
# Photos                                                                      #
###############################################################################

# Prevent Photos from opening automatically when devices are plugged in
defaults -currentHost write com.apple.ImageCapture disableHotPlug -bool true

###############################################################################
# oh-my-zsh
###############################################################################

echo "Setting up oh-my-zsh"
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
echo "zsh dotfiles"
rm -f ~/.zshrc
ln .zshrc ~/.zshrc
. ~/.zshrc

cd ~
```


----


# Manual Configuration

These apps need to be configured manually.

For OSX settings, I'm still looking for the command line way to change this preference.

#### Set Scroll Reverser preferences

##### Scrolling Section

Checked:

- Reverse Scrolling
- Reverse vertical
- Reverse Mouse

Unchecked:
- Reverse Trackpad
- Reverse Tablet
- Reverse Horizontal

##### App Section

Checked:

- Start at login

Unchecked:

- Show in menu bar

#### Iterm2

* Iterm2 -> Preferences -> General
	* Check: Load preferences from custom folder /Users/filip/repos/dotfiles
	* Check: Save changes to folder when Iterm2 quits

#### System Preferences Configuration

**General**

* Appearance - Dark
* Default web browser: Firefox

**Dock**

* [ ] Magnification
* [ ] Show recent applications in Dock
* [ ] Automatically hide and show the Dock


**Mission Control**

* Dashboard: Off
* [ ] Automatically rearrange Spaces based on most recent use

**Security & Privacy**

* Firewall -> on
* Firewall options -> Enable stealth mode
* FileVault -> Turn On FileVault (encrypt harddrive)

**Display**

* Night Shift (flux) -> Schedule -> Sunrise to Sunset
* Color Temp Max

**Keyboard**

* Touch Bar Shows: Expanded Control Strip
* Press Fn key to: Show F1, F2, etc. Keys
* Turn off spotlight (use Alfred instead)
	* Keyboard -> Shortcuts -> Spotlight -> Deselect all
* Customize control strip -> Delete siri from touchbar
* Use F1, F2 keys as standard on external keyboards

**Trackpad**

* Check everything

**iCloud**

* Checked
	* iCloud drive
	* Keychain
	* Find My Mac

**Internet Accounts**

* Add Google account

**Sharing**

* Ensure everything is unchecked

**Touch ID**

* Add fingers
* Check all

**Users & Groups**

* make sure guest account is turned off

**Siri**

* Disable


**Finder Preferences**

* General
	* New finder window show: home folder
* Sidebar
	* Check all
* Advanced
	* Check all except Keep folders on top

#### Firefox

* Sign into firefox to sync profile & bookmarks

#### Alfred

* Set Cmd + Space to Alfred hotkey

#### Karabiner Elements

* Complex modifications
	* Change caps_lock to command+control+option+shift
