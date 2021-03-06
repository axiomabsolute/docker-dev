# Docker Dev
Concept inspired by: https://github.com/bbartolome/docker-bash

Basic idea is to have a portable development environment using docker. I took
it a bit further and made everything layered (broke it down a lot). This way I
can build images for a Rust or Scala environment (or w/e language) from the
same neovim/tmux/powerline setup, without having to strip out the nodejs part
first.

To run, just:
```
docker run -ti aghost7/nodejs-dev:v0.10.38 bash
```

## Images
- `ubuntu-dev-base`: Ubuntu Trusty image with a few presets such as docker of
already installed. Might add docker-compose in there eventually. Not decided.
Main thing with this image though is that it downgrades from root to a regular
user. It is also configured to allow passwordless `sudo` just like those
nice vagrant images.
- `power-tmux`: Powerline + Tmux. Based from the `ubuntu-dev-base` image.
- `ubuntu-nvim`: NeoVim image. Based from `ubuntu-tmux`. Language agnostic vim
setup (no language-specific plugins in there).
- `nodejs-dev`: nvm + nodejs specific configurations. Tags available:
	- `v0.10.38`: Nodejs v0.10.38
	- `argon`: Nodejs LTS.
- `rust-dev`: NeoVim configuration and autocomplete for the Rust language. 
`stable` is the only current tag available.
- `py-dev`: Python configuration with autocomplete for python.
`aghost7/py-dev:2.7` is the only image available for now.
- `scala-dev`: Scala configuration with ensime server. It is strongly
recommended to keep the ivy cache somewhere (`~/.iv2/cache`) on your
host file system. Ivy is extremely slow at resolving dependencies.
There is only a `latest` tag for the scala image.
- `pg-dev`: Postgresql image with pgcli, a command line client with
autocompletions and syntax highlighting. There is also a web based client
([pgweb](https://github.com/sosedoff/pgweb)) that binds to the 8081 port. Tags
correspond to the Postgresql version, and the only version currently available
is Postgresql 9.3 (so there's just a `9.3` tag atm).
- `my-dev`: MySql image with mycli utility. There is only a `5.6` image.

## Vim Configuration
Vim configurations are broken down into three parts:
- `init.vim`: This is the equivalent for `.vimrc` in NeoVim.
- `plugin.vim`: This contains all plugins which will be installed using 
`vim.plug`.
- `post-plugin.vim`: Contains all plugin-specific configurations for Neovim.
Configurations which aren't plugin-specific reside in the `init.vim` file.

Breaking it down this way allows one to just run
`cat addition.vim >> $XDG_CONFIG_PATH/file` to add new plugins and configs for
specific programing languages and libraries.

## Calling Docker on the Host
The docker daemon run over a socket. The command line tool is just a client to
the daemon. In other words, if we make the socket connectable from within the
container we're in business.

For some reason it needs `privileged` to work as well.
```bash
docker run -ti --rm \
	--privileged \
	-v `readlink -f /var/run/docker.sock`:/var/run/docker.sock \
	aghost7/ubuntu-dev-base:latest \
	bash
```

## SSH Forwarding and Git
For ssh, just pass the socket over to the container.
```
docker run -ti --rm \
	-v $SSH_AUTH_SOCK:$SSH_AUTH_SOCK \
	-e SSH_AUTH_SOCK=$SSH_AUTH_SOCK \
	aghost7/ubuntu-dev-base:latest \
	bash
```
I like to avoid having to reconfigure git every time, so I mount a volume for
`.gitconfig`. `~/.ssh/known_hosts` is also anoying.

## Getting the Clipboard Working
Basically, X11 is built in a manner that allows sending display data over the
wire. I've managed to run GUI applications from a headless server through an
ssh connection in the past. The way I'm doing this is through the same old
unix socket trick for controlling the docker daemon that's on the host machine.

```bash
docker run -ti --rm \
	-e DISPLAY=$DISPLAY \
	-v /tmp/.X11-unix:/tmp/.X11-unix:ro \
	aghost7/ubuntu-dev-base:latest bash
```

On the host you'll need to disable one of the security features in X11.
```bash
xhost +si:localuser:$USER > /dev/null
```

Source: http://stackoverflow.com/questions/25281992/alternatives-to-ssh-x11-forwarding-for-docker-containers

## Complete Example Startup Script
This is what I use on my current machine to get it working with everything.

https://github.com/AGhost-7/dotfiles/blob/master/bin/dev

## TODO
- Upgrade to Xenial.
- Get coursier running on the scala image.
- Get something better for browsing git commits visually. Something like [gitv](https://github.com/gregsexton/gitv)
- https://github.com/junegunn/vim-journal ? Or some sort of markdown display
mode for vim.
