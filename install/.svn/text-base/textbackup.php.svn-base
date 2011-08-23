<?
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

require_once('functions.php');

//copy all items from get and sanitize all values
foreach ($_GET as $key=>$value) $_POST[$key]=$value;
foreach ($_POST as $key=>$value) $_POST[$key]=sanitize($value);

schedulelock($_POST['UID']);

$userinfo   = useridlookup($_POST);
$thisuserid = $userinfo['usn'];
$_POST['thisuserid'] = $thisuserid;        //copy to use in functions

$users      = getusers();

if (checksec($users[$thisuserid]['sec'],100)) {
        require_once('maintfunctions.php');
        
        $backup_file = createbackup();

		header("Cache-control: private"); // fix for IE
		header("Content-Length: ". filesize($backup_file));
		header("Content-type: application/x-gz-compressed");

 		if(preg_match("/MSIE 5.5/", $_SERVER['http_user_agent']))
			header("Content-Disposition: filename=" . basename($backup_file));
		else
			header("Content-Disposition: attachment; filename=" . basename($backup_file));

        readfile($backup_file);
}

scheduleunlock();

//JMS 8/17/03
//  -reenabled test for administrator SEC
//JMS 1/26/03
//  -updated for new compression method
//  -file now sent as attached
//JMS 4/23/04
// -build backup file only ONCE, save to disk and send THAT file
// -use correct filename (matching one saved to site)
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
?>
