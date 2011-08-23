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

switch($_POST['submit']){

		case "List Signups" :
        case "datehelp" :
        case "listresource" : //list all signups for a resource

            echo '<form name="SignupForm" method="' . submit_method . '" action="' . list_signups_prog . '">';

            ?>
            <hr><b>List resource signups:</b><br>
            <table border=0 cellpadding=2><tr><td>
            Resource: <select name="resource">
            <?
            foreach (sortby($resourcelist,'order') as $oneitem) {
                    if ($calendarauth | checkpermissions($users[$thisuserid]['resourceblock'],"0",$oneitem['resource'])) {
                            echo "<option value=\"" . $oneitem['resource'] . "\"";
                            if ($resource == $oneitem['resource']) echo " selected";
                            echo ">${oneitem['long']}</option>\n";
                    }
            }
            echo "<option value=\"All\"";
            if ($resource == "All" | $resource == "") echo " selected";
            echo ">All</option>\n";
            ?> </select> &nbsp;&nbsp;&nbsp;
            <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >
            <input type="submit" name="submit" value="List Signups">
            <? if ($calendarauth || $administrator) { ?>
                <input type="checkbox" name="billformat" value="1" <?php if ($_POST['billformat']!="") echo " checked"; ?> >Bill Report Format
            <? } ?>
            </td></tr><tr><td>
            Selected dates: <input type="text" id="selectbegin" name="selectbegin" value="<? echo $_POST['selectbegin']; ?>">
            to <input type="text" id="selectend" name="selectend" value="<? echo $_POST['selectend']; ?>">
            <a href="<? echo list_signups_prog . "?UID=$UID&submit=datehelp"; ?>"><font size="-1">[Help with Dates]</font></a>

            <script type="text/javascript">
                Calendar.setup({
                    inputField     :    "selectbegin",          // id of the input field
                    ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                    weekNumbers    :    false,            //do not show week numbers
                    showOthers     :    true,             //show other months
                });
            </script>
            <script type="text/javascript">
                Calendar.setup({
                    inputField     :    "selectend",          // id of the input field
                    ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                    weekNumbers    :    false,            //do not show week numbers
                    showOthers     :    true,             //show other months
                });
            </script>

            <br>
            (If no dates are entered, all future sign-ups will be shown.)<br>
            </td></tr></table>

            <?
            if ($_POST['submit'] == "datehelp") { ?>
                    <p>You may specify a date using forward slashes "/", dashes "-", or using plain english.
                    <p><b>Examples shown below:</b>
                    <br>1/13/02<br>01/13/02<br>1-13-02<br>1/13/2002<br>13 January 2002<br>
                    <b>English terms can be used as well:</b>
                         <br>today, yesterday, last week, next month, last year
                         <br>next friday, last tuesday
            <? } ?>

            </form> <?

            if ($_POST['submit'] == "datehelp") break;

            $billformat = $_POST['billformat']=="1";

            if ($_POST['selectbegin']!="" | $_POST['selectend']!="") {
                    //validate date entries
                    $validatebegin = validatedate($_POST['selectbegin']);
                    $validateend = validatedate($_POST['selectend']);
                    if (($validatebegin==-1) ) {
                            echo "<b><font size='4'>Your starting date is invalid</font></b>";
                            break;
                    }
                    if ((validateend==-1)) {
                            echo "<b><font size='4'>Your ending date is invalid</font></b>";
                            break;
                    }
                    if ($validatebegin > $validateend) {
                            $temp = $validateend;
                            $validateend = $validatebegin;
                            $validatebegin = $validateend;
                    }
                    echo "<font size='4'>Signups shown for selected dates: ";
                    echo date(m."/".d."/".y, $validatebegin);
                    echo " to ";
                    echo date(m."/".d."/".y, $validateend);
                    echo "</font><br><br>";
                    $datelimited = True;
            } else {
                    $datelimited = False;
            }

            $resourceselection = array();
            if ($_POST['resource'] == "All") {
                foreach(sortby($resourcelist,'order') as $resource) {
                    if ($calendarauth | checkpermissions($users[$thisuserid]['resourceblock'],"0",$resource['resource']))
                        $resourceselection[]=$resource['resource'];
                }
            } else {
                   $resourceselection[] = $_POST['resource'];
            }

            $actionlogitems = readactionlog();

            foreach($resourceselection as $resource) {
                $allsignups = array();

                if (!$datelimited) {
                    foreach ($users as $usn => $user) {              //get future signups for all users
                        $signup = getusersignups ($usn,true);
                        if (count($signup)>0) {                       //and grab those for this resource
                            foreach ($signup as $item) {
                                $item['usn'] = $usn;
                                if ($item['status'] != "") {
                                    $item['comment'] .= "&nbsp;[" . $item['status'] . "]";
                                }
                                if ($item['resource'] == $resource) $allsignups[] = $item;
                            }
                        }
                    }
                } else {
                    for ($day=strtotime(date("m/d/y",$validatebegin)); $day<strtotime(date("m/d/y",$validateend))+(24*3600); $day += 24*3600) {
                        $signups = getdaysignups($day,$resource);
                        if (count($signups)>0) foreach($signups as $signup) {
                            $item = getusersignups ($signup['usn'],false);
                            $item = $item[$signup['signup']];
                            if (count($item)==0) {
                                $item['signup']    = $signup['signup'];
                                $item['priority']  = $signup['priority'];
                                $item['starttime'] = $day + $signup['hour']*60*60;
                                $item['endtime']   = $item['starttime'] + $signup['length']*60*60;
                                $item['comment'] = "";
                            }
                            if ($item['status'] != "") {
                                $item['comment'] .= "&nbsp;[" . $item['status'] . "]";
                            }
                            $item['usn']    = $signup['usn'];
                            if ($item['usn'] != 0) $allsignups[] = $item;  //add it unless it is a no-user signup
                        }
                    }

                    //add deleted actionlog items
                    if ($billformat) foreach ($actionlogitems as $entry) {
                        $item = $entry['data'];
                        if ($item['resource'] == $resource && $item['ReasonCancel']!="") {
                            $item['comment'] = 'LATE DELETION - ' . $item['ReasonCancel'];
                            $item['signup'] = '';
                            if (($item['endtime'] >= $validatebegin) && ($item['starttime'] <= $validateend+(24*3600)))
                                $allsignups[] = $item;
                        }
                        if ($item['resource'] == $resource && $item['action']=="no-showed") {
                            $item['comment'] = 'NO SHOW - ' . $item['comment'];
                            $item['signup'] = '';
                            if (($item['endtime'] >= $validatebegin) && ($item['starttime'] <= $validateend+(24*3600)))
                                $allsignups[] = $item;
                        }
                    }

                }
                if ($resource == "All" | $resource == "") break;

                ?>
                <b>Signups for <? echo $resourcelist[$resource]['long'] ?>:</b><p>
                <?
                if (count($allsignups) > 0) {
                        $allsignups = sortby($allsignups,'starttime');
                        if ($billformat) $allsignups = sortby($allsignups,'usn');
                        ?> <ul> <?
                        foreach ($allsignups as $item) {
                                ?><li><?
                                //if ($item['endtime']>zulutime() && $item['signup']!='' && !$billformat) {
                                if ($item['signup']!='' && !$billformat) {
                                        echo '<a href="' . schedule_prog . '?UID=' . $UID . '&resource=' . $item['resource'];
                                        echo '&tabledate=' . $item['starttime'] . '&submit=edit&usn=' . $item['usn'] . '&signup=' . $item['signup'] . '">';
                                }
                                echo decodesignup($item,$users[$item['usn']]['firstname'] . " " . $users[$item['usn']]['lastname'],$ashtml);

                                echo "</a><br>\n";
                        }
                        ?> </ul> <?
                } else {
                        ?> <ul><li><b><em>No signups located matching search criteria.</em></b></li></ul><p> <?
                }
            }

            break;

        case "List User Signups" :
        case "listuser" :  //list user signups

                // no user # supplied?
                if (!in_array("usn", array_keys ($_POST)) | $_POST['usn'] == "")
                        if (!in_array("username", array_keys ($_POST)) | $_POST['username'] == "") {
                                //assume lookup of this_user_id
                                $_POST['usn'] = $thisuserid;
                        } else {
                                //try looking up name, if not found, use this user
                                $_POST['usn'] = lookupusn($_POST['username']);
                                if ($_POST['usn'] == "") {
                                    $_POST['usn'] = $thisuserid;
                                }
                        }
                $usn = $_POST['usn'];

                ?> <form name="SignupForm" <? echo 'method="' . submit_method . '" action="' . list_signups_prog . '"'; ?> >
                User:
                <? usermenu($usn,"usn",true,0); ?>
                <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >
                <input type="hidden" name="mode" value="listuser">
                <input type="submit" name="submit" value="List User Signups">
                </form> <?

                $signup = getusersignups ($usn,true);                //get future signups

                validatesignups($usn,true);

                ?>
                <b>Future signups for <? echo $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] ?>:</b><p>
                <?
                if (count($signup) > 0) {
                        $signup = sortby($signup,'starttime');
                        echo "<ul>";
                        foreach ($signup as $item) {
                                echo "<li>";
                                echo '<a href="' . schedule_prog . '?UID=' . $UID . '&resource=' . $item['resource'];
                                echo '&tabledate=' . $item['starttime'] . '&submit=edit&usn=' . $usn . '&signup=' . $item['signup'] . '">';

                                $item['usn'] = $usn;
                                echo decodesignup($item,$resourcelist[$item['resource']]['short'],true);

                                echo "</a><br>\n";
                        }
                        ?> </ul> <?
                } else {
                        ?> <b><em>No future signups located.</em></b><br> <?
                }

                break;
}

scheduleunlock();
printfooter();

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
//JMS 12/29/03
// -added support for status and historical signups (remembered)
// -make sure resources are listed in correct order (as defined by "order")
//JMS 1/9/04
// -dont show no-user (recurring) signups in report
// -allow link to past signups
//JMS 1/18/04
// -modified handling of status
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!