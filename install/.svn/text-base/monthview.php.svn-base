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

if (count($resourcelist)>0) {
    if (in_array("All",explode("+",$_POST['resource'])))
        if ($_POST['avonly']==1) {
            $todisp = array();
            foreach ($resourcelist as $res=>$item) if ($item['donotshow']!=1 && $_POST['starttime']!="" && $_POST['endtime']!="") $todisp[] = $res;
        } else {
            $todisp = array_keys($resourcelist);                //all = all items in resource list
        }
    else $todisp = explode("+",$_POST['resource']);        //otherwise, just the resouce(s) listed
} else {
    $todisp = array(); //no resources at all
}

//put resources into "order" order
$temp = array();
foreach ($todisp as $res)
        if (!$thisuserid | checkpermissions($users[$thisuserid]['resourceblock'],"0",$res)
           | checksec($users[$thisuserid]['sec'],10) | show_blocked_resources)
                $temp[] = $resourcelist[$res];
$temp = sortby($temp,'order');
$todisp = array();
foreach ($temp as $index=>$item) $todisp[$index] = $item['resource'];

// - - - - - - - - - - - - - - - -
//draw one week's signups

$otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&';

echo '<table width="100%" border="0" cellspacing="0" cellpadding="0" bordercolor="#cccccc" bgcolor="#ffffff">';
echo '<tr><td align="left" width=60><font size=-1>';

//$targetdate = strtotime((date("m",$tabledate)-1) . "/1/" . date("Y",$tabledate));
$targetdate = mktime(0,0,0,date( "m",$tabledate)-1,1,date("Y",$tabledate));
if (noweekends) {
    if (date("w",$targetdate)==7) $targetdate += 60*60*24;
    if (date("w",$targetdate)==0) $targetdate += 60*60*24;
}

echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'vi=month&ta=' . $targetdate . '">';
echo '&lt; Previous';
echo '</a></font></td>';

echo '<td align="center"><b><font size=+1>' . date("F Y",$tabledate);
if (count($todisp)==1) echo " - " . $resourcelist[$todisp[0]]['long'];
echo "</font></b></td>";

echo '<td align="right" width=60><font size=-1>';

//$targetdate = strtotime((date("m",$tabledate)+1) . "/1/" . date("Y",$tabledate));
$targetdate = mktime(0,0,0,date( "m",$tabledate)+1,1,date("Y",$tabledate));
if (noweekends) {
    if (date("w",$targetdate)==7) $targetdate += 60*60*24;
    if (date("w",$targetdate)==0) $targetdate += 60*60*24;
}

echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'vi=month&ta=' . $targetdate . '">';
echo 'Next &gt;';
echo '</a></font></td></tr></table>';

//echo '</td></tr>';
echo '<table width="100%" border="1" cellspacing="0" cellpadding="0" bordercolor="#cccccc" bgcolor="#ffffff">';

echo '<tr>';
if (!noweekends) {
    echo '<td align="center"><b>Sun</b></td>';
}
echo '<td align="center"><b>Mon</b></td>';
echo '<td align="center"><b>Tue</b></td>';
echo '<td align="center"><b>Wed</b></td>';
echo '<td align="center"><b>Thu</b></td>';
echo '<td align="center"><b>Fri</b></td>';
if (!noweekends) {
    echo '<td align="center"><b>Sat</b></td>';
}
echo '</tr>';

$tabledate = strtotime(date("m/d/Y",$tabledate));
$firstdom  = strtotime(date("m/1/Y",$tabledate));
$monthoffset = date("w",$firstdom);

for ($myweek = 0; $myweek < 6; $myweek++) {

        $otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&';

        $mysunday = $myweek*7*24*60*60 + 60*60*2 + $firstdom - $monthoffset*24*60*60;        //calc. date of last Sunday
        if ($myweek == 0 | date("m",$mysunday) == date("m",$tabledate)) {

        echo '<tr valign="top" align="left">';
        if (noweekends) {          //**NO WEEKEND MOD** 
            $startday = 1;
            $ndays    = 6;
        } else {
            $startday = 0;
            $ndays    = 7;
        }
        for ($dow = $startday; $dow < $ndays; $dow++) {

                $mydate = mktime(0,0,0,date( "m",$mysunday),date("d",$mysunday)+$dow,date("Y",$mysunday));

                $istabledate = (date("m/d/Y",$mydate)==date("m/d/Y",$tabledate));
                $istoday     = (date("m/d/Y",$mydate)==date("m/d/Y",(zulutime() + timezone)));
                if ($istabledate)
                        echo "<td bgcolor=#ffffaa>";
                else {
                        if ($istoday) echo '<td bgcolor=#aaffaa>';
                        else echo '<td bgcolor=#eeeeee>';
                        $targetdate = ($mydate);
                        if (default_view == 'week')  $targetdate = $mysunday;                //make sunday first day if we switch to week view
                        echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'vi=&ta=' . ($targetdate) . '">';
                }
                if (date("m",$mydate) == date("m",$tabledate))
                        echo '<font color=#000000 size="-1">';
                else
                        echo '<font color=#bbbbbb size="-1">';

                if (date("j",$mydate)=="1" )
                        echo "<b>" . date("M j",$mydate);
                else {
                        if ($istoday) echo "<b>";
                        echo date("j",$mydate);
                }

                if (!$istabledate & !$istoday) echo "<img src='ruleline.gif' border=0 width=80 height=12>";
                if ($istoday) echo "&nbsp;&nbsp;&nbsp;<em>Today</em>";
                echo "</font></a></b></td>\n";
        }

        echo '</tr>';
        echo '<tr valign="top" align="left">';
        for ($dow = $startday; $dow < $ndays; $dow++) {

                $mydate = mktime(0,0,0,date( "m",$mysunday),date("d",$mysunday)+$dow,date("Y",$mysunday));

                if (date("m",$mydate) == date("m",$tabledate))
                        echo '<td><font size="-1">';
                else
                        echo '<td bgcolor="#eeeeee"><font size="-1">';

                $datestamp = $mydate;

                $allsignups = array();
                foreach ($todisp as $resource) {
                        $signups = getdaysignups($mydate,$resource);        //get this resources's signups
                        if (count($signups)>0) foreach ($signups as $onesignup) {
                                $onesignup['resource']  = $resource;
                                $onesignup['starttime'] = $onesignup['hour'];
                                if ($onesignup['usn'] != 0) $allsignups[]           = $onesignup;
                        }
                }

                if (count($allsignups)>0)
                    $allsignups = sortby($allsignups,'starttime');
                    foreach ($allsignups as $onesignup) {
                    
                        if (color_from_status || show_comment) {
                            //if one of the modes requires it, get details for this signup
                            $info = getusersignups($onesignup['usn'],false);
                            if (count($info)>0) {
                                $info = $info[$onesignup['signup']];
                            } else {
                                $info['comment'] = '';                                        
                                $info['status'] = '';
                            }
                        }

                        $otherforminfo = 'UID=' . $UID . '&ta=' . $tabledate . '&usn=' . $onesignup['usn'] . '&si=' . $onesignup['signup'] . '&re=' . $onesignup['resource'] . '&vi=' . $_POST['vi'] . '&';

                        if ((allow_past_edit || $calendarauth || floor(($datestamp)/60/60/24) >= floor((zulutime() + timezone)/60/60/24)) && $thisuserid!="")
                                echo '<a href="' . schedule_prog . '?submit=edit&' . $otherforminfo . '">';

                        //choose color for signup
                        if ($_POST['usn'] == $onesignup['usn'] & $_POST['signup'] == $onesignup['signup'])
                                echo "<font color=#333333>";
                        elseif (color_from_status) {
                                if ($info['status']=="") {
                                    echo "<font color=#990000>";
                                } elseif ($info['status']==0) {
                                    echo "<font color=#009900>";
                                } else {
                                    echo "<font color=#999900>";
                                }
                        } elseif (!automanage_alternates) {
                            echo "<font color=#009900>";
                        } else {
                            if ($onesignup['priority'] == 1) echo "<font color=#009900>";
                            elseif ($onesignup['priority'] == 2) echo "<font color=#999900>";
                            elseif ($onesignup['priority'] >  2) echo "<font color=#990000>";
                        }

                        if (count($todisp)>1) echo "<b>" . $resourcelist[$onesignup['resource']]['initials'] . "</b>:";

                        if ($onesignup['hour'] == 0)
                                echo " &lt; ";
                        else
                                echo " " . date(timeformat,$onesignup['hour']*60*60 + $mydate);

                        if ($onesignup['hour'] + $onesignup['length'] == 24)
                                echo "- &gt;";
                        else
                                echo "-" . date(timeformat,($onesignup['hour'] + $onesignup['length'])*60*60 + $mydate);

                        echo " ";
                        
                        $signupdesc = "";

                        if ($thisuserid) {
                            if (show_comment && $info['comment']!="") {   //show signup's comment in block
                                $signupdesc = wordwrap(stripcslashes($info['comment']),hyphen_length,"<br>",1);
                            }
                            if ($signupdesc == "" || show_name) {  //force show name OR nothing in description yet
                                if (strlen($signupdesc)>0) $signupdesc = " - " . $signupdesc;
                                if (show_full_name) {
                                    $signupdesc = hyphenated($users[$onesignup['usn']]['firstname'] . " " . $users[$onesignup['usn']]['lastname']) . $signupdesc;
                                } else {
                                    if (show_first_name && $users[$onesignup['usn']]['firstname'] != "") {
                                        $signupdesc = hyphenated($users[$onesignup['usn']]['firstname']) . $signupdesc;
                                    } else {
                                        $signupdesc = hyphenated($users[$onesignup['usn']]['lastname']) . $signupdesc;
                                    }
                                }
                            }
                        } else {  //limited public view
                            if (public_show_comment && $info['comment']!="") {   //show signup's comment in block
                                $signupdesc = wordwrap(stripcslashes($info['comment']),hyphen_length,"<br>",1);
                            }
                            if (public_show_name) { 
                                if (strlen($signupdesc)>0) $signupdesc = " - " . $signupdesc;
                                if (public_show_full_name) {
                                    $signupdesc = hyphenated($users[$onesignup['usn']]['firstname'] . " " . $users[$onesignup['usn']]['lastname']) . $signupdesc;
                                } else {
                                    if (public_show_first_name && $users[$onesignup['usn']]['firstname'] != "") {
                                        $signupdesc = hyphenated($users[$onesignup['usn']]['firstname']) . $signupdesc;
                                    } else {
                                        $signupdesc = hyphenated($users[$onesignup['usn']]['lastname']) . $signupdesc;
                                    }
                                }
                            }
                        }
                        echo $signupdesc; //wordwrap($signupdesc,hyphen_length,"<br>",1);

                        echo "<br></font></a>\n";

                    }

                if (date("m/d/Y",$datestamp) == date("m/d/Y",0+$_POST['starttime']) & !in_array($_POST['submit'],array("info","Delete","No-Show","edit"))) {
                        if (date("m/d/Y",$datestamp) != date("m/d/Y",0+$_POST['endtime'])) {
                                echo '&lt;<em><font size="-1">START</font></em>&gt;';
                        } else
                                echo '&lt;<em>START / END</em>&gt;';
                } elseif (date("m/d/Y",$datestamp) == date("m/d/Y",0+$_POST['endtime']) & !in_array($_POST['submit'],array("info","Delete","No-Show","edit")))
                        echo '&lt;<em><font size="-1">END</font></em>&gt;';

                if (count($allsignups)<3) echo "<br>";
                if (count($allsignups)<2) echo "<br>";

                echo "</td>\n";

        }
        echo '</tr>';
        }        //end of test for next month


}

echo "</table>";

foreach ($todisp as $resource) {
    resourcelink($UID,$resourcelist[$resource], true);
}

//==============================================================================
//JMS 4/1/02
// -added full support of viewing only selected resources
// -fixed "wrong year" bug (use ".../Y" in date tests)
//JMS 9/30/02
// -fixed daylight savings time bug (calculate current day of week from textual date)
//JMS 11/14/02
// -added complete am/pm clock support
// -fixed day-to-day indicator (<->) to work with new full-day signup style (0:00-24:00)
//CBW 1/12/03
// -fixed bug with "resourceblock" field in userlist.csv file which used to
//      be limited to 15 resources.  The integer field was changed to a
//      "b-format" string which should be limitless (within reason)
//JMS 11/1/03
// -added public_lastname_view option
//JMS 12/11/03
// -fixed empty-resource bug
// -added edit_other_users option
//JMS 12/29/03
// -added color from status option
//JMS 1/27/04
// -added more name/comment display options
//JMS 2/6/04
// -added full suite of signup lableling configuration options
//JMS 2/7/04
// -fixed IE/table problems
//JMS 7/29/04
//  -fixed next/prev bug with noweekends mode
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
?>
