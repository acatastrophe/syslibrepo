# Week 11: Installing Omeka

## Steps

1. Prepare the VM for the Omeka install
2. Create a new user and database for Omeka use
3. Download and install Omeka
4. Adjust the configuration and permissions
5. Complete the install through the web app

### Prepare the VM for the Omeka installation

We need two more programs and one module before we can install Omeka. First, download ImageMagick:

`sudo apt install imagemagick`

As noted previously, ImageMagick is a program for dealing with raster images. We already have a PHP module designed to interact with it.

We'll also need `unzip`, which will be used to unpack the program files later on. Since it comes as a `.zip` instead of a `.tar.gz` file, we can't unpack it like a tarball with `tar`.

`sudo apt install unzip`

(Of course, you can read the page for `unzip` first with `apt show unzip`, just to be on the safe side.)

Now, enable an Apache mod to allow for URL rewriting:

`sudo a2enmod rewrite`

`a2enmod` is an Apache command for enabling modules. Its counterpart is `a2dismod`.

Omeka will need the ability to write URLs in order to produce new pages for each item added to its collection.

Finally, restart Apache:

`sudo systemctl restart apache2`

### Create a new user and database for Omeka use

Omeka also needs MySQL access. We'll set that up before the program is installed.

First, become root and enter root's MySQL instance;

```
sudo su
mysql -u root
```

Make a database, a username, and a password for Omeka:

```
create user 'omeka'@'localhost' identified by 'XXPASSXX';
create database omeka;
grant all privileges on omeka.* to 'omeka'@'localhost';
show databases;
```

You'll see an Omeka database listed here. With everything we've done so far, there should be about seven databases that root can see; opacuser will probably only see three.

Quit MySQL with `\q` and exit root's instance with `exit`.

### Download and Install Omeka

If you're not already in the `var/www/html/` directory, go there now. This is where we'll download and extract the Omeka program.

This process will use `wget` to take the zip file from Omeka's git repo:

```
sudo wget https://github.com/omeka/Omeka/releases/download/v3.1/omeka-3.1.zip
unzip omeka-3.1.zip
```

This will extract a directory called `omeka-3.1.zip`, which you might want to change to `omeka` for simplicity's sake. If you choose to do that:

```
sudo mv omeka-3.1 omeka
```

### Adjust the Configuration and Permissions

Inside this new directory, there will be a file called `db.ini`. We need to add the MySQL database, username, and password in the appropriate fields. There's also a link 
to documentation about which fields do what--of note, the first field should be set to `localhost`. Check the documentation for what the other fields do, and whether or not you need 
to amend them. Personally, I left everything after the fourth field alone.

Now, change the permissions for the Omeka directory. This is the same process as what we did with Wordpress:

`chown -R www-data:www-data *`

Restart both Apache and MySQL:

```
sudo systemctl restart apache
sudo systemctl restart mysql
```

### Complete the Install Through the Browser

Again, navigate to `http://11.111.111.11/omeka` either in w3m or with a graphical browser. A graphical browser would be preferred, especially if you're planning to do additional work in 
Omeka after setup. There will be a web form to fill out here.

Omeka, once set up, manages content by using Dublin Core metadata and a tagging system. Find more information on Dublin Core [here](https://www.dublincore.org/).

### Notes

The only major issue I found during this process was a snag with DOM, which is disabled in some versions of Ubuntu by default but is usually enabled. There's a step in the Wordpress
 setup that enables DOM, but I didn't do it before I installed Omeka and had to handle it later in the pattern. If that issue arises a second time, DOM was enabled with:

`sudo apt-get install php-xml`

Again, this should be included in a previous step with Wordpress, but just in case...!
