# FlickrCLI

A command-line interface to [Flickr](https://www.flickr.com/). Upload and download photos, photo sets, directories via shell.

## Custom Setup Instructions 

### Dropbox setup 

If you need a full Dropbox => Flickr sync:

```
cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
~/.dropbox-dist/dropboxd   # starts dropbox daemon, attempts to authorize. follow instructions
wget -O dropbox.py https://www.dropbox.com/download?dl=packages/dropbox.py
chmod a+x dropbox.py
./dropbox.py start
./dropbox.py status   # to check status of dropbox daemon
df -h   # to track disk usage as dropbox fills up. grab a coffee.
```

### PHP & Flickr CLI Setup 

```
sudo apt-get update
sudo apt-get install git
git clone https://github.com/TheFox/flickr-cli.git   # install flickr CLI
sudo apt-get install php php-curl php7.0-bcmath php-xml
cd flickr-cli
wget -O install-composer.php https://getcomposer.org/installer
php install-composer.php
php composer.phar install   # will install all dependencies
nano config.yml 
```

Get Flickr API key and secret from [here](https://www.flickr.com/services/apps/create/apply/).

Paste the following into the file then CTRL+X to save:

``` 
flickr:
    consumer_key: <paste key here>
    consumer_secret: <paste secret here>
```

`./bin/flickr-cli auth --config=config.yml` then follow the prompts

### Flickr CLI Usage

To upload a complete directory:
`./bin/flickr-cli --config=config.yml upload [PATH]`  

Or even better, have the completed files moved to another directory in case you need to re-run a directory that had failed files or stopped while in progress:

```
mkdir moved
./bin/flickr-cli --verbose --config=config.yml --move=/root/moved upload [PATH]
```

## Standard Installation

1. Clone from Github:

		git clone https://github.com/TheFox/flickr-cli.git

2. Install dependencies:

		composer install

3. Go to <https://www.flickr.com/services/apps/create/apply/> to create a new API key.
The first time you run `./bin/flickr-cli auth` you'll be prompted to enter your new consumer key and secret.

## Usage

First, get the access token:

	./bin/flickr-cli auth

### Upload

	./bin/flickr-cli upload [-d DESCRIPTION] [-t TAG,...] [-s SET,...] DIRECTORY...

### Download

	./bin/flickr-cli download -d DIRECTORY [SET...]

To download all photosets to directory `photosets`:

	./bin/flickr-cli download -d photosets

Or to download only the photoset *Holiday 2013*:

	./bin/flickr-cli download -d photosets 'Holiday 2013'

To download all photos into directories named by photo ID
(and so which will not change when you rename albums or photos; perfect for a complete Flickr backup)
you can use the `--id-dirs` option:

	./bin/flickr-cli download -d flickr_backup --id-dirs

This creates a stable directory structure of the form `destination_dir/hash/hash/photo-ID/`
and saves the full original photo file along with a `metadata.yml` file containing all photo metadata.
The hashes, which are the first two sets of two characters of the MD5 hash of the ID,
are required in order to prevent a single directory from containing too many subdirectories
(to avoid problems with some filesystems).

## Usage of the Docker Image

### Setup

To use this software within Docker follow this steps.

1. Create a volume. This is used to store the configuration file for the `auth` step.

        docker volume create flickrcli

2. Get the access token (it will create `config.yml` file in the volume).

        docker run --rm -it -u $(id -u):$(id -g) -v "$PWD":/mnt -v flickrcli:/data thefox21/flickr-cli auth

   or you can store the `config.yml` in your `$HOME/.flickr-cli` directory and use:

        mkdir $HOME/.flickr-cli
        docker run --rm -it -u $(id -u):$(id -g) -v "$PWD":/mnt -v "$HOME/.flickr-cli":/data thefox21/flickr-cli auth

### Usage

Upload directory `2017.06.01-Spindleruv_mlyn` full of JPEGs to Flickr:

    docker run --rm -it -u $(id -u):$(id -g) -v "$PWD":/mnt -v flickrcli:/data thefox21/flickr-cli upload --config=/data/config.yml --tags "2017.06.01 Spindleruv_mlyn" --sets "2017.06.01-Spindleruv_mlyn" 2017.06.01-Spindleruv_mlyn

For Docker image troubleshooting you can use:

    docker run --rm -it -u $(id -u):$(id -g) -v "$PWD":/mnt -v flickrcli:/data --entrypoint=/bin/bash thefox21/flickr-cli

### Paths

- `/app` - Main Application directory.
- `/data` - Volume for variable data.
- `/mnt` - Host system's `$PWD`.

## Documentations

- [Flickr API documentation](http://www.flickr.com/services/api/)
- [Docker documentation](https://docs.docker.com/)

## License

Copyright (C) 2016 Christian Mayer <https://fox21.at>

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. You should have received a copy of the GNU General Public License along with this program. If not, see <http://www.gnu.org/licenses/>.
