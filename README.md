[![Build Status](https://travis-ci.org/ssdoz2sk/ssdoz2sk_blog.svg?branch=master)](https://travis-ci.org/ssdoz2sk/ssdoz2sk_blog)

佈署在 github.io 上的 Blog 可以有兩種方案   

一種是利用 github.io 內建的 Jekyll，但只有支援一些的 [外掛](https://help.github.com/articles/adding-jekyll-plugins-to-a-github-pages-site/)，ruby 跟 Jekyll [版本](https://pages.github.com/versions/)也不是最新的，   

另外一種是在 client 端寫好，編譯完成後的 HTML 跟 CSS Push 到 Github Page，而這時的 Pages 就只是個單純的靜態網頁存放地點而已。   

<!--more-->

而且 Page 內的外掛有不支援自動產生 tags 的部分，所有的標籤需要手動增加，在 Github 上找到一個 [外掛](https://github.com/pattex/jekyll-tagging/pull/60)，依照 posts 上的 tags 屬性能自動產生，有了標籤就不用擔心哪天文章多的時候會變得難以整理。再加上我有三台電腦在使用，寫 Blog 的時間也不太固定，Win10 灌 ruby 又很麻煩(我懶)。   

再說，Jekyll 也沒像 wordpress 一樣有提供 Web 的編輯頁面，如果有時候只是為了更改幾個錯字的話，電腦得安裝 ruby + jekyll 才能更改 Blog 內的內容，也不是很方便。   

感謝 Travis-CI 提供免費的服務，利用它就可以在每次的 push 後，自動更新我的 Blog 了。不過還是有延遲啦，環境設定完成到編譯結束 commit 完花了兩分鐘左右就是了。不過就可以利用 Github 的 edit file 也可以在手機上改些小錯誤了。   

## Travis-CI 
持續整合(Continuous integration, CI)，是敏捷開發、極限開發所使用到的一種工具，透過持續化的自動建置，每次的 commit 都可以看到是否有問題產生，經由不同的測試腳本，可以自動使用不同版本的編譯器，測試是否會在哪一個版本會產生問題。   

## 如何使用 Travis-CI
打開 [Travis-CI](https://travis-ci.org/)，並利用 Github 帳號登入，然後打開 [Profile](https://travis-ci.org/profile) 勾選要監聽的 Project，如此一來 Travis-CI 就會對該專案的每個 commit 進行測試了。   

然後在專案的根目錄建立一個 `.travis.yml`，內容如下   

```yml
sudo: false
dist: trusty

language: ruby

rvm:
  - 2.3.1

before_install:
    - gem install jekyll bundle

install:
    - git clone --depth 1 https://github.com/ssdoz2sk/ssdoz2sk_blog.git ssdoz2sk_blog
    - cd ssdoz2sk_blog
    - bundle install
    - git clone --depth 1 https://$GH_TOKEN@github.com/ssdoz2sk/ssdoz2sk.github.io _site

script:
    - bundle exec jekyll build --incremental

after_success:
    - cd _site
    - git add -A .
    - git commit -m "update from travis $(date '+%Y-%m-%d %H:%M %z')"
    - git push --quiet
```

Travis-CI 的預設是系統是 Ubuntu 12.04，增加個 `dist:trusty` 可以指定成 14.04。
簡介一下過程，就是安裝 Jekyll ，只把本 Blog 最新版本的抓取下來，安裝相依賴的套件，然後把 Pages 的最新版本抓下來到 `_site`。   

Build Jekyll 產生靜態檔案，`script` 標籤內必須要有指令，不然會出現找不到 `rake` 的錯誤。   

## 從 Travis-CI Push 上 Github Pages
再來，編譯過後的該如何推上 Pages 呢？   

1. 首先先到 Github 上，打開帳號設定，選擇 [Personal access tokens
Generate new token](https://github.com/settings/tokens)   
打好相關的設定，這 Token 的用途，還有權限(只要 `public_repo` 就好)   
新增一個 new token，再來就會拿到一個 token ，請複製起來。   
![Github generate new token]({{ site.url }}/public/img/2017-06-09/github_generate_new_token.png)    

2. 到 Travis-CI 內剛剛打開的 Project ，點選右上角的設定，到 Environment Variables 增加一個環境變數，比如說 `GH_TOKEN` : `XXXXXXXXXXXXXXXXXXXXXXX`   
![Travis CI setting]({{ site.url }}/public/img/2017-06-09/travis_ci_setting.png)   
![Travis CI setting2]({{ site.url }}/public/img/2017-06-09/travis_ci_setting2.png)   

>  `git push` 請記得加上 `--quiet`，你總不希望你的 token 在網路上裸奔是吧！

現在可以推上 github ，然後 Travis-CI 會自動運行 script。在 Travis-CI 能看到編譯過程。出了問題也可以慢慢除錯。一切如果正常的話會出現 passing ， 如果這 Blog 還正常活著應該會看到 ![Build Status](https://travis-ci.org/ssdoz2sk/ssdoz2sk_blog.svg?branch=master)。

這篇會順便發在 [README.md](https://github.com/ssdoz2sk/ssdoz2sk_blog/blob/master/README.md)