# Na MACu

## Inštalácie

### Vscode

```
https://code.visualstudio.com/download
```

pridanie do PATH cez Command Palette príkazom `Install 'code' command in PATH`

### Homebrew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo >> /Users/<USER>/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/<USER>/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
brew update

brew --version
```

nájdenie balíku

```
brew search <nazov-baliku>
```

nainštalované balíky

```
brew list
brew list --cask
```

odinštalovanie balíku

```
brew uninstall <nazov-baliku>
brew uninstall --cask <nazov-baliku>
```

### Midnight Commander

```
brew install midnight-commander

mc --version
```

### Git

```
brew install git
git config --global user.name "<USERNAME>"
git config --global user.email "<EMAIL>"

git --version
```

### SSH key

```
ssh-keygen -t ed25519 -C "<EMAIL>"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
pbcopy < ~/.ssh/id_ed25519.pub
cat ~/.ssh/id_ed25519.pub
```

### PHP, Composer, Laravel

```
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
source /Users/<USER>/.zprofile

php --version
composer --version
laravel --version
```

### Node

```
brew install node

node --version
npm --version
```

### Mailpit

```
brew install mailpit

mailpit version
```

### Meilisearch

```
brew install meilisearch

meilisearch --version
meilisearch --master-key=meilisearch-secret-key
```

### Cocoapods

```
brew install cocoapods

pod --version
```

### Watchman

```
brew install watchman

node --version
npm --version
```


### Java Development Kit

```
brew install --cask zulu@17

brew info --cask zulu@17

open /opt/homebrew/Caskroom/zulu@17/<version number>
```

inštaluje sa cez `Double-Click to Install Azul Zulu JDK 17.pkg`

pridať do terminálu

```
open ~/.zshrc

export JAVA_HOME=/Library/Java/JavaVirtualMachines/zulu-17.jdk/Contents/Home # vložiť do `.zshrc`

source ~/.zshrc

java -version
```

### Android Studio

```
brew install --cask android-studio
```

`~/Library/Android/sdk` - SKD \
`~/.android/avd` - emulátory

vymazanie

```
rm -rf ~/Library/Android/sdk
rm -rf ~/.android/avd
rm -rf ~/.gradle/caches/
rm -rf ~/.gradle/build-cache/
rm -rf ~/Library/Caches/AndroidStudio*
```

### Xcode

inštaluje sa cez `App Store`

```
https://apps.apple.com/sk/app/xcode/id497799835
```

`~/Library/Developer/CoreSimulator/Devices/` - simulátory \
`~/Library/Developer/Xcode/DerivedData/` - buildy \
`~/Library/Developer/Xcode/Archives/` - archivované buildy \
`~/Library/Developer/Xcode/iOS DeviceSupport/` - debug support \
`~/Library/Developer/Shared/Documentation/DocSets/` - uložená dokumentácia \
`~/Library/Caches/com.apple.dt.Xcode/` - Xcode cache

vymazanie

```
rm -rf ~/Library/Developer/CoreSimulator/Devices/*
rm -rf ~/Library/Developer/Xcode/DerivedData/*
rm -rf ~/Library/Developer/Xcode/Archives/*
rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/*
rm -rf ~/Library/Developer/Shared/Documentation/DocSets/*
rm -rf ~/Library/Caches/com.apple.dt.Xcode/*
```

---

## Nástroje

### Zobrazenie veľkých adresárov

`du -h -d 5 -t 1M ~` – vypíše priečinky do hĺbky 5, väčšie než 1 MB \
`sort -hr` – zoradí od najväčšieho \
`head -20 ` – výsledok len pre 20 záznamov \
`awk` – pridá farbu podľa veľkosti

```
du -h -d 5 -t 1M ~ | sort -hr | head -20 | \
awk '{
  size=$1
  path=$2
  if (size ~ /G$/) color="\033[1;31m"
  else if (size ~ /M$/ && int(size) >= 500) color="\033[1;33m"
  else color="\033[1;32m"
  printf "%s%-8s\033[0m %s\n", color, size, path
}'
```
