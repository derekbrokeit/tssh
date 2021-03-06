# Tunneled ssh (tssh)
Copyright 2012 by *Derek A. Thomas*

Working on large servers containing multiple machines hidden behind a
central server is a pain. I find myself taking time just to jump between
the different computers with an anoying middle man getting in the way.
That's where tunneling comes in, which is frequently use to forward
ports so that we can more easily connect from one machine to another.

    # the setup is kinda like this:
                                                               +----------------+
                     Time consuming multiple jumps             |                |
                                                       +-------o other computer |
      +-----------+            +-------------+         |       |                |
      |           |            |   central   |         |       +----------------+
      |  my local |            |-------------|         |
      |  computer +----------> |  gate       +---------+
      |           |            |  keeper     |         |
      |           |            |             |         |       +---------------+
      +------v----+            +-------------+         |       | remote server |
             |                                         +-------o  -----------  |
             |   using tssh, the connection seems direct       | Gotta get work|
             +------------->----------->------------>----------o done          |
                                                               +---------------+

With tssh, I hope that the whole act of tunneling can be greatly
simplified with the least amount of load or cost to your computers or
servers.

* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * 

## Installation

Installation is quite simple. Here's an example script that installs tssh.

    #!/bin/sh
    tmp=$(mktemp -d)
    git clone git@github.com:scicalculator/tssh.git $tmp
    pushd . > /dev/null
    cd $tmp/bin

    # now link the files to our bin directory
    for file in $(ls) ; do
        ln -s $PWD/$file $HOME/bin
    done
    popd > /dev/null

    rm -r $tmp
    # all done

You can copy it to any file usch as `tssh-install.sh` and run it like:

    $ chmod +x tssh-install.sh
    $ ./tssh-install.sh

## Usage

### Setup 

tssh uses a log file `~/.tsshrc` to hold a list of know servers and central
servers (currently limited to 1) that several machines are behind. The file can
be automatically generated like follows:
    
    # let's set up our server list
    $ tssh-setup
    tssh-setup: setup for server list
     - creating new file: ~/.tsshrc
     - append a star at the end of servernames
         for the central, port-forwarding server
     - Port is optional, but is used for port forwarding

     1.   Name:  central*
       address:  example.com
          user:  my_username
          port:
          msg(not a password) :

     2.   Name:  remote_server
       address:  198.168.1.100
          user:  some_username
          port:  7070
          msg(not a password):
              > tmux attach
     3.   Name:  ^C
     
    tssh-setup: 2 servers added

    # now let's print out the server list just to make sure
    $ tssh-setup -p
      ---- .tsshrc ----
    name                  |  address           |  user             |  port | message
    ********************************************************************************
    central*              |  example.com       |  my_username      |       | 
    remote_server         |  198.168.1.100     |  some_username    |  7070 | tmux attach
    ---- .tsshrc ----

    # Remeber, it is important to give attach a '*' to the computer acting as your
    # central gate. There are currently no checks in place to make sure you do this
    # since you can add them in any order. 

We're all setup now to bounce to remote_server directly! Time to get on
over there.

### remote connection

Now to get to the meat of it, the actual `tssh` program. With this
program you can do some interesting things.

  1. First, we can connect directly to the server, which automatically
  runs our default message. This setups up a tunnel through the
  "central" computer to "remote_server" on the remote LAN. Also, when
  you disconnect, it will automatically free up your port (e.g. 7070 or
  whatever you set it) when you are done.

        # note, we can actually use smaller parts of the name as long as it is unique
        $ tssh remote
        tssh: remote
            tunnel -> remote_server
        # this attaches to the default tmux session on remote server

  2. A new message can be sent.

        $ tssh remote "echo hello ; sleep 5 ; echo byebye"
        tssh: remote_server
            message: echo hello ; sleep 5 ; echo byebye
            tunnel -> remote_server
        hello
        byebye
        $


  3. Also, we can setup a proxy server to our central node. This is
  helpful when your remote server has access to certain websites that
  you want to to gain access to from outside of the LAN.

        $ tssh -P 5050
        tssh: Proxy to central* > port 5050
        my_username@central: ~ > 
        # as long as your connection to central is open you 
        # can set your browser's proxy settings to localhost:5050 

### file-system mounting

For those times you need to interact directly with your remote computer's file
system, I've got you covered. To use this, you must install `sshfs`, which is
available on most package distributions. By default, sshfs connects are mounted
in `~/sshfs/${server-name}`, but the directory can be customized with the
environent variable `SSHFS_DIR`, which will be a place to mount your remote
file systems.

    # ok, now let's get going
    $ tssfs remote
    tsshfs: successfully mounted /Users/user_name/sshfs/remote_server
    # now you can go there and see what's available
    $ cd ~/sshfs/remote_server
    $ ls
    another_dir  dev  some_file
    $ tree
    ├── another_dir
    ├── dev
    │   └── something.git
    │        └── README.md
    └── some_file
    # now let's unmount it (you can do this from anywhere outside of the directory)
    $ cd
    $ tumount remote
    tumount: successfully unmounted /Users/user_name/sshfs/remote_server

* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * 

## license 

Copyright (c) 2012 Derek Ashley Thomas

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
