# Week 10: Installing Wordpress

## Steps

1. Make sure the VM is ready to accept Wordpress
2. Get and unpack the files
3. Create a MySQL database and user to be used with Wordpress
4. Configure Wordpress and change file ownership
5. Install through the web app

### Make sure the VM is ready

We do this for completely obvious reasons. Start out by ensuring that the machine itself doesn't need any updates:

```
sudo apt update
sudo apt upgrade
```

If you installed PHP and MySQL pretty recently, this shouldn't be an issue, but just to be safe, ensure you have the right versions of each program. You want 7.4
 or better on PHP and 5.7 or better on MySQL.

```
php --version
mysql --version
```

Several PHP modules are necessary to make Wordpress operate at its full potential:

`sudo apt install php-curl php-xml php-imagick php-mbstring php-zip php-intl`

Respectively, these modules are cURL, which communicates between many different kinds of servers; a module for XML support; ImageMagick, which deals with raster images; 
Multibyte String, which gives us more than just UTF-8 characters; a zipper/unzipper; and an internationalizer extension that does a whole bunch of shit.

Once these are done downloading, restart Apache2 and MySQL. 

```
sudo systemctl restart apache2
sudo systemctl restart mysql
```

### Get and unpack the Wordpress files

Once your stage is set, you'll download the Wordpress package. As opposed to getting it through `apt`, this one will be taken from its web presence.

```
cd /var/www/html
sudo wget https://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```

This will create a `wordpress` directory in the `html` directory. Its path is `/var/www/html/wordpress`

### Create a MySQL database and user for use with Wordpress

Obviously, if we want our Wordpress site to have anything to do with a database, we need to give Wordpress access to MySQL. There's documentation in Wordpress for how to do this,
 but it relies on using phpMyAdmin, which is graphical. This process avoids introducing a graphical element and an extra security risk.

Become root and log into root's MySQL instance:

```
sudo su
mysql -u root
```

Make a new user for Wordpress:

```
create user 'wordpress'@'localhost' identified by 'XXPASSWORDXX';
create database wordpress;
grant all privileges on wordpress.* to 'wordpress'@'localhost';
show databases;
```

You should see `wordpress` among the existing MySQL databases. Quit MySQL with `\q` and exit root's user instance with `exit`.

### Configure Wordpress and change file ownership

As with most of the other programs previously installed on this VM, we need to change the config file so that the logins we set up work with Wordpress.

If you aren't already, navigate to the wordpress directory:

`cd /var/www/html/wordpress`

Wordpress looks for a file called `wp-config.php`. The blank install doesn't come with a file by that name, but it does come with `wp-config-sample.php`. Copy it to the correct file, 
leaving the old sample behind, and open the new file with nano:

```
sudo cp wp-config-sample.php wp-sample.php
sudo nano wp-config.php
```

There will be several fields near the top that look like this:

```
define( 'DB_NAME', 'db_name' );
```

In the second section for each of these fields, add the names of your database, username, and password that were defined in the last step. The host will be `localhost`. Don't change
 the charset.

Then, at the end of the config file, add this line:

```
define( 'FS_METHOD','direct');
```

This disables FTP uploads to the site. FTP is a plaintext--ergo, unencrypted--protocol, and won't be secure. Best to avoid the problem from the get-go.

Finally, exit the config file and run the following command:

`sudo chown -R www-data:www-data *`

The `chown` command changes user permissions. The `-R` operator is a recursive change, and the `www-data:www-data` operator defines the user as a member of Apache's web group.
 (This is a really scuffed explanation... it's right at the edge of my ability to explain lol.)

#### Optional Step: Rename your Wordpress directory

If you want your Wordpress page to be at a different address other than `http://11.111.111.11/wordpress`, you'll need to change the name (and therefore, the path) of the directory 
you keep all the Wordpress stuff in. It's easy enough:

```
cd /var/www/html
sudo mv wordpress blog
```

Obviously `blog` can be swapped with whatever else you want the directory to be named.

### Install through a web app

Either return to your native graphical interface, or use `w3m` to navigate to the webpage. It's at your site IP /wordpress, or whatever you renamed the directory to, if you decided to do that.

`http://11.111.111.11/wordpress`

You'll see a setup page here. Make sure to assign a unique username and password here. Wordpress will offer you a randomized strong password--it's a good idea to take that, or to 
make a different strong password, as you want your web layer to be as secure as possible. I chose to copy my username and password into a `login-info.txt` file elsewhere in my machine.

#### Final notes from Dr. Burns

Dr. Burns specifies two extra items:

1. Email isn't an element in this setup process, since it's kind of a pain to set up an email server correctly. We would also need a domain name for that, and we're clearly just
 working off the IP, anyway.
2. Make sure you use http instead of https for this site. Https also usually requires a domain name.

### Notes for this week

Not particularly hard! I didn't have any major issues with Wordpress setup. On to Omeka! 
