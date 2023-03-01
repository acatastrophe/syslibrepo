#Making a Basic OPAC

##Steps
1. Make a PHP script for searching
2. Make an HTML page that allows the user to interact with the PHP script
3. Put more stuff in the OPAC for the user to look for

###Making a PHP Script

*This part is easy. Dr. Burns asked ChatGPT for a bit of script to fit this purpose (IMO probably the best and only use for ChatGPT--I won't get into that here though).*

First, we need to make a file:

`cd /var/www/html/
sudo nano search.php`

Paste in this code:

`<?php
// Load MySQL credentials
require_once 'login.php';

// Establish connection
$conn = mysqli_connect($db_hostname, $db_username, $db_password) or
  die("Unable to connect");

// Open database
mysqli_select_db($conn, $db_database) or
  die("Could not open database '$db_database'");

// Check if search query was submitted
if (isset($_POST['search'])) {
    // Sanitize user input to prevent SQL injection attacks
    $search = mysqli_real_escape_string($conn, $_POST['search']);

    // Get the start and end dates for the date range
    $start_date = mysqli_real_escape_string($conn, $_POST['start_date']);
    $end_date = mysqli_real_escape_string($conn, $_POST['end_date']);

    // Build the MySQL query with a WHERE
    // clause that includes the date range filter
    $query = "SELECT * FROM books WHERE
        (author LIKE '%$search%' OR
        title LIKE '%$search%' OR
        publisher LIKE '%$search%') AND
        copyright BETWEEN '$start_date' AND '$end_date'";

    // Execute the query
    $result = mysqli_query($conn, $query);

    // Check if any results were returned
    if (mysqli_num_rows($result) > 0) {
        // Loop through the results and output them
        while ($row = mysqli_fetch_assoc($result)) {
            echo "ID: " . $row["id"] . "<br>";
            echo "Author: " . $row["author"] . "<br>";
            echo "Title: " . $row["title"] . "<br>";
            echo "Publisher: " . $row["publisher"] . "<br>";
            echo "Copyright: " . $row["copyright"] . "<br><br>";
        }
    } else {
        echo "No results found.";
    }

    // Free up memory by closing the MySQL result set
    mysqli_free_result($result);
}

// Close the MySQL connection
mysqli_close($conn);

echo "<p>Return to search page: <a href='http://11.111.222.222/opacbb.html'>http://11.111.222.222/opacbb.html</a></p>";

?>`

Obviously, you need to swap your external IP in place of the `11.111.222.222` stuff in there, but otherwise this should give you a pretty bland, if servicable, search. 

### Making an HTML search form

*This allows the user to access the search script that we just made.*

The PHP script that we just made comes with a placeholder name for the HTML page we're about to make. Obviously you can pick another name, but you'll need to change 
it in the linkback function at the very bottom of that PHP script, if you decide to go that route.

For now, make a new HTML file:

`sudo nano opacbb.html`

Paste in some code to allow for search boxes and communication with the PHP script:

`<html>
<head>
<title>MySQL Server Example</title>
</head>
<body>

<h1>A Basic OPAC</h1>

<p>In the form below,
<b>optionally</b> enter text in the search field.
You can search by author, title, or publisher.
Capitalization is not necessary.
It's okay to enter partial information,
like part of an author's, title's, or publisher's name.</p>

<p>The date fields are <b>required</b>.
You can use the date fields to limit results.
I added some extra records,
which you can view to know what you can query:</p>

<p><a href="http://11.111.222.222/opac.php">http://11.111.222.222/opac.php</a></p>

<p>This is very much a toy, stripped down OPAC.
The records are basic.
Not only do they not conform to MARC,
but they don't even conform to something
as simple as Dublin Core.
I also don't provide options
to select different fields,
like author, title, or publisher fields.
Instead the search field below searches
all the fields in our <b>books</b> table.
The key idea is to get a sense,
an intuition, of how an OPAC works, though.</p>

<h2>My Basic Library OPAC</h2>
<form method="post" action="search.php">
    <label for="search">Search:</label>
    <input type="text" name="search" id="search">
    <br>
    <label for="start_date">Start Date:</label>
    <input type="date" name="start_date" id="start_date">
    <br>
    <label for="end_date">End Date:</label>
    <input type="date" name="end_date" id="end_date">
    <br>
    <input type="submit" value="Search">
</form>


</body>
</html>`

By the way, this HTML also links to the OPAC PHP listing that we made earlier, which just shows the whole catalogue.

### Add to the OPAC

*What's the use of a search function if you can't search for anything?*

You need to get back into MySQL to do this next step. Use the same login function that we used last time:

`mysql -u name -p`

Log in with the password you set earlier.

Access the correct database:

`use opacbd;`

If you want to see the contents that you already have:

`select * from books;`

Now, insert some new contents into the table:

`insert into books
(author, title, publisher, copyright) values
('Emma Donoghue', 'Room', 'Little, Brown \& Company', '2010-08-06'),
('Zadie Smith', 'White Teeth', 'Hamish Hamilton', '2000-01-27');`

If you want to delete a record:

`delete from books where author='Julia Phillips';`

Obviously, you can delete by ID or title or publication date, or whatever.

The `\` in the code block above is a note to MySQL that the following character isn't code, but instead part of the field. Use it before single quotes, periods, commas, semicolons, 
and other things that might be commands.

Now you have a barely-functioning OPAC!

### Notes

I found out the hard way this week that--oh yeah! There's a character limit for the author and title categories! I couldn't enter Mr. Humble and Dr. Butcher like I wanted to because 
of that. Otherwise, I didn't run into too many problems. Just getting used to coding more quickly.

