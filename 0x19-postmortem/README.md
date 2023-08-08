Upon the release of ALX System Engineering DevOps project 0x19, approximately 05:00, Coordinated Universal Time (UTC) on the 8th of May, 2023, an outage occurred on an isolated Ubuntu 14.04 container running an Apache web server. GET requests on the server led to 500 Internal Server Error's, when the expected response was an HTML file defining a simple Holberton WordPress site.

Debugging Process Bug debugger Brennan (BDB... as in my actual initials... made that up on the spot, pretty good, huh?) encountered the issue upon opening the project and being, well, instructed to address it, roughly 23:00 UTC. He promptly proceeded to undergo solving the problem.

Checked running processes using ps aux. Two apache2 processes - root and www-data - were properly running.

Looked in the sites-available folder of the /etc/apache2/ directory. Determined that the web server was serving content located in /var/www/html/.

In one terminal, ran strace on the PID of the root Apache process. In another, curled the server. Expected great things... only to be disappointed. strace gave no useful information.

Repeated step 3, except on the PID of the www-data process. Kept expectations lower this time... but was rewarded! strace revelead an -1 ENOENT (No such file or directory) error occurring upon an attempt to access the file /var/www/html/wp-includes/class-wp-locale.phpp.

Looked through files in the /var/www/html/ directory one-by-one, using Vim pattern matching to try and locate the erroneous .phpp file extension. Located it in the wp-settings.php file. (Line 137, require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).

Removed the trailing p from the line.

Tested another curl on the server. 200 A-ok!

Wrote a Puppet manifest to automate fixing of the error.

Summation In short, a typo. Gotta love'em. In full, the WordPress app was encountering a critical error in wp-settings.php when tyring to load the file class-wp-locale.phpp. The correct file name, located in the wp-content directory of the application folder, was class-wp-locale.php.

Patch involved a simple fix on the typo, removing the trailing p.

Prevention: 
This outage was not a web server error, but an application error. To prevent such outages moving forward, please keep the following in mind.

Test! Test test test. Test the application before deploying. This error would have arisen and could have been addressed earlier had the app been tested.

Status monitoring. Enable some uptime-monitoring service such as UptimeRobot to alert instantly upon outage of the website.

Note that in response to this error, I wrote a Puppet manifest 0-strace_is_your_friend.pp to automate fixing of any such identitical errors should they occur in the future. The manifest replaces any phpp extensions in the file /var/www/html/wp-settings.php with php.

But of course, it will never occur again, because we're programmers, and we never make errors! 

This postmortem was from a certain individual, and story so much resonates with what I faced myself. In the course of my journey, I have grown so open to the ups and downs of Software Engineering! m
