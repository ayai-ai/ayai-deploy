# Running Ayai, the research MMORPG

## The main parts involved

You need to run two servers to run the game: the front-end server and the back-end server. The front-end is simply HTML and JavaScript that are served as static files, using a Python script. The back-end is written in Scala and contains the game logic and database access.

The servers were set up to run with Vagrant, which runs these servers inside of a virtual machine. Vagrant is supposed to make it easier to run the servers on any platform.

`ayai-frontend` and `ayai-backend` are [Git submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) in this repository. To properly initialize them, when cloning this repository, add the `--recursive` flag to `git clone`.

## Instructions

(The script `launch_ayai` is meant to automate most of this. But before trying to run it, read the comments at its top. The script has not been confirmed to work yet.)

Install http://www.vagrantup.com/ and install Ansible (`brew install ansible` on OS X, `sudo apt-get install ansible` on Debian based machines). If you are using Windows you still need to check this directory out, but you will not be able to use Vagrant/ansible. You should instead create a VirtualBox VM with Ubuntu 12.04, install ansible, and run `provisioning/local.sh` instead.

Run `git clone http://github.com/ayaigame/ayai-deploy.git` and change into that directory.

Run `vagrant up`. This will download and setup a basic Ubuntu 12.04 LTS virtual machine. It will then attempt to provision. This will fail the first time halfway through. Run `vagrant provision` to restart the process, which should succeed this time.

This process will take some time as it is setting up a server suitable for Ayai as well as some general backend trickery to make development easier.

It will also attempt to compile the Ayai backend as the last step. This may take a long time and if you wish to see its progress, I would Ctrl-C out of the provisioning and manually build it myself (this will happen naturally following the next steps).

After this, run `vagrant ssh`. You are now ssh'ed into the Vagrant virtual machine. `cd /vagrant/` to see a directory that is synchronized with the directory you checked out into.

Firstly, run `sudo service nginx restart`. For some reason, ansible believes nginx to have been restarted when it hasn't yet (possibly due to a non-standard Websocket friendly version being used).

Secondly, cd into (inside either /vagrant/ or wherever you checked out ayai-deploy to) ayai-frontend. Edit the file `js/ayai/Ayai.js` on the Connection line, to now point to `ws://192.168.100.10/ws`. Save this file.

Finally, cd into (inside of /vagrant/, not on your own machine) ayai-backend. Run `sbt run`. If you cancelled the compile process earlier, it will attempt to compile here. After several messages you will eventually see `Running SockoServer`. This means that the database, generated worlds, and every remaining step has been completed.

Navigate in your browser (on your local machine, not inside of vagrant) to `192.168.100.10/debug.html`. If all goes well, you should now have a copy of Ayai running on your machine.

When you are done with development, switch to wherever you checked out ayai-deploy and run `vagrant halt`. This will turn off the Vagrant virtual machine and prevent it from hogging any resources.

The reason we chose to do this was to keep dependencies across multiple platforms synchronized and since there are CORS issues with running a web server/app server on two different domains. This, of course, can be circumvented, but has been left as an exercise to the reader.

## Instructions for small-scale testing

When testing small changes, you can run the servers without Vagrant. Launching the servers should be quicker in this case. However, when the server is run outside of Vagrant, the admin interface (which provides account creation, character creation, and editing game objects) will not work.

You can run a limited version of the back-end server with `sbt run`, from your IDE, instead of running the server in Vagrant.

You can run the front-end with the instructions in that repository’s README. Basically, just `python run.py` to statically serve those files.
