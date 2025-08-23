# Na MACu

## Inštalácie

### Homebrew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo >> /Users/<USER>/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/<USER>/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
brew update

brew --version
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

## Nástroje

### Zobrazenie veľkých adresárov

`du -h -d 5 -t 1M ~` – vypíše priečinky do hĺbky 5, väčšie než 1 MB
`sort -hr` – zoradí od najväčšieho
`head -20 ` – výsledok len pre 20 záznamov
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
