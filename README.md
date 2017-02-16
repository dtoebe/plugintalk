Phoenix Gophers Go 1.8 Release Party talk: Plugins


To biuld:

First clone this repo and `cd` into it

```Shell
$ docker build -t asanzdiego/markdownslides .

$ docker run -it -v ${PWD}/doc:/home/markdownslides/doc asanzdiego/markdownslides ./build.sh max doc
```

Then cd in to `doc/export`

Run a simple HTTP server

```Shell
$ python -m SimpleHTTPServer
```


Created with https://github.com/asanzdiego/markdownslides