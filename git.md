# Git a Github

**!** `#` v kóde znamená **komentár**

---

inštalácia `git` cez príkazový riadok a overenie verzie

```
apt-get install git
git --version
```

`git` nastavenie

```
git config --global user.name "username"
git config --global user.email "useremail"
```

vytvoriť zložku projektu `project`

```
sudo mkdir project
```

nastaviť oprávnenia pre užívateľa `user`

```
sudo chown -R user git
```

kontrola opravnení

```
ll
```

prejsť do zložky a otvoriť v nej editor s podporou gitu ako je `vscode`, ortodoxný linuxový uctievač použije `vim` pretože si nič iné nezaslúži

```
cd project
code .
```

vytvoriť nový súbor `readme.md`

inicializovať `git`

```
git init
# Initialized empty Git repository in /var/www/project/.git/
```

zobrazenie skrytých súborov

```
ls -la
```

overenie stavu projektu

```
git status

# On branch master

# No commits yet

# Untracked files:
#  (use "git add <file>..." to include in what will be committed)
#        readme.md

# nothing added to commit but untracked files present (use "git add" to track)
```

evidovanie súboru `readme.md`

```
git add readme.md

git status

# On branch master

# No commits yet

# Changes to be committed:
#  (use "git rm --cached <file>..." to unstage)
#        new file:   readme.md
```

commitnutie súboru, parameter `-m` pridá komentár

```
git commit -m "prvý commit readme.md"

# [master (root-commit) 73ca806] prvý commit readme.md
#  1 file changed, 53 insertions(+)
#  create mode 100644 readme.md
```

rekaputulácia: po zmene súboru `readme.md` s komentárom `zmena`

```
git status
git add readme.md
git commit -m "zmena"
```

historia zmien projektu

```
git log

# commit 73ca806b654fc2bdb590faa1d9b88cdd63728b23
# Author: user <email>
# Date:   Mon Apr 18 10:22:08 2022 +0200

#    prvý commit readme.md
```

vytvorené súbory `index.html` a `app.js`

pridanie a commitnutie nových súborov, parameter `.` vyberie všetky súbory v adresári, paremeter `*.png` vyberie všetky súbory s príponou png, atď...

```
git add .
git commit -m "pridané nové súbory"
```

ak po úprave a označení súboru `git add` nechceme poslať zmenu do novej verzie, môžeme to zrušiť príkazom `git restore --staged`

```
git restore --staged readme.md
```

preskočenie `git add` pridaním parametra `a` do commitu, nefunguje pre nové súbory, tie treba pridať cez `git add`

```
git commit -am "preskočenie príkazu git add pridaním parametra a do commitu"
```

po vymazaní súboru `app.js` označenie a commitnutie

```
git add app.js
git commit -m "vymazaný súbor app.js"
```

upravenie logu parametrami, z logu sa vychádza klávesou `q`

```
git log --graph --decorate --abbrev-commit --all --pretty=oneline
```

kontrola zmeny v súbore príkazom `git diff`

```
git diff

# +
# +kontrola zmeny v súbore príkazom `git diff`
# +
# +```
# +git diff
# +```
```

vrátenie na poslednú verziu súboru príkazom `git checkout`

```
git checkout -- readme.md
```

pre vrátenie sa staršej verzii programu v `git log` si nájdeme commit k verzii, ktorý pridáme do príkazu `git checkout`

```
git checkout 85ea366b6224f0c71be512a75da1a39cdbc86328
```

vrátenie na master vetvu zadaním príkazu `git checkout master`

```
git checkout master
```

pri commite bez komentáru sa môže otvoriť Vim, upravovať sa dá v insert móde stlačením `i`, do prvého riadku napísať komentár, stlačením `ESC` sa vráti do command módu, uložiť a zavrieť sa dá príkazom `:wq`

```
git add .
git commit
i
# napísať komentár
:wq
```

na githube vytvoríme nový repozitár `git`, github ponúkne možnosti ak je ak ešte nieje alebo už je vytvorený repozitár, príkaz `git remote add origin` vytvorí spojenie s githubom.

príkaz `git push -u origin main` odošle zmeny na server, parameter u nastaví `origin main` ako defaultné, stačí ďalej písať `git push` až do zmeny

```
# nový repozitár
git init
git add README.md
git commit -m "first commit"
git branch -M main          # master
git remote add origin git@github.com:user/project.git
# git remote add origin https://github.com/user/project.git
git push -u origin main     # master

```

```
# existujúci repozitár
git remote add origin git@github.com:user/project.git
# git remote add origin https://github.com/user/project.git
git branch -M main          # master
git push -u origin main     # master
```

príkaz `git pull origin master` stiahne zo servera zmeny v `master` vetve, potom stačí písať `git pull`

```
git pull origin master
git pull
```

príkaz `git remote update` skontroluje zmeny na serveri, na základe stavu podľa `git status` spustiť `git push` (Your branch is ahead...) alebo `git pull` (Your branch is behind...)

```
git remote update
# Fetching origin
git status
# On branch master
# Your branch is ahead of 'origin/master' by 1 commit.
git push
```

viac informácii o zmene `git whatchanged`

```
git whatchanged origin/master -n 1
```

v prípade, že sa robí na viacerých miestach, pri `git push` môže nastať konflikt, ak je na serveri už iná verzia

```
git push

# To github.com:user/project.git
#  ! [rejected]        master -> master (fetch first)
# error: failed to push some refs to 'git@github.com:user/project.git'
# hint: Updates were rejected because the remote contains work that you do
# hint: not have locally. This is usually caused by another repository pushing
# hint: to the same ref. You may want to first integrate the remote changes
# hint: (e.g., 'git pull ...') before pushing again.
# hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

v takom prípade treba stiahnuť verziu a opraviť konflikt

```
git pull

<<<<<< HEAD
    <title>Dokumentácia</title>
======
    <title>Nový titulok</title>
>>>>>> ebfb64354104e67b847995494f8c71da4525db8a
```

príkaz `git branch` zobrazí všetky vetvy

```
git branch
```

novú vetvu vytvoríme `git branch` s názvom novej vetvy

```
git branch newbranch
```

medzi vetvami sa prepíname `git checkout` a názov vetvy

```
git checkout newbranch
```

vytvoríme nový súbor `new.html`, označíme `git add` a spravíme commit `git commit -m "komentár"`

```
git add new.html
git commit -m "komentár"
```

príkazom `git push origin newbranch` odošleme súbor do novej vetvy

```
git push origin newbranch
```

novú vetvu a prepnutie do nej je možne spraviť skrátene s parametrom `b`

```
git checkout -b newbranch
```

vetvy zlúčime tým, že sa prepneme do hlavnej vetvy a zadáme príkaz `git merge` a názov vetvy

```
git checkout master
git merge newbranch
```

tak isto pri zlučovaní vetvy môže nastať konflikt, ktorý je treba najprv vyriešiť a následne commitnúť

```
<<<<<< HEAD
    <title>New</title>
======
    <title>New 2</title>
>>>>>> newbranch2


git add new.html
git commit -m "komentár"
```

rozdiel v zlúčení medzi `merge` a `rebase` je ten, že pri `merge` sa presunú z jednej vetvy do druhej, pričom sa vetvy zanechajú, pri `rebase` sa vetvy spoja do jednej

súbory a adresáre ktoré nechceme zdielať na server, zapíšeme do súboru `.gitingnore` ktorý si najprv vytvoríme

```
/node_modules
/vendor
.env
```

`.gitignore` commitneme a odošleme na server

```
git add .gitignore
git commit -m "gitignore"
git push origin master
```

vymazanie cache

```
git rm -r --cached .
```

lokálne premenovanie vetvy

```
git checkout nazov-vetvy
git branch -m novy-nazov-vetvy
```

lokálne vymazanie vetvy

```
git branch --delete nazov-vetvy
```
