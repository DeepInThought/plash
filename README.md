# plash

Plash is a swiss army knife for containers that easily turns into a machete. Current version is 0.1 alpha.


## Install
`pip3 install git+https://github.com/ihucos/plash.git`

*Api and configuration format unstable*


## Tutorial

Run nvim without installing it to your operating system.
```
$ plash ubuntu :add-apt-repository ppa:neovim-ppa/stable :apt neovim -- nvim myfile
```
Your home directory is mounted as home on the container. Building is cached. This feels like nvim is runned as any other application

You can import command line arguments from files
```
$ cat ./nvim
#!/usr/bin/env plashexec
ubuntu
nvim
:add-apt-repository ppa:neovim-ppa/stable
:apt
	neovim
$ plash @nvim
```
Note the shebang, after marking your ./nvim file executable you could directly run it and even put the file into your PATH. The idea of plash is to have only a very lightweight virtualization, programms run by it should have mostly access to all resources seen by "native" programs. (Currently plash is on top of docker, I want to change it to libcontainer/runc)

Here is another simple example of a plash file:
```
:apt
	package1
	package2

:layer # layering is explecit

:run touch myfile
```

Plash scripts can be seens as one dimensional lisp, 'layer' is actually an macro
```
:layer apt
	package1
	package2
	package3
```
Is the same as
```
:layer
:apt package1
:layer
:apt package2
:layer
:apt package3
:layer
```
Another macro is the action warp:
```
:warp run cp -r {./data_dir_at_host} /app/data
```

Build time mounts are supported
```
:mount .
:pwd .
:rebuild-when-changed
	myfile
	mydir
```

You can have build time arguments, of course rebuilding happens if they change.
```
:import-envs
  MYDIR
  PATH:HOME_PATH
:eval
	mkdir $MYDIR
	cd $MYDIR
```

Altough it can be done, plash does not try to be an general purpose language and relies on inline scripts

```
:script
	[ "$DEBUGTOOLS" = 1 ] && plash :pip ipdb
	true
```

```
:define touch
         #!/usr/bin/env python
         import sys, subprocess
         subprocess.check_call(
         ['plash', ':run', 'touch', sys.argv[1]])
:touch myfile
```

But this is actually just for quick one-shot functions. You can implement new actions by importing python modules that have callables registered with the plash.eval.register decorator. See stdlib.py for examples.
```
plash ubuntu --no-stdlib :import myplashlib :funcyfunc
```


### Using plash to create docker images

### Virtualenv replacement
One of Python's virtualenvs shortcoming is that packages often can not be compiled at another computer. With plash we can isolate all dependencies inside a container and still be very leightweight on the development side.

```
 $ plash --debian --apt binutils python-dev --pip-requirements ./requirements.txt --install ./python -- python
 Installed to ./python
```

This will give you an executable file named `./python` that can be commited to version control and should behave just like the python installed in your operating system. Behind the scenes the requierements are installed in an fresh debian system. Rebuilding happens automatically.




In case you wondered, the executable `./python` file is a wrapped plash call.
```
$ cat ./python
#!/bin/sh
plash --debian --apt binutils python-dev --pip-requirements ./requirements.txt -- python "$@"
```

