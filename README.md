# tridentnpl.github.io

# Test on your local computer

## install 

1. install ruby

ubuntu:
```shell
sudo apt update
sudo apt install ruby-full ruby-bundler
```
> Don't execute `sudo apt install ruby`,it may cause errors in installing jekyll.

2. install jekyll and bundler

ubuntu:
```shell
sudo gem install jekyll
sudo gem install bundler
```


3. install jekyll dependencies

Enter the project root folder.

ubuntu:
```shell
sudo bundle install
```

## run

```shell
bundle exec jekyll build --watch
bundle exec jekyll serve -w
```

Open the website `http://127.0.0.1:4000/` in your browser

