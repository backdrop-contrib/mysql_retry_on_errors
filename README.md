MySQL Retry on Errors
=====================

Under certain conditions, Backdrop can throw fatal database errors/exceptions on sites
with lots of concurrent editing or during complex batch operations. The easiest solution is simply to wait a brief period
and then retry the query.

This project contains a modified MySQL/MariaDB database driver for Backdrop, which will tell Backdrop to automatically
retry a query (after a random brief delay) if it encounters a database error such as a deadlock condition or 
wait timeout. If after a set number of attempts the error persists, Backdrop will fall back to its default 
behavior and throw the fatal error/exception.

If you have never experienced this type of problem, you do not need this project.


## Installation


This project takes a little more effort than a normal module to get working.

First, install and enable the module as normal.

Next, you must either copy the module's "database/mysql_retry_on_errors" directory into core, or, create
a symlink to it like so (in linux):

`ln -s /backdrop_site/modules/mysql_retry_on_errors/database/mysql_retry_on_errors /backdrop_site/core/includes/database/mysql_retry_on_errors`

Either way, you should end up with the following files at the following locations:

* core/includes/database/mysql_retry_on_errors/database.inc
* core/includes/database/mysql_retry_on_errors/install.inc
* core/includes/database/mysql_retry_on_errors/query.inc
* core/includes/database/mysql_retry_on_errors/schema.inc

If you are in a Windows environment, you can create something very similar to a symlink using the "MKLINK /J" command.


## Configuration and Settings


Next, you must edit your settings.php file and tell it to use the new database driver, as well as configure
a few settings.

Add the following just after your $database array is set up:

```
/**
 * The following settings are for setting up the mysql_retry_on_errors database driver.
 * Note the following database error codes:
 * - 1213 = deadlock
 * - 1205 = wait timeout
 * - 1412 = table structure being modified while trying to access
 */

$database['driver'] = 'mysql_retry_on_errors';  // comment-out to use default mysql driver
$settings['mysql_retry_on_errors'] = array(
  'max_retries' => 3,
  'retry_on_error_codes' => array(1213, 1205, 1412), 
  'min_delay' => 100,  // in milliseconds
  'max_delay' => 1000,  // in milliseconds (anything over 2000 will be capped at 2000)
);
```

If saving this immediately breaks your site, comment out the `$database['driver']` line, and 
visit admin/reports/status to check for issues. Mostly commonly the problem will be that
the new driver files could not be found.


## Usage and Seeing if it Works

If all goes well, there is nothing further you need to do. Any time a query runs into one of the specified
error codes, it will retry after a short random delay, and log the event.

To make sure retry events are logged, visit admin/config/development/logging and make sure "Debug" is also selected.

In your watchdog (admin/reports/dblog) retry attempts will be logged as type "mysql_retry_on_errors".


## Updating Backdrop Core

Unfortunately, because this is a modification to Backdrop's core directory structure, you might have to 
re-copy or re-create the symlink after updating core.


## Future Plans to Make it Easier

A PR will be submitted to add this retry logic into Backdrop's core code.  If that is accepted, this project will
become obsolete, and there will be no need to worry about what to do after future Backdrop core updates.

See this issue for updates: https://github.com/backdrop/backdrop-issues/issues/7092

## Current Maintainers

* [Richard Peacock (swampopus)](https://github.com/swampopus) - Originally created for Backdrop CMS.
* Seeking additional maintainers.







