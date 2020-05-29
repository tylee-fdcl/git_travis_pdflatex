### LaTeX + Git + Travis &rightarrow; release pdf


## 4. TeX Live with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc

Thanks to [Joseph Wright](https://tex.stackexchange.com/users/73/joseph-wright) who pointed out that they use something based on this setup for LaTeX3 development.

#### Pro:
* Uses any compiler you want, this can be useful for example if you need pdflatex for some cases like the `minted` package.
* Fast, because of caching
* Almost all required packages are downloaded automatically
* Tested to work with the `minted` package
* Tested to work with [custom fonts](#texlive-custom-fonts)

#### Con:
* You need to copy extra build files to each repository, besides the `.travis.yml`.

Build time example file: 1-2 minutes

Want this? Instructions [below](#pdflatex).

## <a name="pdflatex">Instructions for building with pdflatex/lualatex/latexmk/xelatex/texliveonfly/etc and TeX Live</a>

If for some reason you prefer the pdflatex (or any other) engine with the TeX Live distribution, read on.
This is based on the [LaTeX3 build file](https://github.com/latex3/latex3/blob/master/support/texlive.sh).

This method installs an almost minimal TeX Live installation on Travis, and compiles with pdflatex.
This repo contains:
- The TeX Live install script `texlive_install.sh` including profile `texlive/texlive.profile` (specifies for example the TeX Live scheme)
- A Travis configuration file
- Demonstration LaTeX files in `src/`

### Instructions

* Install the Travis GitHub App by going to the [Marketplace](https://github.com/marketplace/travis-ci), scroll down, select Open Source (also when you want to use private repos) and select 'Install it for free', then 'Complete order and begin installation'. 
 * Now you should be in Personal settings | Applications | Travis CI | Configure and you can allow access to repositories, either select repos or all repos.
* Copy the files in the folder [`4-texlive`](4-texlive) to your repo, so `.travis.yml` and the `texlive/` folder.
* Specify the right tex file in the `.travis.yml` under the `script` section. Possibly you also need to change the folder in `before_script` if not using `src/`.
* Commit and push, you can view your repositories at [travis-ci.com](https://travis-ci.com/).
(Thanks to [@jason-neal](https://github.com/PHPirates/travis-ci-latex-pdf/pull/6) for improving this)
* For deploying to GitHub releases, see the notes [below](#deploy).
* If the build fails because some package is missing you have to add it manually in `texlive_packages`. This configuration uses the `texliveonfly` package which tries to download missing packages but sometimes the error message is non-standard and that fails. 

  In that case, put the package you want to install in `texlive/texlive_packages`, by checking at https://www.ctan.org/pkg/some-package to see in which TeX Live package it is contained (which may be different than the LaTeX package name). If you find a package which is not automatically downloaded, it would be great if you could let us know by submitting an issue.

  You can find required LaTeX packages of your document (say main.tex) by installing the snapshot package and then running `pdflatex -draftmode -interaction=batchmode "\RequirePackage{snapshot}\input{main}"` (thanks to [Pablo Gonzalez](https://github.com/PHPirates/travis-ci-latex-pdf/pull/24#issuecomment-521780469) for finding this).
   This will produce a file `snapshot.dep` with dependencies. Note this requires that you have all required packages installed on your system.
   
* If you use custom fonts, [read on](#texlive-custom-fonts).
* Tip from [gvacaliuc](https://github.com/gvacaliuc/travis-ci-latex-pdf): In order to maintain the install scripts in a central repo and link to them, you could also just copy `.travis.yml` and replace
```yaml
install:
 - source ./texlive/texlive_install.sh
```
with
```yaml
install:
  - mkdir ./texlive/
  - 4-texlive
  - 4-texlive
  - 4-texlive
  - source ./texlive/texlive_install.sh
```
* Preferably you fork this repo so you can maintain your own build files with the right packages.

Note that sometimes `tlmgr` selects a broken mirror to download TeX Live from, so you get an error like `Output was gpg: verify signatures failed: eof`. Restarting the build will probably fix this, it will auto-select a different mirror. (Thanks to [@jason-neal](https://github.com/jason-neal/travis-ci-latex-pdf-texlive/commit/d48a5f92d2394f27371dd32c94a16415de499058) for testing this.)

You can also read a nice blog post by Jeremy Grifski ([@jrg94](https://github.com/jrg94)) about using the setup from this repo including the minted package at https://therenegadecoder.com/code/how-to-build-latex-with-travis-ci-and-minted/

### <a name="texlive-custom-fonts">Using custom fonts</a>

In order to enable Travis to use custom fonts, i.e. those fonts not provided by Ubuntu or TeX Live packages, it is probably easiest to push them with git.
An example is in this repo, with the file [src/fonts.tex](src/fonts.tex) and the fonts in [src/fonts](src/fonts).
You can do the following to use them on Travis:
* Make sure you have the fonts pushed to you GitHub repository
* Make sure that your `texlive_packages` file contains the packages `collection-fontsrecommended` and `luaotfload` 
* If you are starting from scratch, you can adapt [4-texlive/fonts-travis-yml/.travis.yml](4-texlive/fonts-travis-yml/.travis.yml). If you want to change your current .travis.yml, do the following:
    * In the `.travis.yml`, uncomment the following lines (make sure to change src/fonts to your own path if it is different)
    ```yaml
    before_install:
      - mkdir $HOME/.fonts
      - cp -a $TRAVIS_BUILD_DIR/src/fonts/. $HOME/.fonts/
      - fc-cache -f -v
    ```
    * Make sure to compile with lualatex or xelatex, for example if you previously had `texliveonfly main.tex` and `latexmk -pdf main.tex` in your `.travis.yml`, then change them to `texliveonfly main.tex --compiler=lualatex` and `latexmk -pdflua main.tex`.


## <a name="deploy">To automatically deploy pdfs to GitHub release</a>
### First time setup

We will add a configuration to the `.travis.yml` such that a pdf will be automatically uploaded to GitHub releases when you tag a commit, also see the [Travis documentation](https://docs.travis-ci.com/user/deployment/releases/).

* We will generate a GitHub OAuth key so Travis can push to your releases, with the important difference (compared to just gettting it via GitHub settings) that it's encryped so you can push it safely.
* (Windows) [Download ruby](https://rubyinstaller.org/downloads/) and at at end of the installation make sure to install MSYS including development kit.
* Run `gem install travis --no-document` to install the Travis Command-line Tool.
### For every new project
* Go to the directory of your repository, open the command prompt (Windows: <kbd>SHIFT</kbd>+<kbd>F10</kbd> <kbd>W</kbd> <kbd>ENTER</kbd>) and run `travis login --pro --github-token mytoken` where `mytoken` is a token from https://github.com/settings/tokens (or leave the `--github-token` flag out if you want to use your password). 
* Run `travis setup releases --pro` and leave File to Upload empty.
* Replace everything below your encryped api key with (changing the path to your pdf file, probably the same folder as your tex file is in)
```yml
  file: 
  - ./src/nameofmytexfile.pdf
  - ./otherfile.pdf
  skip_cleanup: true
  on:
    tags: true
    branch: master
```
* Commit and push.
* If you are ready to release, just tag and push.
