<?php
//--------------------------------------------------------------
// ORS - Online Resource Scheduler
// Copyright (c) Jeremy Shaver 2002-2010
// Developers include: Jeremy Shaver, Chris Watkins and Jeffrey Tacy
//
// See the license.html file for details on distribution and use
//
// This program is free software; you can redistribute it and/or
//  modify it under the terms of the GNU General Public License
//  as published by the Free Software Foundation; either version
//  2 of the License, or (at your option) any later version.
//
// This program is distributed in the hope that it will be useful,
//  but WITHOUT ANY WARRANTY; without even the implied warranty of
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
//  GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public
//  License (license.html) along with this program; if not, write
//  to the Free Software Foundation, Inc., 59 Temple Place, Suite
//  330, Boston, MA 02111-1307 USA
//  or visit: http://www.opensource.org/licenses/gpl-license.php
//
//--------------------------------------------------------------

//------------------------------------------
// Defaults used by various routines in the scheduler.
// Change these options with care.
//
// Note that in most cases, references to time
// are going to be given in seconds.
//------------------------------------------

error_reporting(E_ALL & ~(E_NOTICE));

//--File, Server and directory info

$myroot = Null;

//-----------------------------------------------------------------
//*** Installation problems? ***
//NOTE: If the installer has problems installing the data directory, you may want to uncomment the
// next line and hard-code in a folder name (including the "scheduling" folder):

//$myroot = 'hardcoded/folder/name/goes/here/scheduling';     //insert your folder name here if the above doesn't work

//-----------------------------------------------------------------



if ($_ENV["SERVER_ADDR"] != "127.0.0.1") {
        define ("submit_method","post");                 // method for on-line based method ['get' or 'post']
} else {        //local proxy server
        define ("submit_method","post");                  // get or post (get is sometimes faster with proxy server)

	//-----------------------------------------------------------------
	//OPTIONAL: Off-line testing folder address [used for OFF-LINE testing only, not necessary normal operation]
	// The following folder address will be used ONLY when you are testing on a local proxy server
	// and is needed ONLY if the automatic-folder-derminiation code below does not work.
	// Enter the local path to the scheduling folder here:

	//  $myroot = 'c:/local/hardcoded/folder/name/goes/here/scheduling';     //insert your local folder name here (local proxy server only)

	//-----------------------------------------------------------------
}

if ($myroot==Null) {
	//Try to automatically determine folder address
	$myroot = getcwd();        						//try using the php-determined folder name
	if (substr($myroot,-7)=="install") $myroot = substr($myroot,0,-8);  	//drop "/install", if there
}

define ("root_dir", $myroot . "/data/");          // directory of data folder

//--Base PHP file names: the following should be changed IF (and only IF!) the names of the base php files are changed
define("required_php_extension", ".php");                                       //required extension for php scripts
define ("schedule_prog",           "index" . required_php_extension);               //php main progam
define ("util_prog",               "utilities" . required_php_extension);           //php utility progam
define ("util_menu_prog",          "utilities_menu" . required_php_extension);
define ("list_members_prog",       "list_members" . required_php_extension);
define ("list_signups_prog",       "list_signups" . required_php_extension);
define ("user_management_prog",    "user_management" . required_php_extension);
define ("email_announcement_prog", "email_announcement" . required_php_extension);
define ("resource_management_prog","resource_management" . required_php_extension);
define ("backup_util_prog",        "backup_utilities" . required_php_extension);
define ("log_util_prog",           "log_utilities" . required_php_extension);
define ("recurring_prog",          "recurring" . required_php_extension);

define ("config_file",         root_dir."../config.php");                       //raw config
define ("config_encoded",      root_dir."config_enc.php");                      //encoded config

//--User-modifiable file locations
define ("memberlist_name",      root_dir . "memberlist.csv");                    //location and name of member list file
define ("actionlog_name",       root_dir . "actionlog.csv");                     //filename for action log
define ("authorizedlist_name",  root_dir . "authorizedlist.csv");                //location and name of authorized user list (other than members)
define ("resourcesfile_name",   root_dir . "resources.csv");                     //location and name of file containing list of resources
define ("lockfile_name",        root_dir . "scheduling.lock");                   //name of the "lockfile" which keeps multiple users' requests from conflicting
define ("announcementfile_name", root_dir . "announcements.html");               //location and name of "announcement" file to prepend to page when user first logs in
define ("page_background",      "");                                             //image for background of scheduler pages

