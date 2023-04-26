# Week 12-13: Koha Installation Process

## Steps

1. Set up a new VM act as the Koha server
2. Install Koha on the new VM
3. Configure Koha through the web interface
4. Learn Koha!

### Set Up a New VM as the Koha Server

Koha is an open-source ILS software suite that's often used by small and medium-
sized public libraries, though it also sees application in university libraries and
 other situations. It has an active development and a community of users and 
administrators on Element (and elsewhere). Koha will fit nicely on top of the existing
 web server that we've made, and will offer us a public interface, a staff interface, 
and much more.

First, return to the Google Cloud console, where the previous VM was set up. Make a new 
instance with the following settings:

-E2 series
-2 vCPU, 4 GB memory
-Same operating system as the original VM (Ubuntu 20.04)

Dr. Burns has a [guide](https://cseanburns.net/WWW/systems-librarianship/05-using-gcloud-virtual-machines.html)
 to gCloud VMs earlier in the course textbook. For further setup information, see that 
section.

Now, we need to create a firewall rule in order to allow for staff access through a 
specific HTTP port. Normally, HTTP traffic proceeds through port 80 under normal 
circumstances; we need staff access to happen through port 8080, and therefore we 
need to set the firewall to allow port 8080 access.

Go to `gCloud > VPN Network > Firewall` and create a firewall rule. Name it whatever 
you like and describe it however you like. Set its targets to `All instances in the 
network`, the Source IPv4 ranges to `0.0.0.0/0`, and the specified protocols and ports
 to `TCP`, then add `8080` to the Ports box. Click create, and you're done!

Now that the gCloud settings are prepared, log into the new machine and fully update it:

```
sudo apt update
sudo apt upgrade
```

Next, clean up the disc to help save space:

```
sudo apt autoremove -y && sudo apt clean
```

Then, we need to get GnuPG2, which is an encryption tool (plus more). Find more 
information about the program [here](https://launchpad.net/ubuntu/+source/gnupg2). Get 
the program like this, then reboot the machine to ensure everything runs well:

```
sudo apt install gnupg2
sudo reboot now
```

Finally, we want to add the Koha repository to this machine. That way, running `sudo 
apt update` will get information from remote repos. Dr. Burns recommends logging in as 
root from here on in:

`sudo su`

Remember, act carefully when you act as root.

To add the Koha repo to this server, run the following:

```
echo 'deb http://debian.koha-community.org/koha stable main' | sudo tee /etc/apt/sources.list.d/koha.list
wget -q -O- https://debian.koha-community.org/koha/gpg.asc | sudo apt-key add -
```

The first command will get the repo; the second uses GnuPG2 to add a digital signature
 to verify the repo.

Now the server is set up, and we can install and configure Koha!

### Install Koha

This step should be relatively short in comparison. We already have the Koha repo on 
this VM, so we should be able to get Koha from its repo. Run the following:

```
apt update
apt show koha-common
apt install koha-common
```

This updates the repository, outputs the package information for the program, and 
finally, downloads the program. This will take a few minutes, so be patient.

#### Configure Koha in the VM

Substep here: there's some configuration to be done on Koha before we get to the web 
interface. Start by changing the port settings in the `koha-sites.conf` file:

```
nano /etc/koha/koha-sites.conf
```

Change the line reading `INTRAPORT="80"` to `INTRAPORT="8080"`. Write out and close.

Now, Koha needs a MySQL instance. Get the program, then set up a root user and password 
protect it:

```
apt install mysql-server
mysqladmin -u root password bibliolib1
```

Now, some settings in Apache2 need to be changed. We didn't download that to this VM, 
but it comes packaged with Koha, so it's been downloaded already. 

```
a2enmod rewrite
a2enmod cgi
systemctl restart apache2
```

These settings allow for URL rewriting and Common Gateway Interface use, so user 
requests may be used to execute programs. We also restarted the program to make sure 
changes took.

Now, create a Koha database:

```
koha-create --create-db bibliolib
```

And tell Apache2 to listen on port 8080 (the port we specified for allowable web traffic 
earlier on):

```
nano /etc/apache2/ports.conf
```

Add `Listen: 8080` and restart Apache2 again with `systemctl restart apache2`.

Finally, we want to do a few things. Per [Dr. Burns](https://cseanburns.net/WWW/systems-librarianship/20-install-koha.html)
: "disable the default Apache2 setup, enable traffic compression using deflate, enable
 the bibliolib site, and then reload Apache2's configurations and restart again."

```
a2dissite 000-default
a2enmod deflate
a2ensite bibliolib
systemctl reload apache2
systemctl restart apache2
```

### Configure Koha on the web interface

Most of the rest of our Koha setup happens through the browser. Again, this can be 
performed through w3m, but I recommend a graphical browser because of how complicated 
Koha is.

Find the Koha default username and password here:

`nano /etc/koha/sites/bibliolib/koha-conf.xml`

The data will be under `<config>`, in lines 257 and 258. Use CTRL+_ to go to the 
correct line. 

Navigate to the VM's IP address:8080 to start (so, `http://11.111.111.111:8080`). Make 
sure you're not using HTTPS for this!

Run through the installation. The Koha community has [documentation](https://koha-community.org/manual//22.11/en/html/installation.html)
on the questions during this process--most of it is self-explanatory, though.

After you're done with the installer, enable the public OPAC like so:

`More > Administration > Global System Preferences > Opac > OpacBaseURL`

Enter `http://11.111.111.111` in this field, and click save.

### Learn Koha

I did a little poking around under the hood, but not a whole lot... we'll
 save this step for next week :)

### Final Comments

After all this time in the command line, Koha is kind of a wild ride! It's a 
very complicated program, so it's hard to get a hang of, but again, we'll 
talk about that later on.
