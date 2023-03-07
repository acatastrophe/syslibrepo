# Making our Basic OPAC Secure and Web-Usable

## Steps
-Create a web-usable data entry page
-Create a PHP script to catch entries from the data entry page and convey 
them to MySQL
-Secure the page with a username and password

### Create a web-usable data entry page

Obviously, it's deeply annoying to enter items through the MySQL database 
in the CLI. Let's make a slightly less annoying entry method.

Make a new directory for this purpose under the html directory:

```
cd /var/www/html
sudo mkdir cataloging
```

Add a new index file:

```
cd cataloging
sudo nano index.html
```

Add the following code:

```
<!DOCTYPE html>
<html>
<head>
    <title>Enter Records</title>
</head>
<body>
    <h1>OPAC Library Administration</h1>

    <p>This is the library administration page for entering records into the OPAC.</p>
    <p>Please do not use this page unless you are an authorized cataloger.</p>

    <form action="insert.php" method="post">
        <label for="author">Author:</label>
        <input type="text" name="author" id="author" required><br><br>

        <label for="title">Book Title:</label>
        <input type="text" name="title" id="title" required><br><br>

        <label for="publisher">Publisher:</label>
        <input type="text" name="publisher" id="publisher" required><br><br>

        <label for="copyright">Copyright:</label>
        <input type="date" id="copyright" name="copyright">

        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

Obviously this can be switched around based on the fields that are 
allowed for in our MySQL database. If I add a rating field, for example, 
I can plug in a rating box here and collect that input! Just make sure 
that you also allow for that information in the next step. Otherwise 
it'll slip away into the digital aether.

### Create a PHP script to collect data from the webpage

We already defined a place for the data to go--the html we just made pushes 
data to a place called `insert.php`. Now we just need to populate that 
space with meaningful code that will transmit things to MySQL!

Start by making the file:

`sudo nano insert.php`

Drop in the following code:

```
<?php

// Load MySQL credentials
require_once '../login.php';

// Establish connection
$conn = mysqli_connect($db_hostname, $db_username, $db_password) or
  die("Unable to connect");

// Open database
mysqli_select_db($conn, $db_database) or
  die("Could not open database '$db_database'");

// Prepare and bind SQL statement
$stmt = $conn->prepare("INSERT INTO books (author, title, publisher, copyright) VALUES (?, ?, ?, ?)");
$stmt->bind_param("ssss", $author, $title, $publisher, $copyright);

// Set parameters and execute statement
$author = $_POST["author"];
$title = $_POST["title"];
$publisher = $_POST["publisher"];
$copyright = $_POST["copyright"];

if ($stmt->execute() === TRUE) {
    echo "New record created successfully";
} else {
    echo "Error: " . $stmt->error;
}

// Close statement and connection
$stmt->close();
$conn->close();

echo "<p>Return to the cataloging page: <a href='http://11.111.111.111/cataloging/'>http://11.111.111.111/cataloging/</a></p>";
?>
```

Obviously, change the dummy IP address to the one that matches your VM's IP.

Test out your catalog collection mechanism by adding new data!

### Make it secure

You need to set up some basic security for this page as soon as possible. 
Virtual machines are bombarded by access attempts constantly, and if you 
want this catalog to be any use, you have to keep the data inside it true.

The security method that Dr. Burns had us use involves the Apache2 htpasswd 
function. [Here's an article that he linked to from DigitalOcean.](https://www.digitalocean.com/community/tutorials/how-to-set-up-password-authentication-with-apache-on-ubuntu-18-04)

Set up a username like this:

`sudo htpasswd -c /etc/apache2/.htpasswd libcat`

This will prompt you to set a password and confirm it. Like with MySQL, the 
password will not be written out on the screen. This doesn't need to be 
terribly hard to remember to be more effective than nothing!

The web server needs to know about this change. To make the htpasswd apply 
to our online cataloging module, we need to change the config files. Open 
them here: 

`sudo nano /etc/apache2/apache2.conf`

Scroll down to find the following code block (at roughly line 173):

```
<Directory /var/www/>
  Options Indexes FollowSymLinks
  AllowOverride None
  Require all granted
</Directory>
```

You can either change `AllowOverride None` to `AllowOverride All`, or you can 
comment out `AllowOverride None`, like so:

`# AllowOverride None`

and add in `AllowOverride All` in its place. 

Write out this change and close the config file.

Now, we need an .htaccess file back in the html directory:

```
cd /var/www/html/cataloging
sudo nano .htaccess
```

Paste in this code:

```
AuthType Basic
AuthName "Authorization Required"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

Write out the file and check the config:

`apachectl configtest`

If you get back `Syntax OK`, you've done it right! Restart Apache2 and do a 
status check:

```
sudo systemctl restart apache2
systemctl status apache2
```

If you go back to your cataloging page (externalIP/cataloging/), a dialogue 
box will prompt you to enter the username and password that you set earlier.
That's `libcat` and the password.

### Notes

The only major issue I ran into this week was that I entered a title in the 
author field and vice versa several times in a row without realizing. I 
spent about4 20 minutes combing through the html and PHP files that 
controlled the online entry system before I rezlilsed what had 
happened. Otherwise, nothing particularly tricky this week!

On the other hand, I've been working on my own website this week, so I'm 
getting a little more practice with basic html and css. Maybe someday 
I'll make a full website with a catalogue like this one!




