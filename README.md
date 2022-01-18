tsuki!'s own fork of gulag, with changes from a lot of different other instances.

# Setup
```sh
# clone the repository & init the submodules
git clone https://github.com/cmyui/gulag.git && cd gulag

# clone the submodules (oppai-ng)
git submodule init && git submodule update

# python3.9 is often not available natively
# https://github.com/deadsnakes/python3.9
sudo add-apt-repository ppa:deadsnakes/ppa

# install project requirements (separate programs)
sudo apt install python3.9 python3.9-dev python3.9-distutils \
                 mysql-server redis-server nginx build-essential certbot

# install pip for python3.9
wget https://bootstrap.pypa.io/get-pip.py
python3.9 get-pip.py && rm get-pip.py

# install gulag's python requirements
python3.9 -m pip install -U pip setuptools \
                         -r requirements.txt

# setup pre-commit's git hooks
# https://pre-commit.com/
pre-commit install

# if root, ignore this command
sudo su -

# create a new user (must be root)
mysql
create user 'username'@'localhost' identified by 'password';
grant all privileges on * . * to 'username'@'localhost';
flush privileges;
exit;

# create the database
mysql -u username -p
create database your_db_name;

# verify the database exists
show databases;

# import gulag's mysql structure
mysql -u username -p your_db_name < migrations/base.sql

# generate an ssl certificate for your domain (change email & domain)
sudo certbot certonly \
    --manual \
    --preferred-challenges=dns \
    --email your@email.com \
    --server https://acme-v02.api.letsencrypt.org/directory \
    --agree-tos \
    -d *.your.domain -d your.domain

# copy our nginx config to `sites-enabled` & open for editing
sudo ln -r -s ext/nginx.conf /etc/nginx/sites-enabled/gulag.conf
sudo nano ext/gulag.conf

# reload the reverse proxy's config
sudo nginx -s reload

# create a config file from the sample & open it for editing
cp .env.example .env && nano .env

# start the server
./main.py
```

# Directory Structure
    .
    ├── app                   # the server - logic, classes and objects
    |   ├── api                 # code related to handling external requests
    |   |   ├── domains           # endpoints that can be reached from externally
    |   |   |   ├── api.py        # endpoints available @ https://api.ppy.sh
    |   |   |   ├── ava.py        # endpoints available @ https://a.ppy.sh
    |   |   |   ├── cho.py        # endpoints available @ https://c.ppy.sh
    |   |   |   ├── map.py        # endpoints available @ https://b.ppy.sh
    |   |   |   └── osu.py        # endpoints available @ https://osu.ppy.sh
    |   |   |
    |   |   ├── init_api.py       # logic for putting the server together
    |   |   └── middlewares.py    # logic that wraps around the endpoints
    |   |
    |   ├── constants           # logic & data for constant server-side classes & objects
    |   |   ├── clientflags.py    # anticheat flags used by the osu! client
    |   |   ├── gamemodes.py      # osu! gamemodes, with relax/autopilot support
    |   |   ├── mods.py           # osu! gameplay modifiers
    |   |   ├── privileges.py     # privileges for players, globally & in clans
    |   |   └── regexes.py        # regexes used throughout the codebase
    |   |
    |   ├── objects             # logic & data for dynamic server-side classes & objects
    |   |   ├── achievement.py    # representation of individual achievements
    |   |   ├── beatmap.py        # representation of individual map(set)s
    |   |   ├── channel.py        # representation of individual chat channels
    |   |   ├── clan.py           # representation of individual clans
    |   |   ├── collection.py     # collections of dynamic objects (for in-memory storage)
    |   |   ├── match.py          # individual multiplayer matches
    |   |   ├── menu.py           # (WIP) concept for interactive menus in chat channels
    |   |   ├── models.py         # structures of api request bodies
    |   |   ├── player.py         # representation of individual players
    |   |   └── score.py          # representation of individual scores
    |   |
    |   ├── state               # objects representing live server-state
    |   |   ├── cache             # data saved for optimization purposes
    |   |   ├── services          # instances of 3rd-party services (e.g. databases)
    |   |   └── sessions          # active sessions (players, channels, matches, etc.)
    |   |
    |   ├── bg_loops.py           # loops running while the server is running
    |   ├── commands.py           # commands available in osu!'s chat
    |   └── packets.py            # a module for (de)serialization of osu! packets
    |
    ├── ext                   # external entities used when running the server
    ├── migrations            # database migrations - updates to schema
    ├── tools                 # various tools made throughout gulag's history
    ├── main.py               # an entry point (script) to run the server
    └── settings.py           # manages configuration values from the user
