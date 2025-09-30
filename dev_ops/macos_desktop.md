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

     git clone https://github.com/powerline/fonts.git --depth=1
     cd fonts
     ./install.sh
     cd ..
     rm -rf fonts
   
     ZSH_THEME="agnoster"
     Mở Terminal > Preferences (hoặc iTerm2 nếu bạn dùng iTerm2).
     Chọn tab Profiles → Text (hoặc Appearance).
     Ở mục Font, chọn một font có chữ Powerline hoặc Nerd Font, ví dụ: MesloLGS NF, Hack Nerd Font, FiraCode Nerd Font, DejaVu Sans Mono for Powerline
   ```
