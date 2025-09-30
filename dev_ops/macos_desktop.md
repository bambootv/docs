1. Iterm2
   ```sh

     xcode-select --install

     git --version

     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
     eval "$(/opt/homebrew/bin/brew shellenv)"

     brew install zsh
     sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
     upgrade_oh_my_zsh
   ```
