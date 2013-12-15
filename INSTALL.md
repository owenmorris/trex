Installation on Linux
=====================

These are the steps I took to build Trex.  This has been tested on Ubuntu 13.04 x64

Notes:

Originally tried installing on Ubuntu 12.04 LTS x64 however got the error:
configure.ac:4: error: Autoconf version 2.69 or higher is required

I have logged into my server as root.  If you're logged in as another use you may need to use `sudo`.

Install Prerequisites:

    apt-get update
    apt-get upgrade
    apt-get install vim zip build-essential automake autoconf git-core subversion libssl-dev libtool
    reboot


Build and install Trex:

    cd /var
    git clone git@github.com:scripting/trex.git
    cd trex
    git submodule init
    git submodule update
    ./build.sh

Trex will tell you to set the library path location.  Note this for later when trex is run.  Install trex:

    cd build
    make install


Installation on Mac OS X
========================

You will need to install some prerequisite packages followed by installing some dependent packages required to compile the code.  

1. Install git from [Git for Mac Download](http://git-scm.com/download/mac)

2. Install XCode (this is required as the version of v8 included as part of the referenced git submodule can only be built with XCode, later fixes were included to allow v8 to be built with the command line tools).

3. Now build the prerequisite tools required to compile trex using Homebrew:

    `brew install autoconf automake libtool openssl libxml2`

4. Compile the trex code.  Follow the compilation instructions for Linux above. 
When setting the library path below change `export LD_LIBRARY_PATH` to 
`export DYLD_LIBRARY_PATH`

Running Trex
============

Ensure trex can find the libraries created during build (if installed under /var/trex) - modify as necessary:

    export LD_LIBRARY_PATH=/var/trex/deps/usr/lib 

or on Mac: 
    
    export DYLD_LIBRARY_PATH=/var/trex/deps/usr/lib 
    
Run trex as follows:
    trex https://raw.github.com/scripting/trex/master/opml/trexBoot.opml

Site loads at http://localhost:8080/

If you'd like to run this more long term you can use the command:

    nohup trex https://raw.github.com/scripting/trex/master/opml/trexBoot.opml &

and it will continue to run after you log out.

Test trex:

    curl -H "Host: dave.smallpict.com" http://localhost:8080

The HTML for <http://scripting.com> should be displayed

Point a domain name at it as well as a wildcard subdomain.  In these instructions I use the domain "example.com".  

    http://andrewshell.example.com:8080

In order to configure to configure to serve your own pages the worldoutline file bundled with Trex will need to be updated.

Serving your own outlines
=========================

First, create an outline using Fargo or another outliner.  Make it accessible
in a public location - either a dropbox public URL or hosted elsewhere that
the trex server is able to connect to via http.

To serve this outline using Trex you will need to edit the files bundled
in the github repo that hold the Trex configuration details.  The following
files will need to be edited under opml/ in the trex directory:

* trexBoot.opml
* trexWorldOutline.opml

You will also need to create an outline to hold the user directory.  See the
section called 'userDirectory.opml' below.

These files will need to be accessible using http so you will need to install
a webserver on your local machine and share out the files.  You can either
copy the files or symlink them to the place you have installed trex.  However,
if you do symlink you will need to ensure that your web server is configured 
to follow symlinks - by default they won't as a security measure.  

trexBoot.opml
-------------

Locate the line that contains the url of the world outline file under the trex 
responder at the bottom of the file (or search for var opmlURL).  Modify the
url to point to your copy of trexWorldOutline.opml - for example at
http://localhost/trexWorldOutline.opml

trexWorldOutline.opml
---------------------

Edit trexWorldOutline.opml and locate the headline titled 'userdirectory'.  It
will be an opml include that points to the main smallpicture directory of
named outlines.  Modify the url to the userDirectory.opml file you will 
create in the next section. 

userDirectory.opml
------------------

Create a new opml file called (by convention) userDirectory.opml.  Each 
headline in the outline should have text that is the single word which
will be included in the name of the outline eg. 'example'.  Edit the
node attributes so that the headline is an include node with an url that 
points to the public outline that is being served.  

This can be done from a text editor or outliner.  If you want to use Fargo
this can be done by creating a new outline and copying the file from 
~/Dropbox/Apps/Fargo to the directory serving your files from the web server.

Pointing Trex at your new config
--------------------------------

Run Trex and point it at your new trexBoot.opml.  For example, if you have
shared all of the files out from the document root of your web server, run it
as follows:

    trex http://localhost/trexBoot.opml &

Then use curl to test that the outline is being served properly (use the name
that you set up in userDirectory.opml - here 'example' is used):

    curl -H "Host: example" localhost:8080

If things aren't set up properly you should get an error message from Trex.
If not you should get html in the default format showing the rendered outline.
