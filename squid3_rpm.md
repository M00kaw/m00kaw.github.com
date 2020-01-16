
# Fedora 17 - Create Squid-RPM with MySQL Support

So, this was actually an article i wrote back in 2013, on my old blog. (Remember the time, when we were using x86 software?)

I still think it's viable, so I'm re-publishing it on my new blog.

## Here we go:

So, the story goes:

I was trying to setup squid3 on my Fedora-server and it should authenticate users via a MySQL Database. According to the squid site

(http://wiki.squid-cache.org/ConfigExamples/Authenticate/Mysql), the requriment was:”Make sure squid is compiled with –enable-basic-auth-helpers=DB option.”

So this is how it all started, and now I’m trying to explain how to check and add flags to .rpm package.

First off – let’s search for the package and retrieve the info

    $ yum search squid
    $ yum info squid

Now that we know the package is in the repository, we need to check if it has been compiled with the right flags. Add the correct folder-structure:

    $ mkdir -p rpmbuild/SOURCES
    $ cd rpmbuild/SOURCES/

Install yum-utils, so we’re able to download the .rpm

    $ yum install yum-utils

Let’s download the source-rpm:

    $ yumdownloader --source squid


Extract the specifications:

    rpm2cpio squid-3.2.9-1.fc17.src.rpm | cpio -i

Open up the .spec file (with nano) and check the flags under (line 120): “%configure \”

    nano -c squid.spec

The flag we need isn’t added “–enable-basic-auth-helpers=DB option.”
So, we’re going to add that flag manually.
On line 144 (after the last flag), type the following:

    -- enable-basic-auth-helpers=DB \

Save the file as “squid.spec” (in nano ctrl +o + Enter/Return and ctrl + x + enter/Return)

We need to install some packages to create our own .rpm
according to: http://fedoraproject.org/wiki/How_to_create_an_RPM_package#Preparing_your_system

    $ yum install @development-tools
    $ yum install fedora-packager

(DO NOT BUILD RPM AS ROOT!!)


from the man-page of rpmbuild:

    -bb    Build a binary package (after doing the %prep,  %build,  and  %install stages).

Let's give it a try:

    $ rpmbuild -bb squid.spec

My Terminal outputs:

    [m00kaw@teh-geek SOURCES]$ rpmbuild -bb squid.spec --nobuild
    error: Failed build dependencies:
    openldap-devel is needed by squid-7:3.2.9-1.fc17.i686
    pam-devel is needed by squid-7:3.2.9-1.fc17.i686
    db4-devel is needed by squid-7:3.2.9-1.fc17.i686
    expat-devel is needed by squid-7:3.2.9-1.fc17.i686
    libxml2-devel is needed by squid-7:3.2.9-1.fc17.i686
    libcap-devel is needed by squid-7:3.2.9-1.fc17.i686
    libecap-devel is needed by squid-7:3.2.9-1.fc17.i686
    libtool-ltdl-devel is needed by squid-7:3.2.9-1.fc17.i686
    cppunit-devel is needed by squid-7:3.2.9-1.fc17.i686

Install all the dependencies:

    $ yum install openldap-devel pam-devel db4-devel expat-devel libxml2-devel libcap-devel libecap-devel libtool-ltdl-devel cppunit-devel

Let's try again:

    $ rpmbuild -bb squid.spec

Wait some time….
Wait some more time…
Wait some more, mor time…

Done…..

cd back to rpmbuild/ and check the folder, where the .rpm is located:

    $ ls -l RPMS/i686/
    squid-3.2.9-1.fc17.i686.rpm            
    squid-sysvinit-3.2.9-1.fc17.i686.rpm
    squid-debuginfo-3.2.9-1.fc17.i686.rpm

cd into that folder and install squid

    $ cd RPMS/i686/
    $ su -c 'rpm -Uhv squid-3.2.9-1.fc17.i686.rpm'

confirm squid is installed locally

    [m00kaw@teh-geek i686]# which squid
    /usr/sbin/squid

start squid with systemctl:

    #start squid
    $ systemctl start squid

    #check status:
    $ systemctl status squid

Check that the right flag is enabled (–enable-basic-auth-helpers=DB option)

    $ squid -v
    Squid Cache: Version 3.2.9
    configure options:   '--build=i686-redhat-linux-gnu'  
    <lots of stuff>
    '--enable-basic-auth-helpers=DB'
    <lots of stuff>

As we can see – the flag is indeed enabled!


SUCCESS
