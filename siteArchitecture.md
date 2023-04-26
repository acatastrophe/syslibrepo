# Week 14-15: Site Architecture

## Steps
1. Make the URL Work??
2. Make Wordpress Pretty
3. Make Omeka Have Things In It
4. Make Omeka Pretty
5. Make Koha Have Things In It

### Make the URL Work?

The following is the full text (partially edited) of my Major Issue 
Troubleshooting assignment, since it covers the biggest thing I accomplished 
this week:

My problem was that the Wordpress/Omeka server was responsive and functional, but I could not access the web pages through a browser. 
The desired solution was to restore web accessibility and also to give the end user an easier way to find the front end of the site by using a custom domain name instead of an external 
IP address.

At first, I hit a roadblock trying to figure out why the virtual machine was responsive but the site was not. 
The error occurred after a short time away from the project, and my free trial of Google Cloud had expired–I had already found that the virtual machine 
instances were both halted and had to be restarted before they were functional. However, I had collapsed the view of their external IP addresses and couldn’t 
see that one had changed. A note from a classmate in our Element chat made me realize I needed to check there, and I found that the new external IP worked, at least 
for the homepage–I didn’t try any further links at the time.

At this point, I decided to work on setting a domain name in order to make the Wordpress site easier to access by the public. 
I browsed Google Domains’ options for domains titled “Salami” (my nickname from my partner, and the name I decided to use for my zine and small-pub collection), 
eventually settling on salami.quest for $15 per year. At first I set up a simple redirect from salami.quest to 34.27.199.105, but then I searched for official 
documentation and non-official advice about migrating an existing project to a domain name, and found that the most helpful was the subsection Setting up DNS under 
Google Cloud Community’s [Setting Up LAMP on Compute Engine guide](https://cloud.google.com/community/tutorials/setting-up-lamp). Then, following the Setting up DNS subsection, I 
opened the Google Domains management menu for salami.quest and wrote a custom `A` record with host name `salami.quest` and data as `34.27.199.105`, with Time To Live set for 1 hour. 
After a brief wait, [salami.quest/index.php](http://salami.quest/index.php), [salami.quest/wordpress](http://salami.quest/wordpress), and [salami.quest/omeka](http://salami.quest/omeka) linked to my index,
 front page, and digital management system respectively.
 
However, when I tried to navigate to Wordpress pages linked to the front page, my browser attempted to reach the old external IP address. 
This happened with hand-linked pages for obvious reasons, but also for pages generated by Wordpress that should have redirected, such as blog posts that appeared on the front page. 
After some Googling, I found a [StackOverflow discussion](https://stackoverflow.com/questions/535534/wordpress-host-ip-changed) that fixed the problem. The suggestion was to carefully 
change the `siteurl` and `home` values in the `wp_options` table of Wordpress’s database in MySQL. I logged in as root and (after accidentally changing some name labels to IP addresses, 
which I fixed by taking a quick look at a [sample table without name labels changed](https://wp-staging.com/docs/the-wordpress-database-structure/)) swapped both instances of the old 
external IP to the new one. The code to see and address those option 
discrepancies is as follows:

```
sudo su
mysql -u root
show databases;
Select * from wp_options where option_name IN('siteurl','home'); 
update wp_options set option_value='http://11.11.111.111/wordpress' where option_id='2';
update wp_options set option_value='http://11.11.111.111/wordpress' where option_id='1';
```

With the `select` command, you can see that the two options should read as the same URL (and I could see that the URL was the old one!)

After that, every page loaded properly. The final issue was that pages beyond the front of Wordpress loaded as the external IP address rather than the salami.quest domain, 
but as I’d ultimately fixed what was actually broken and had done what I set out to do when purchasing a domain name, I chalked it up to some issue with Wordpress’s URL writing 
permissions and quit.
The only other remaining problem was the issue of salami.quest routing to the index.php document rather than the Wordpress frontpage. 
This was also an issue I decided not to fix totally, but I did make a quick band-aid fix that makes the experience of navigating to salami.quest a little smoother. 
I swapped index.php and index.html in the dir.conf file, thereby restoring Apache2’s preference for index.html. Then, I wrote an [HTML and CSS webpage](http://salami.quest) with a small 
disclaimer about the nature of my library’s materials (which are largely fan items concerning copyrighted media properties, and are in a legal gray area as publications) 
and added a hyperlink to the Wordpress site. The CSS used in the webpage was existing code from my personal website, which I had lying around in my Codepen account. 
Ideally I would have set up a redirect from salami.quest to salami.quest/wordpress, but as time was short, I decided to capitalize on the opportunity to add a materials disclaimer, 
instead.

### Make Wordpress Pretty

This part was actually more frustrating than the last! I'm used to working on websites the very, very old-fashioned way (with HTML and CSS and maybe a touch of Javascript), so site 
builders inevitably drive me up the wall. When so many options are assigned to one functionality, like a click, you end up fumbling more than you actually do anything... anyway.

When I first installed Wordpress, I opted not to mess with themes too much--ultimately none of them really looked like what I wanted them to, so I went with the pre-loaded Twenty 
Twenty-Two and modified it to fit my needs. I swapped in a free image with a transparent background as the header, made a post to sit on the front page, touched up the About page
 using a free template, and added a set of buttons to the header. One navigates to the About page, one goes to Omeka, and the last two go to Koha, to the staff and the public portals. 
Finally, I added a link to my GitHub repository in the footer of the page as "View Devlog" (though the idea of this account being an actual Devlog is kind of laughable), and called it.

### Make Omeka Have Things In It

I had already decided to use the library websites as a repository for my own collection of zines and indie publications; I have a number of them in my physical collection at home, and 
even more in my various digital hoarding spaces. The Omeka library was more important to me because I have more digital items than physical; additionally, Omeka defaults to Dublin 
Core metadata, which fits the sorts of items I cataloged better than MARC metadata does. I added several of my TTRPGs and zines to the catalogue before messing around with it further.

Additionally, I cleaned up Omeka a little bit. Originally, I had entered some Gundam manga series when I installed the software, and I didn't really want to roll back all the word I did 
for that week, so I set them all to private. The collection they were in also got set to private.

After I'd entered about a dozen items, I set one item (the Bowser/Luigi zine, naturally) as featured, and made a collection with some of the tabletop roleplaying games.

### Make Omeka Pretty

This step was pretty short, because I didn't mess around with Omeka themes beyond the preloaded. Initially I didn't like the look of the Seasons theme and had been using the Berlin theme,
 but found that Seasons looked a little less empty. The Spring settings meshed well with Koha's default color, as well as some of the dominant colors I picked for Wordpress, so 
I set that theme and left it. I also turned on advanced site-wide search to allow for Boolean searching and ticked `Display Featured Item` and `Display Featured Collection` so that 
everyone would see what kind of weirdo I am. Ultimately I decided that I didn't like the look of homepage text, so I left it off (my collection has a different front page, anyway), and 
I set navigation to full-width. 

At some point I'd like to see if I can make a logo occupy the same space as the title. This theme swaps them--I can't set a logo and a title at the same time--but I'd love to have a little 
PNG of a salami next to the Salami ILS title!

### Make Koha Have Things In It

This part is barely worth talking about because it's kind of embarassing... as I said previously, MARC records only kind of work for the items that I'm cataloging. I gave it my best 
effort, and even found a blog post that offered MARC records of some popular board games to work from, but ultimately found that it was just kind of a shitshow no matter how I 
sliced it. I managed two dozen items and called it quits without changing the theming for the site. I already picked green. Koha is green by default. Moving on.

(Side note: I noticed Koha had options for restricted items, though I didn't mess around with it. Perhaps I will in the future!)

### Have a Site!

Now I have a functioning site! As I said before, I made a little front page to tie the gap between the top level domain and the Wordpress instance, so you can visit [Salami ILS here](http://salami.quest)!

### Notes

Not many for the final week. I enjoyed tying things together, and I'm really proud that I got the custom domain name to work! I plan to maintain this site on my own, add some more of my 
items, and build it out. It'll be a nice little portfolio piece... or just a good repo for my uncommon items.