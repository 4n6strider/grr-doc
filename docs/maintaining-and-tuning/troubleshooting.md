# The upstart/init.d scripts show no output

When I run an init.d script e.g. "/etc/init.d/grr-http-server start" it
does not show me any output.

Make sure that the "START" parameter in the corresponding default file,
e.g. "/etc/default/grr-http-server", has been changed to "yes".

# I cannot start any/some of the GRR services using the init.d scripts

When it run an init.d script e.g. "/etc/init.d/grr-http-server start" it
indicates it started the service although when I check with
"/etc/init.d/grr-http-server status" it says it is not running.

You can troubleshoot by running the services in the foreground, e.g. to
run the HTTP Front-end server in the foreground:

    sudo grr_server --start_http_server --verbose

# Any/some of the GRR services are not running correctly

Check if the logs contain an indication of what is going wrong.

Troubleshoot by running the services in the foreground, e.g. to run the
UI in the foreground:

    sudo grr_server --verbose --start_ui

# Cannot open libtsk3.so.3

error while loading shared libraries: libtsk3.so.3: cannot open shared
object file: No such file or directory

The libtsk3 library cannot be found in the ld cache. Check if the path
to libtsk3.so.3 is in /etc/ld.so.conf (or equivalent) and update the
cache:

    sudo ldconfig

# Cron Job view reports an error

Delete and recreate all the cronjobs using GRR console:

    aff4.FACTORY.Delete("aff4:/cron", token=data_store.default_token)
    from grr.server.aff4_objects import cronjobs
    cronjobs.ScheduleSystemCronFlows(token=data_store.default_token)

# Protobuf 2.5.0

GRR only works with the Protobuf library v2.5.0 or newer. If your
distribution comes with an older version, follow the instructions below:

Remove any version of protobuf that is older than 2.5.0.

Note that this will also remove a couple of dependencies:

    sudo apt-get remove libprotobuf python-protobuf

To build Protobuf from source, install the following packages on a
Debian-based system:

    sudo apt-get install build-essential uuid-dev zlib1g-dev python-dev

Download protobuf version 2.5.0 or later from
[protobuf](http://code.google.com/p/protobuf/)

Extract the package:

    tar xfv protobuf-2.5.0.tar.gz
    cd protobuf-2.5.0

Build and install libprotobuf:

    ./configure --prefix=/usr/
    make
    sudo make install
    sudo ldconfig

Build and install the Python bindings:

    cd python
    python setup.py build
    python setup.py test
    sudo python setup.py install

# Missing Rekall Profiles

If you get errors like:

    Error loading profile: Could not load profile nt/GUID/ABC123...
    Needed profile nt/GUID/ABC123... not found!

when using rekall, you’re missing a profile (see the [Rekall
FAQ](http://www.rekall-forensic.com/faq.html) and
[blogpost](http://www.rekall-forensic.com/posts/2014-02-20-profile-selection.html)
for some background about what this means).

The simplest way to get this fixed is to add it into [Rekall’s list of
GUIDs](https://github.com/google/rekall-profiles/blob/gh-pages/v1.0/src/guids.txt),
which is of great benefit to the whole memory forensics community. You
can do this yourself via a pull request on
[rekall-profiles](https://github.com/google/rekall-profiles), or simply
email the GUID to <rekall-discuss@googlegroups.com>. Once it’s in the
public rekall server, the GRR server will download and use it
automatically next time you run a rekall flow that requires that
profile. If your GRR server doesn’t have internet access you’ll need to
run the GetMissingProfiles function from the GRR console on a machine
that has internet access and can access the GRR database, like this:

    from grr.server import rekall_profile_server
    rekall_profile_server.GRRRekallProfileServer().GetMissingProfiles()