Usage
===============

For the most part, torrentwatch-xa is very intuitive and self-explanatory. These usage notes explain some of the advanced features, details, and behaviors that are not immediately obvious.

### Item Colors

The small colored bars on the left edge of some items indicate the states of the items. Click the Legend button to see what each color means.

### Check for Updates

Check for Updates checks once every 7 days for new versions. If you turn it off, then turn it back on, it will take up to 7 days to check again.

### Seed Ratio Settings

If set to a positive number, each Favorite's seed ratio setting overrides its parent Feed's seed ratio setting, which overrides the global Default Seed Ratio setting. To allow inheritance to occur, leave the setting blank. Any negative number gets overridden by -1. If the global Default Seed Ratio is blank, it is overridden by -1.

Transmission itself has a seed ratio limit that will override any limit set within torrentwatch-xa.

### Configure > Feeds

torrentwatch-xa provides you with several default feeds when starting fresh with no config file. If you've added your own feeds, you should probably disable or remove any of these default feeds that you don't use to improve twxa_cli.php's performance and reduce the load placed on the feed host(s), saving their operators bandwidth. Please be sure to visit your favorite feeds' websites often so that they can earn advertising revenue from your support and help keep the anime fansubbing community alive--thanks!

### Configure > Trigger

torrentwatch-xa can trigger email notifications by SMTP or shell scripts or both. Shell scripts can be used for post-processing including sending email notifications in place of or in addition to the built-in SMTP notifications.

SMTP Notifications trigger on these events:
- Favorite item starts downloading (usually started by cron job, but web UI can do it as well)
- error while downloading
- error in Script

Scripts trigger on these events:
- Favorite item starts downloading
- non-Favorite item starts downloading
- error while downloading

To run a shell script, check Enable Script, provide the full path to a single shell script with no parameters in the Script field. Your shell script must have rwx permissions for www-data, and no parameters may be supplied in the Script field. See /var/lib/torrentwatch-xa/examples for example shell scripts that you can customize to suit your needs.

To use the built-in SMTP notifications, check SMTP Notifications and fill in the From: Email and To: Email fields and all the SMTP fields. SMTP Port defaults to 25 if left blank. From: Name and HELO Override are optional and will use default settings if left blank. If the cron job has trouble sending email notifications, it is probably unable to retrieve the hostname for use as the HELO, and the HELO Override will be necessary.

The Test button tests the SMTP settings currently in the form by sending a test email. You must click the Save button to actually save the settings.

torrentwatch-xa uses PHPMailer 5.2.23 to send emails, so you may need to refer to PHPMailer documentation for help in understanding any SMTP error messages that appear.

### Season and Episode Notation

A quick explanation of the new season and episode notation in the "episode label" shown on each line to the left of the timestamp at the right edge of the feed list:

- SxE = single episode
- SxEv# = single episode with version number
- YYYYMMDD = single date
- S1xE1-S1-E2 = batch of episodes within one season
- YYYYMMD1-YYYYMMD2 = batch of dates
- S1xFULL = one full season
- S1xE1-S2xE2 = batch of episodes starting in one season and ending in a later season
- S1xE1v2-S2xE2v3 = batch of episodes starting in one season and ending in a later season, with version numbers

For items not recognized as having an episodic numbering, Glummy ("_ ) is displayed.

Internally, the new Favorite matching engine uses direct comparisons of the separate season and episode as discrete numeric values and does not deal with this notation at all.

Later, when the Favorite Episodes filter functionality is implemented, it will also use this notation (except for Glummy, who is for display only).

The ideal notation for videos is actually SxVxEv# (Season x Volume x Episode version #); if downloading anime BluRay Disc sets becomes super-popular, I may implement this notation style throughout torrentwatch-xa in a future version.

### Current Episodes Filter Notation

The Episodes filter currently in each Favorite is still the old TorrentWatch-X filter. The notation style is the old style, like so:

- SxE = single episode
- SxEp = single episode, PROPER or Repack
- S1xE1-S2xE2 = batch of episodes starting in one season and ending in a later season

### RegEx Matching Style vs. Simple vs. Glob

The Favorites fields behave differently in RegEx Matching Style than in Simple or Glob in that PCRE Unicode regular expressions are used in the Filter, Not, and Qualities fields in RegEx mode.

### Authentication for Private RSS Feeds

See the section "Only Public Torrent RSS or Atom Feeds Are Supported" in the **Design Decisions Explained** section below for more details.

### Auto-Del Seeded Torrents

When enabled, either the web UI or twxa_cli.php (run by the cron job) will automatically delete completely-downloaded, fully-seeded torrents from Transmission without trashing the torrent's contents. Auto-Del Seeded Torrents is also smart enough not to delete any torrents that are not found in the download cache, preventing it from deleting torrents that were added to Transmission via other means.

The web UI is fully synchronized with Transmission so that items auto-deleted by twxa_cli.php will be removed from the web UI correctly without requiring a browser refresh.

### Magnet Links Saved as Files

If Also Save Torrent Files is enabled and torrentwatch-xa retrieves a magnet link instead of a .torrent file, it will save the magnet link in a file. There are third-party tools that easily convert magnet links stored in text files to torrent files, if you prefer the torrent file over the magnet link.

The ability to save magnet links was added to deal with the increasingly-common feeds that have only magnet links and no links to torrent files.

### Bulk Favorites Importer twxa_fav_import.php

_WARNING!! The bulk importer is experimental; use it at your own risk! Be sure to back up your config file before any bulk import!_

1.1.0 includes twxa_fav_import.php, a command-line tool that can import a tab-separated-values (TSV) file containing a list of favorites.

A good way to use the bulk Favorites importer is to go to anichart.net, look up next season's anime titles, and then create the TSV and import it. This will catch the first episode of each show right when it starts. Be aware that importing a massive number of Favorites will slow down torrentwatch-xa if you are running a Raspberry Pi or other low-powered CPU.

Create a plain text TSV file with these columns in order from left to right:

1. Name (required)
2. Filter (required)
3. Quality (optional)

Filter and Quality can be regular expressions. (Set Configure > Favorites > Matching Style to RegExp in order to use regular expressions.)

Close the browser if you have torrentwatch-xa's web UI open.

Then, at the command line, run:

`sudo cp /var/lib/torrentwatch-xa/config_cache/torrentwatch-xa.config <path to put the backup>`

`sudo /usr/bin/php /var/www/html/torrentwatch-xa/twxa_fav_import.php <path to TSV>`

You can watch the web server's log file (default: /var/log/apache2/error.log) for PHP errors and torrentwatch-xa's log (default: /var/log/twxalog) for import errors.
If there are no errors, go ahead and open torrentwatch-xa in the browser and make sure the new Favorites are imported.

If the TSV file confuses PHP's fgetcsv() function, there is a good possibility you will corrupt your config file. If you have to put the backup file back, do this:

Close the browser if you have torrentwatch-xa's web UI open.

`sudo cp <path to put the backup> /var/lib/torrentwatch-xa/config_cache/torrentwatch-xa.config`

Design Decisions Explained
===============

There are situations for which a mutually-exclusive design decision cannot be avoided. The below are design decisions that will never be "fixed."

### Only Public Torrent RSS or Atom Feeds Are Supported

I have found that due to the highly fluid nature of the torrent scene, it's better to stick with public torrent RSS or Atom feeds than deal with the many different authentication systems of private torrent feeds. Just about everything you could want is going to be available via multiple public torrent feeds anyway.
But, if you absolutely must use a private RSS feed with authentication, there is an easy way to hook torrentwatch-xa up to it. There are many third-party RSS feed tools that can connect to RSS feeds that have authentication and then re-publish the feeds without authentication. I have not tried these apps listed here myself, but most of them should be able to do this: [http://www.makeuseof.com/tag/12-best-yahoo-pipes-alternatives-look/](http://www.makeuseof.com/tag/12-best-yahoo-pipes-alternatives-look/)

### Some Numbering Schemes Only Make Sense to Humans

The title "Holly Stage for 50 - 3" is meant to be interpreted as title = "Holly Stage for 50" and Episode 3, with Season 1 implied.
(Fans know that "Holly Stage for 50 - 3" really should be read as title = "Holly Stage for 49", Season 2, Episode 3, to further complicate matters.)
But the engine currently reads it as title = "Holly Stage for" and Season 50, Episode 3. Why? Because it was determined that the ## - ## pattern much more often means Season ## - Episode ##.

Sadly, because the engine was forced to make the choice, fans of "Holly Stage for 50" must "hack" the favorite to get it to download properly. There is no way to solve this problem without referring to some centralized database of anime titles or relying on some sort of AI, neither of which are going to happen in torrentwatch-xa any time soon.

### 720 and 1080 Without i or p

Titles such as Gamers! - 05 720.mkv are now recognized as Season 1, Episode 5 with resolution of 720p. For a long while, I allowed torrentwatch-xa to continue mismatching it as Season 5, Episode 720 on the off chance that the series is popular enough to have episodes that go that high, but such long-running series are just too outnumbered by the crews who release titles without i or p after 720 and 1080.

### Item Says It's an Old Favorite but is Actually New and Should Be Downloaded

This can happen if there are parallel numbering styles for the same torrent. For instance, with HorribleSubs Boku no Hero Academia 17 (Season 1, Episode 17), some crew on the Feedburner Anime (Aggregated) feed was re-releasing it later as Season 2, Episode 4. What happened then was that once torrentwatch-xa saw the Season 2 track, it jumped onto it and began ignoring the Season 1 numbering. The Season 1-numbered episodes would come out a few hours earlier than the re-release each week and not be auto-downloaded, making it seem like a detection failure.

This is not a bug. Technically, the season and episode detection engine is working properly; it's the crew that was renumbering episodes that was causing problems. The episode would auto-download once the Season 2 renumbering was released.

One easy workaround is to use the Favorite Episodes filter to restrict the downloads to just the Season 1 numbering: 1x1-1x99 would "trap" the series into Season 1 numbering.

### Items Drop Off the Feed Lists

If one starts an item downloading from a feed list, and that item is bumped off the end of the feed list by newer items on the next browser refresh, the item will not appear in the Downloaded or Downloading filtered lists even if the item still shows on the Transmission tab as downloading or downloaded. This is because the item simply is no longer in the list to be filtered and then shown by the Downloading and Downloaded filters. It seems counterintuitive until one understands that the Downloaded and Downloading filters are view filters on the feed list, not historical logs nor connected to Transmission's internal list.

### Watch Dir

transmission-daemon provides a watch directory feature. To enable it, use `watch-dir` and `watch-dir-enabled` in `settings.json`.