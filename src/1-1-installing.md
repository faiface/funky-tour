# Installing

> **Note.** These instructions are for Linux. If you're on another operating system, you can try to follow them doing analogous actions on your OS.

Funky is currently implemented in [Go](https://golang.org/) and there are no pre-compiled binaries provided yet.

Therefore, the first step is to install the Go language either from the [official website](https://golang.org/) or from your package manager.

Go uses `$GOPATH` directory to download packages and install Go programs. If you installed a recent version of Go, you don't have to set it up, it defaults to `~/go`.

Next you need to download Funky. You do that with the Go package manager:

```
$ go get -u github.com/faiface/funky/...
```

This downloads Funky repository to the `$GOPATH/src` directory and installs the binaries to the `$GOPATH/bin` directory.

To make the binaries easily runnable from the command line, you need to add `$GOPATH/bin` (or `~/go/bin` if `$GOPATH` is not set) to the `$PATH` environment variable. Add this to your `~/.bashrc` file (or an equivalent of that for your shell):

```
# replace $GOPATH with /home/yourname/go if $GOPATH is not set up
export PATH=$PATH:$GOPATH/bin
```

Now, one last thing. Funky uses the `$FUNKY` environment variable to indentify the location of the standard library. Add this to your `.bashrc` file:

```
export FUNKY=$GOPATH/src/github.com/faiface/funky/stdlib
```

**And that's it!** Let's get ready for some action!
