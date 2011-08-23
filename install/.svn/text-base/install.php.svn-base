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

?>
<head>
	<title>
	Scheduler Setup Step 1
	</title>
</head>
<body>
<table border="1" cellspacing="0" cellpadding="0" backgroundcolor="#eeffff">
  <tr>
    <td align="center"> 
      <h2><img src="scheduling.gif" width="376" height="61"
 alt="ORS Online Resource Scheduler"></h2>
 </td></tr>
<tr> <td>

<h2>Installation Step 1: Create folders and Copy files</h2>

<?
// Test for PHP version
echo "Identified PHP as Version " . PHP_VERSION . "...<br>";
if (PHP_VERSION<'4.0.5') {
    ?>
	<h3>Install <font color="#990000">FAILED</font></h3>
    <b>ORS requires at least version 4.0.5 of PHP. Please contact your system administrator or web hosting service to
    identify how to execute these scripts in PHP version 4.0.5 or higher. You may also want to read the FAQs in the installation instructions.<p></b>
    <?
    return;
}

// Test for folder write permissions...
$file = "Folder permissions test";
$fd = fopen("../test.txt", "w+");
if ($fd!=FALSE) {
     fclose ($fd);
     $success = @unlink("../test.txt");
} else {

	$myroot = getcwd();    //try using the php-determined folder name
	if (substr($myroot,-7)=="install") $myroot = substr($myroot,0,-8);  	//drop "/install", if there
    ?>
	<h3>Install <font color="#990000">FAILED</font></h3>
    <b>We were unable to write to the base scheduling folder (<? echo $myroot; ?>).
    It is quite possible that you need to set the permissions on the base folder to "world writable" to install ORS
    (you can reset them back to standard permissions after installation). Please talk to your system administrator
    and review the installation instructions for more information.</b>
    <?
    return;
}

//-----------------------------------
// do installation

require_once('defaults.php');	// get root_dir and other info

// - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
//Copy PHP files which need to have their extension modified appropriately for execution
// (NOTE: internal references to these files need to be made using required_php_extension
//    or we might not find the file when user requests other-than-"php" extension!) 
foreach (array("index.php","utilities.php","recurring.php",
			   "backup_utilities.php","email_announcement.php",
			   "list_members.php","list_signups.php","log_utilities.php",
			   "resource_management.php","user_management.php",
			   "utilities_menu.php","upgrade.php","resourcecomments.php") as $file) {
    echo ".";
	if (required_php_extension != "") {
    	$success = copy($file, "../" . substr($file,0,-4) . required_php_extension);
    } else
    	$success = copy($file, "../" . $file);
	if (!$success) break;
}

// - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
//Copy all other files which either don't need (or SHOULDN'T have) their extension changed

if ($success) {
    foreach (array(
    		"defaults.php","functions.php",
    		"dayview.php","monthview.php","dayview_horiz.php","config.php","textbackup.php",
    		"maintfunctions.php",
    		"blue.gif","green.gif","line.gif","red.gif","ruleline.gif",
    		"scheduling.gif","selected.gif","white.gif","yellow.gif",
    		"recurring.gif","helpicon.gif","warnicon.gif",
    		"pencil.gif","today.gif","ors.css",
    		"license.html")
    	as $file) {
    	echo ".";
    	$success = copy($file, "../" . $file);
    	if (!$success) break;
    }
}

// - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

// Install root directory
if ($success) {
	$file = substr(root_dir,0,-1);
	echo ".";
    if (!file_exists($file)) $success = mkdir ($file, 0700);	// install root_dir
	$success = file_exists($file);
	if ($success) chmod($file, 0700);	//set access to appropriate level
}

// Install backup file directory
if ($success) {
	$file = root_dir . "../backup";
	echo ".";
    if (!file_exists($file)) $success = mkdir ($file, 0777);	// install backup folder
	$success = file_exists($file);
	if ($success) chmod($file, 0700);	//set access to appropriate level
}

//------------------------------------
// give feedback to user

$startlink = "upgrade" . required_php_extension;

if ($success) {
?>
	<h3>Install Step 1 Complete - <a href="../<? echo $startlink; ?>">Click to continue installation...</a></h3>
<? } else { ?>
	<h3>Install <font color="#990000">FAILED</font></h3>

	Problem with file: <? echo $file; ?> <br>

<? } ?>

</td></tr></table>
</body>

<?

//----------------------------------------------------------------------
// JMS 3/02/03
//   -begain tracking changes
//   -added scheduling.gif to items to copy
//   -removed automatic refresh link
// JMS 5/19/03
//   -changed \limit to /limit in .htaccess file
// JMS 8/18/03
//   -added basic support for different extensions (.php4 for eg)
//   -added test for wrong version of PHP
//   -added test for unable to write to base scheduling folder
// JMS 10/4/03
//   -revised required PHP version (4.0.5)
//   -added test for register_globals above version 4.2
// JMS 1/5/04
//   -added recurring
// JMS 1/26/04
//   -added additional icons
// JMS 2/7/04
//   -fixed bug which wouldn't catch some file creation problems
// JMS 2/19/04
//   -removed .htaccess file creation (moved to upgrade.php)
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
//JMS 7/2010
// - NOTE: most change information is now stored in the repository log on sourceforge
//   rather than in the files.
