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

require('utilities_menu.php');
if (!$thisuserid) return;


if (!$administrator) {
    echo "<h3>You are not authorized to perform this action</h3>";
    scheduleunlock();
    printfooter();
    return;
}

switch($_POST['submit']){
		case "emaillog" : //email the log
                require_once('maintfunctions.php');
                echo "<b>Action log has been e-mailed to the following e-mail addresses:</b><br>";
                if (demomode) {echo "Demo mode - e-mail disabled";}
                echo "&nbsp;&nbsp;&nbsp;" . mailactionlog(false) . "<br>";

        case "reviewactionlog" : //display action log

                ?>
                <b>Action Log:</b>
                <?
                echo '&nbsp;&nbsp;<a href = "' . log_util_prog . '?UID=' . $UID . '&submit=emaillog">[E-Mail Log to Administrators]</a><p>';

                $items = readactionlog();

                if (count($items) == 0) {
                        echo "<b>Log is empty</b>";
                        break;
                }

                echo "</td></tr></table>";

                $items = array_reverse($items);
                echo "<table border=0>";
                echo "<tr><td><b>Date</b></td><td><b>Action</b></td><td><b>Details</b></td></tr>\n";
                foreach ($items as $linenum => $entry) {
                        $data = $entry['data'];
                        echo '<tr><td nowrap valign="top">' . $entry['date'] . '&nbsp;&nbsp;&nbsp;</td><td valign="top">';
                        $give_details = False;
                        if ($data['action'] == "Bad Password") {
                                echo "<b>" . $data['action'] . " for " . $data['lastname'] . "</b></td>";
                                $give_details = True;
                        } elseif (in_array(strtolower($data['action']),
                            array("deleted authorized user","added authorized user","updated authorized user","restore from backup","updated configuration","assigned usn","deleted user"))) {
                                echo $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " <b>" . $data['firstname']{0} . ". " . $data['lastname'] . "</b>";
                                echo "&nbsp;&nbsp;&nbsp;</td><td>";
                                if (in_array("sec", array_keys ($data))) echo " SEC = " . $data['sec'];
                                if (in_array("usn", array_keys ($data))) echo " USN = " . $data['usn'];
                                $give_details = True;
                        } else {
                                if ($data['thisuserid'] != $data['usn'] & $data['usn'] != "") {
                                        $targetuser = $users[$data['usn']]['firstname']{0} . ". " . $users[$data['usn']]['lastname'];
                                        if ($users[$data['usn']]['lastname']=="") $targetuser = "(usn " . $data['usn'] . ")";
                                        echo $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " <b>" . $targetuser . "</b>";
                                        $give_details = True;
                                } elseif ($data['thisuserid'] != "") {
                                        echo $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " self";
                                        $give_details = True;
                                } else
                                        echo $data['action'];
                                echo "&nbsp;&nbsp;&nbsp;</td><td>" . $resourcelist[$data['resource']]['short'];
                                if (in_array("sec", array_keys ($data))) echo "SEC = " . $data['sec'];
                                if (in_array("starttime", array_keys ($data)) & in_array("endtime", array_keys ($data)))
                                        echo ", " . date(dateformat . " " . timeformat,$data['starttime']) . "-" . date(dateformat . " " . timeformat,$data['endtime']);
                                if (in_array("priority", array_keys ($data))) echo ", priority " . $data['priority'];
                                if (in_array("signup", array_keys ($data))) echo " (signup " . $data['signup'] . ")";
                        }
                        if ($give_details) {
                                echo '<td valign="top"><font size="-1">';
                                echo '<a href = "' . log_util_prog . '?UID=' . $UID . '&submit=actiondetail&linenum=' . $linenum . '">';
                                echo '&lt;Details&gt;</a></font>';
                        }
                        echo "</td></tr>\n";
                }
                echo "</table>\n";
                break;

        case "actiondetail" : //display action log entry details

                echo '<b>Action Log Entry Details:</b>';
                echo '<a href = "' . log_util_prog . '?UID=' . $UID . '&submit=reviewactionlog">[View entire action log]</a>';
                echo '<p>';

                $items = readactionlog();

                if (count($items) == 0) {
                        echo "<b>Log is empty</b>";
                        break;
                }

                $items = array_reverse($items);

                $entry = $items[$_POST['linenum']];
                $data  = $entry['data'];

                echo "Date: " . $entry['date'];
                echo "<table border=0>";
                echo "<tr><td>&nbsp;&nbsp;&nbsp;</td><td><b>Key</b></td><td><b>Value</b></td></tr>\n";
                foreach ($data as $key => $item) {
                        if (in_array($key,array("endtime","starttime","tabledate")))
                                $item .= " (" . date(timeformat . " " . dateformat . "/y",$item) . ")";
                        echo "<tr><td></td><td>" . $key . "&nbsp;&nbsp;&nbsp;</td><td>" . $item . "</td></tr>\n";
                }
                echo "</table>\n<p>";
                break;

        //- - - - - - -

        case "reviewuselog" : //display use log (uid)

                ?>
                <b>Use Log:</b><br>
                <?
                 $items = getuids();
                $items = array_reverse($items);
                echo "<br>";
                echo "<table border=0>";
                echo "<tr bgcolor=\"#CCCCCC\"><td colspan=\"3\"><b>Active Users</b></td></tr>";
                echo "<tr bgcolor=\"#EEEEFF\"><td><b>Login Date</b></td><td><b>User</b></td><td><b>Length</b>(minutes)</td></tr>\n";
                foreach ($items as $data) {
                        echo "<tr bgcolor=\"#EEEEFF\"><td nowrap valign=\"top\">" . date(dateformat . "/y " . timeformat,$data['issuedate']) . "&nbsp;&nbsp;&nbsp;</td><td valign=\"top\">";
                        echo $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'];
                        if ($data['lastused'] < $data['issuedate'] & $data['lastused'] > 0) echo '&nbsp;&nbsp;&nbsp;</td><td valign="top"><center>' . round($data['lastused']/60) . "</center>";
                        elseif ($data['lastused'] >= $data['issuedate']) echo '&nbsp;&nbsp;&nbsp;</td><td valign="top"><center>~' . round(($data['lastused']-$data['issuedate'])/60) . "</center>";
                        else echo "&nbsp;&nbsp;&nbsp;</td><td valign=\"top\"><center>";
                        echo "</td></tr>\n";
                }
                echo "</table>\n";
                if (count($items) == 0) {
                        echo "<b>Active User Log is Empty</b>";
                }

                $items = getuselog();
                $items = array_reverse($items);
                echo "<br>";
                echo "<table border=0>";
                echo "<tr bgcolor=\"#CCCCCC\"><td colspan=\"3\"><b>Logon History</b></td></tr>";
                echo "<tr bgcolor=\"#EEEEFF\"><td><b>Login Date</b></td><td><b>User</b></td><td><b>&nbsp;IP&nbsp;Address&nbsp;</b></td></tr>\n";
                foreach ($items as $data) {
                        echo "<tr bgcolor=\"#EEEEFF\"><td nowrap valign=\"top\">" . date(dateformat . "/y " . timeformat,$data['logondate']) . "&nbsp;&nbsp;&nbsp;</td><td valign=\"top\">";
                        echo $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'];
                        echo "</td><td>&nbsp;" . $data['ip'] . "&nbsp;</td>";
                        echo "</td></tr>\n";
                }
                echo "</table>\n";
                if (count($items) == 0) {
                        echo "<b>Use Log is Empty</b>";
                }
                break;
}

scheduleunlock();
printfooter();

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
//JMS 1/5/04
// -added euro-date support
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
