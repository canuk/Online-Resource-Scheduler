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

//schedulelock($_POST['UID']);

//$userinfo   = useridlookup($_POST);
//$thisuserid = $userinfo['usn'];
//$users      = getusers();
$resourcelist = getresources();

//if ($thisuserid) {
        $resource = $resourcelist[$_POST['resource']];
        ?>

        <html>
        <body>
        <table width="100%">
            <tr align="center"><td bgcolor="<? echo menu_color; ?>">
                <h3><? echo $resource['long']; ?></h3></td></tr>
        
            <? if ($resource['warnings']!="") { ?>
                <tr><td bgcolor="#ffdddd">
                    <ul><font color="#990000"><b>Warning:</b>&nbsp;<? echo stripcslashes($resource['warnings']); ?></ul>
                </td></tr>
            <? } ?>
            
            <? if ($resource['comments']!="") { ?>
                <tr><td>
                    <ul><b>Comments:</b>&nbsp;<? echo stripcslashes($resource['comments']); ?></ul>
                </td></tr>
            <? } ?>

            </table>
        <p><a href=" " onClick="window.close(); return false; "><b>[X] Close Window</b></a>
        </html>
        </body>

        <?

//}

//scheduleunlock();

//-----------------------------------------
//JMS 2/3/04 -disabled need for user to be logged in
//jms 2/13/04
// -added warning functionality
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
?>

