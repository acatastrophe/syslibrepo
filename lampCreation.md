# Week 7: Setting Up a LAMP Server

## Steps

1. Install Apache (or Apache2, in this case)
2. Install PHP
3. Configure PHP so it plays nice with Apache
4. Install MySQL
5. Make a Database
6. ???
7. Profit

### Installing Apache

*Apache is an open-source HTTP server created in 1995 and continually supported ever since. It's the 
most popular web server on the Internet (maybe)! Find it [here](http://httpd.apache.org).*

Be sure your system is up to date with

`sudo apt update

sudo apt upgrade`

Restart if needed with 

`sudo reboot`

We're using Apache2--see 
[Apache's compiling and installation instructions](http://httpd.apache.org/docs/2.4/install.html) for
package specifics.

To check the package information, use 

`apt show apache2`

To install, run

`sudo apt install apache2`

Dr. Burns recommends some checks after installation:

`systemctl list-unit-files apache2.service

systemctl status apache2`

If you see a green "Enabled," you're golden! Or green.

#### Hidden Move: Install w3m

A Dr. Burns recommendation: use the w3m browser to check on Apache's functionality. Get it with:

`sudo apt install w3m`

To use w3m, run either of the following:

`w3m 127.0.0.1

w3m localhost`

You'll see a default Apache webpage (or whatever you've decided to put there instead, if you've
already messed with the default page at some point!).

Exit w3m, like many programs, with 

`q`

To view the default page in a visual browser, get the server's public IP (in a Google-hosted VM, this
is in the Compute Engine>VM Instances location, listed alongside the name). Enter that into the browser
bar behind 

`http://`

and you can see it!

#### Make a Webpage

The default page for Apache2 is stored in the html directory upon install. Go to it with 

`cd /var/www/html/

ls`

You'll see index.html, which has the dummy text you saw with w3m, if you didn't skip that step.

To replace the default webpage, first copy it:

`sudo mv index.html index.html.original`

(Or don't. I'm not your dad.)

Remember that we're not in Kansas anymore, Toto, and we need to use `sudo` or the
Ubuntu gods will just ignore our commands.

Apache2 defaults to display whatever file is called "index.html," so to make a new one, just... make a
new one.

`sudo nano index.html`

Put a little html in there to make it look pretty. I used the following:

```
<html>
<head>
<title>My Apache2 Webpage!</title>
</head>
<body>

<h1>Welcome to my homepage!</h1>

<p>This is my Apache2 webpage, made using an Ubuntu virtual machine!</p>

</body>
</html>
```

Write it out. Now, when you visit your site (or when you visit the domain/index.html), you'll see
this shiny new stuff. index.html.original is also there--just tack its filename onto the end of the domain
in the URL bar!

### Installing PHP

*[PHP](http://php.net) is a general-purpose scripting language used for web pre-processing.*

Ready for the next block? Install both PHP and the LibApache package. LibApache is the bridge between
these two programs.

`sudo apt install php libapache-mod-php`

Restart Apache2.

`sudo systemctl restart apache2`

Check PHP by making a test file. Navigate back to the html directory:

`cd /var/www/html/`

And make a dummy file.

`sudo nano info.php`

Drop in the following code:

```
<?php
phpinfo();
?>
```

View this file at your external IP/info.php, like with the above Apache tests. You should see a big old
install page for PHP with system information.

### Configure PHP

Remember how I said that Apache2 looks for a file titled index.html? Yeah... if we're going to use PHP,
we need to change where Apache2 tends to go. That's relatively easy to do. The file for directory configuration
lives in the /etc/ directory. Go there:

```
cd /etc/apache2/mods-enabled/
ls
```

See the `dir.conf` file? It's what needs to be changed. First, make a copy:

`sudo cp dir.conf dir.conf.bak`

And open up the file to change its preferences.

`sudo nano dir.conf`

Change it to read

`DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm`

Check the configuration:

`apachectl configtest`

If it says Syntax Ok, you're good to go.

This process  changes which item Apache2 pulls first as the landing page for your
external IP. Originally, it was set to default to index.html before index.php, which
means what index.php won't ever show up as the landing page, since we already have an index.html. 

Restart the system:

```
sudo systemctl reload apache2
sudo systemctl restart apache2
```

#### Making an Index.php

By the way, if we want index.php to show up, we need to make one of those...

Go back to your html directory and make a file for it:

```
cd /var/www/html/
sudo nano index.php
```

Dr. Burns had us use a browser detector for the index:

```
<html>
<head>
<title>Browser Detector</title>
</head>

<body>

<p>You are using the following browser to view this site:</p>

<?php
echo $_SERVER['HTTP_USER_AGENT']. "\n\n";
$browser = get_browser(null, true);
print_r($browser);
?>

</body>
</html>
```

This should now show up at your external IP's index page! Check with w3m or a graphical browser.

### Installing MySQL

*MySQL is an open-source database software used in a variety ot enterprises, large and small.*

Get MySQL (specifically, MySQL Community Server) in the same way as the previous programs, with sudo:

`sudo apt install mysql-server`

Make sure it's working alright:

`systemctl status mysql`

To move on, we need to be the root user. Be careful from here on; you can seriously jack up
a machine by messing with it as root.

`sudo su`

To confirm your immense power:

`whoami`

MySQL wants you to log in any time you use it. It comes with a login for the root user, which we need 
to use now; then, we can make a non-root user, so we don't have to be root when we enter items in
the database.

`mysql -u root`

You should now be in MySQL! See the databases with:

`show databases;`

While inside MySQL, you need to end commands with `;` to run them. Keep it in mind.

Now, make a new user:

`create user 'name'@'localhost' identified by 'XXXXX';`

Replace the name with whatever name you want to use, and replace XXXXX with the password you want. If you
get a Query OK, you've made a new user! Note: whenever you log in as this user, your password entry will be 
hidden from the screen--you'll still be typing it, and the terminal will receive input, but it won't show your
password. Just hit return and it'll accept it. 

### Make a Database

Make a new database and test out how it all goes together! From here on, we're using class information, not general stuff, but the process remains the same.

```
create database opacdb;
grant all privileges on opacdb.* to 'name'@'localhost';
show databases;
```

Now you should have a database called opacdb, as well as a user who can fully edit the database. It's time
to stop being root!

```
/q
exit
```

You should be your own user again. Log back into MySQL in your shiny new user account:

`mysql -u name -p`

You'll be prompted to enter your password. Again, this enters without visual input. Check out your views of 
the databases and open up the one you want to use:

```
show databases;
use opacdb;
```

Now we're going to make a table called books, and make some fields to go in it.

```
create table books (
id int unsigned not null auto_increment,
author varchar(150) not null,
title varchar(150) not null;
copyright date not null,
primary key (id)
);
```

Check what it looks like with:

```
show tables;
describe books;
```

#### Add Records to the Database

Time to drop some book records into our database. Here's the sample data from Dr. Burns:

```
insert into books (author, title, copyright) values
('Jennifer Egan', 'The Candy House', '2022-04-05'),
('Imbolo Mbue', 'How Beautiful We Were', '2021-03-09'),
('Lydia Millet', 'A Children\'s Bible', '2020-05-12'),
('Julia Phillips', 'Disappearing Earth', '2019-05-14');
```

To see all of the stuff that just went into the database, run:

`select * from books;`

If you want to add a new column to the table, it will look something like this:

`alter table books add publisher varchar(75) after title;`

You can add to existing records with commands like this:

```
update books set publisher='Simon \& Schuster' where id='1';
update books set publisher='Penguin Random House' where id='2';
update books set publisher='W. W. Norton \& Company' where id='3';
update books set publisher='Knopf' where id='4';
```

The above adds data based on the fixed IDs generated by the database. These IDs are unique to each record and are produced as the 
database grows; when a record is deleted, its unique ID goes with it. That makes it a good marker for adding data.

Here are some other commands that can return specific data from the table without the user needing to read the whole thing:

```
select author from books;
select copyright from books;
select author, title from books;
select author from books where author like '%millet%';
select title from books where author like '%mbue%';
select author, title from books where title not like '%e';
```

### Install PHP/MySQL Support

To make the database information visible on the web, though pages like the ones we set up earlier, we need to install supports
that make MySQL and PHP play nice together. Get these modules with sudo:

`sudo apt install php-mysql php-mysqli`

Restart Apache2 and MySQL, as usual.

```
sudo systemctl restart apache2
sudo systemctl restart mysql
```

"In order for PHP to connect to MySQL, it needs to authenticate itself," says Dr. Burns. I have no clue what that really
means, but here's how to do it:

```
cd /var/www/html/
sudo touch login.php
sudo chmod 640 login.php
sudo chown :www-data login.php
ls -l login.php
sudo nano login.php
```

Basically, you've just created a file that has specific permissions that mean it's readable to Apache2, but not to any old
slob who accesses our files via the web. Now, we need to set credentials in that file. You've just opened the login file in nano, so
enter the following:

```
<?php // login.php
$db_hostname = "localhost";
$db_database = "opacdb";
$db_username = "name";
$db_password = "XXXXX";
?>
```

Obviously, "name" and "XXXXX" are your MySQL username and password from when we set up the user's MySQL login info. The database 
will be wahtever you named the database, if you named it something other than "opacdb."

Next, we make a file that will be a place for the OPAC to render on the web:

`sudo nano opac.php`

Put this in it:

```
<html>
<head>
<title>MySQL Server Example</title>
</head>

<body>
<h1>A Basic OPAC</h1>

<p>We can retrieve all the data from our database and book table
using a couple of different queries.</p>

<?php
// Load MySQL credentials
require_once 'login.php';

// Establish connection
$conn = mysqli_connect($db_hostname, $db_username, $db_password) or
  die("Unable to connect");

// Open database
mysqli_select_db($conn, $db_database) or
  die("Could not open database '$db_database'");

echo "<h2>Query 1: Retrieving Publisher and Author Data</h2>";

// Query 1
$query1 = "select * from books";
$result1 = mysqli_query($conn, $query1);
while($row = $result1->fetch_assoc()) {
    echo "<p>Publisher " . $row["publisher"] .
        " published a book by " . $row["author"] .
        ".</p>";
}

mysqli_free_result($result1);

echo "<h2>Query 2: Retrieving Author, Title, Date Published Data</h2>";
$result2 = mysqli_query($conn, $query1);
while($row = $result2->fetch_assoc()) {
    echo "<p>A book by " . $row["author"] .
        " titled <em>" . $row["title"] .
        "</em> was released on " . $row["copyright"] .
        ".</p>";
}

// Free result2 set
mysqli_free_result($result2);

/* Close connection */
mysqli_close($conn);
?>
</body>
</html>
```

This gives you some hard-coded returns from the database. We'll need to set up ways for the user to query the database later.

All that's left is to test the syntax for everything:

```
sudo php -f login.php
sudo php -f index.php
```

If those both return with Syntax Ok, you're set with a LAMP stack!

### Notes

This wasn't terribly hard--just kind of annoying to do. It would be nice to learn more about the privileges set with `chmod` and 
`chown`, but it's not necessary at the end of the day... Unless? 

Despite everything we did this week, I managed to keep my head and not screw anything up. That much can't be said for other weeks!
:)
