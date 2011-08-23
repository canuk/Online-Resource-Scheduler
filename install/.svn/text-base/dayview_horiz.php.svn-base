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

// Display of week or day view?
if ($_POST['vi'] == 'day') {
    $myweeklength = 1;
    if (dayview_interval==0) 
        define("hour_interval",signup_interval);
    else
        define("hour_interval",dayview_interval);
} else {
    $myweeklength = default_week_length;
    if (weekview_interval==0) 
        define("hour_interval",signup_interval*2);
    else
        define("hour_interval",weekview_interval);
}

// - - - - - - - - - - - - - - - -
// display column header lines

if (count($resourcelist)>0) {
    if ($myweeklength > 1) {
        if (in_array("All",explode("+",$_POST['resource']))){
            if ($_POST['avonly']==1) {
                $todisp = array();
                foreach ($resourcelist as $res=>$item) if ($item['donotshow']!=1 && $_POST['starttime']!="" && $_POST['endtime']!="") $todisp[] = $res;
            } else {
                $todisp = array_keys($resourcelist);                //all = all items in resource list
            }
        } else $todisp = explode("+",$_POST['resource']);        //otherwise, just the resource(s) listed
    } else {
        if (!limit_day_view | in_array("All",explode("+",$_POST['resource']))) {
            if ($_POST['avonly']==1) {
                $todisp = array();
                foreach ($resourcelist as $res=>$item) if ($item['donotshow']!=1 && $_POST['starttime']!="" && $_POST['endtime']!="") $todisp[] = $res;
            } else {
                $todisp = array_keys($resourcelist);                //all = all items in resource list
            }
        } else $todisp = explode("+",$_POST['resource']);        //otherwise, just the resouce(s) listed
    }
} else {
    $todisp = array("0"); //no resources at all
}
echo '<table width="100%" border="1" cellspacing="0" cellpadding="0" bordercolor="#cccccc" bgcolor="#ffffff">';

//put resources into "order" order
$temp = array();
foreach ($todisp as $res)
    if (!$thisuserid || checkpermissions($users[$thisuserid]['resourceblock'],"0",$res)
       || $calendarauth || show_blocked_resources)
            $temp[] = $resourcelist[$res];
$temp = sortby($temp,'order');
$todisp = array();
foreach ($temp as $index=>$item) $todisp[$index] = $item['resource'];

if ($myweeklength > 1)        $resloop = $todisp;
else                         $resloop = array("x");        //only one loop of table

foreach ($resloop as $resloopindex=>$resource) {                //disp a table for each resource in "resource" array (one or all!)

if ($_POST['vi'] == 'week') {
    // Week View (days at top of each column)

    //Show weekends?
    if (noweekends) {
        if (date("w",$tabledate)==6) $tabledate += 2*60*60*24;  //sat->next monday
        $tabledate = $tabledate - (date("w",$tabledate)-1)*60*60*24;  //**NO WEEKEND MOD** round to previous monday (or next monday if we're on sunday)
        $myweeklength=5;
    }

    // Week View (days at left of each column)
    $mydate = $tabledate;
    for ($dow = 1; $dow < $myweeklength+1; $dow++) {
        if (noweekends && (date("w",$mydate)==0 || date("w",$mydate)==6)) continue;  //**NO WEEKEND MOD** 
        $week[$dow] = getday($mydate,$resource);                        //get this day's signups
        $week[$dow] = combineopen($week[$dow]);                         //and compress open slots for table
//        $mydate = strtotime(date( "m",$mydate) . "/" . (date("d",$mydate)+1) . "/" . date("Y",$mydate));
        $mydate = mktime(0,0,0,date( "m",$mydate),date("d",$mydate)+1,date("Y",$mydate));
    }

    $ncolumns = $myweeklength;

    $otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&vi=' . $_POST['vi'] . '&';

    //Grand heading for each resource table
    echo "<tr><td colspan=\"";
    echo count($week[1]) + 1;
    echo "\"><b><center>";
    if (count(explode("+",$_POST['resource'])) > 1 | in_array("All",explode("+",$_POST['resource'])) )        //when more than one is selected, make resource "headings" hyperlinked to choose that resource
    echo "<a href=\"" . schedule_prog . '?' . $otherforminfo . "re=" . $resource . "&ta=" . $tabledate . "\">";
    echo $resourcelist[$resource]['long'] . "</a>&nbsp;";

    resourcelink($UID,$resourcelist[$resource]);  //add resource info links if necessary

    echo "</center></b></td></tr>";

} else {
    // Day View (resources at left of each column)

    //Show weekends?
    if (noweekends) {
        if (date("w",$tabledate)==6) $tabledate += 2*60*60*24;  //sat->next monday
        if (date("w",$tabledate)==0) $tabledate -= 2*60*60*24;  //sun->last friday
    }

    $mydate = $tabledate;

    //Read in each resource's signups
    foreach ($todisp as $loc=>$ind) {
        $week[$loc+1] = getday($mydate,$ind);                                        //get this resources's signups
        $week[$loc+1] = combineopen($week[$loc+1]);                                 //and compress open slots for table
    }

    $otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&vi=' . $_POST['vi'] . '&';

    //Grand heading for the table
    echo "<tr><td colspan=\"";
    echo count($week[1]) + 1;
    echo "\"><b><center>";

    echo '<table width=100% border=0><tr><td align="left" width=60>';
    echo '<font size="-1"><a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($tabledate - (24*60*60*($myweeklength))) . '">&lt; Previous</a></font>';
    echo '</td>';

    echo '<td align="center"><center><b>' . date("D " . dateformat, $mydate) . "</b></center></td>";

    echo '<td align="right" width=60>';
    echo '<font size=-1><a href="' . schedule_prog . '?' . $otherforminfo . 'tabledate=' . ($tabledate + (24*60*60*($myweeklength))) . '">Next &gt;</a></font>';
    echo '</td></tr></table>';
    echo "</center></td></tr>";


    $ncolumns = count($todisp);

}

$colorcode = array(1=>"green", 2=>"yellow", 3=>"red");

// - - - - - - - - - - - - - - - -
//draw actual signup table for one resource or day
if (count($week[1])>0) {

    echo '<tr valign="top" align="center">';

    if ($_POST['vi'] == 'week') {
        echo '<td nowrap bgcolor=#dddddd><font size="-1">';
        echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($tabledate - (24*60*60*($myweeklength))) . '">&uarr;Prev</a></font></td>';
    } else {
        echo '<td nowrap bgcolor=#dddddd><font size="-1">';
        echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($tabledate - (24*60*60)) . '">&uarr;Prev</a></font></td>';
    }
    foreach (array_keys($week[1]) as $hour) {
        echo "<td bgcolor=#dddddd width=60><font size=\"-1\">";
        if (($hour - floor($hour)) < (1/60))
            echo date(timeformat,mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate),date("Y",$tabledate)));
        else
            echo "&nbsp;&nbsp;&nbsp;" .date(":i",mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate),date("Y",$tabledate)));
        echo "</font></td>\n";
    }
    echo '</tr>';

    for ($dow = 1; $dow <= $ncolumns; $dow++) {

        if (noweekends && (date("w",$datestamp)==0 || date("w",$datestamp)==6)) continue;  //**NO WEEKEND MOD**  skip to next cycle of loop if sat or sun

        for ($priority = 1; $priority <= num_signups; $priority++) {
                
            echo '<tr valign="center" align="left">';
            if ($priority==1 && $_POST['vi'] == 'week') {
                $mydate = mktime(0,0,0,date( "m",$tabledate),date("d",$tabledate)+$dow-1,date("Y",$tabledate));

                //Date column
                $day_of_week = date("w",$mydate); 
                if (($day_of_week == 0) || ($day_of_week == 6)){ 
                    $columncolor = weekend_color;
                } else {
                    $columncolor = weekday_color;
                } 
                echo '<td nowrap align="center" bgcolor=' . $columncolor . ' rowspan=' . num_signups . '><font size=-1>';
                if (date("m/d/Y",$mydate) == date("m/d/Y",zulutime())) echo "<b>";
                echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($mydate) . '">';
                echo date("D " . dateformat, $mydate) . "</a></b></font></td>";           //display date

                $ncolumns = $myweeklength;

            } elseif ($priority==1) {
                // Day View (resources at left of each row)
                $otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&vi=' . $_POST['vi'] . '&';

                $mydate = $tabledate;

                //Resource column

                echo "<td nowrap bgcolor=#dddddd rowspan=\"" . num_signups . "\"><font size=\"-1\">";
                if (in_array($todisp[$dow-1],explode("+",$_POST['resource']))) echo "<b>";
                echo "<a href=\"" . schedule_prog . '?' . $otherforminfo . "re=" . $todisp[$dow-1] . "&ta=" . $tabledate . "\">";
                echo $resourcelist[$todisp[$dow-1]]['short'] . "</a></b></font>&nbsp;";
                resourcelink($UID,$resourcelist[$todisp[$dow-1]]);  //add resource info links if necessary
                echo "</td>";                  //display resource name (1st word)
  
                $ncolumns = count($todisp);

            }

            foreach (array_keys($week[1]) as $hour) {
                $data = $week[$dow][$hour];

                if ($_POST['vi'] == 'week')         $datestamp = mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate)+$dow-1,date("Y",$tabledate));
                else                                         $datestamp = mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate),date("Y",$tabledate));;

                if (($hour/3 - floor($hour/3))==0) {
                    $blankcolor     = "ruleline.gif";
                    $blankcolorcode = "";
                } else {
                    $blankcolor = "white.gif";
                    $blankcolorcode = "";
                }

                if (is_array($data[$priority])) {
                    //-- Filled time block, give "edit" and "info" options --
                    $info = getusersignups($data[$priority]['usn'],false);
                    if (!$data[$priority]['recurring'])  {
                        $info = $info[$data[$priority]['signup']];
                    } else {
                        $info = $data[$priority];  // this is all we've got for a recurring
                    }

                    if ($data[$priority]['usn']==0) {
                        $mycolor = "recurring";
                    } elseif (color_from_status) {
                        if ($info['status']=="") {
                            $mycolor = "red";
                        } elseif ($info['status']==0) {
                            $mycolor = "green";
                        } else {
                            $mycolor = "yellow";
                        }
                    } elseif (!automanage_alternates) {
                        $mycolor = "green";                
                    } else {
                        $mycolor = $colorcode[$priority];
                        if ($mycolor == "") $mycolor = "red";
                    }

                    if (floor($datestamp/60/60/24) < floor($info['starttime']/60/60/24)) $info = array();
                    if (strtolower($info['comment']) == "wx") $mycolor = "blue";

                    $otherforminfo = 'UID=' . $UID . '&ta=' . $tabledate . '&usn=' . $data[$priority]['usn'] . '&si=' . $data[$priority]['signup'] . '&re=' . $info['resource'] . '&vi=' . $_POST['vi'] . '&';
                    if (in_array("print",array_keys($_POST))) $mycolor = "white";

                    if ($datestamp >= floor($info['starttime']/60/60/hour_interval)*60*60*hour_interval
                         & $_POST['usn'] == $data[$priority]['usn'] & $_POST['signup'] == $data[$priority]['signup'] )
                            $mycolor = "selected";

                    if ($thisuserid || public_show_comment)
                        $comment = ' alt="' . stripcslashes($info['comment']) . '" ';

                    $content = '';
                    $link    = '';
                    if ($data[$priority]['usn']!=0) {  //if this isn't a "no-user" signup
                        if ($calendarauth || allow_past_edit || ($datestamp+$data[$priority]['length']*60*60) >= (zulutime() + timezone + allow_signup_offset)) {
                            if ($_POST['usn'] == $data[$priority]['usn']
                                 & $_POST['signup'] == $data[$priority]['signup']
                                 & ($_POST['submit'] == "edit" | $_POST['submit']=="Delete")) {
                                    $content .= '<b><em>';
                            } elseif ($thisuserid && ($thisuserid == $data[$priority]['usn'] | $calendarauth | edit_other_users)) {
                                $link = schedule_prog . '?submit=edit&' . $otherforminfo;
                                $content .= '<a href="' . schedule_prog . '?submit=edit&' . $otherforminfo . '"' . $comment . '>';
                                $content .= '<b><em>';
                            } elseif ($thisuserid) {
                                $link = schedule_prog . '?submit=info&' . $otherforminfo;
                                $content .= '<a href="' . schedule_prog . '?submit=info&' . $otherforminfo . '"' . $comment . '>';
                            }
                        }
                    }

                    if ($info['starttime']%(hour_interval*60*60)>0 & date("m/d/Y",$info['starttime'])==date("m/d/Y",$datestamp)) 
                        $content .= '<img src="' . $blankcolor . '" width=22 height=100% align=center border=0>';

                    $signupdesc = "";
                    if ($thisuserid) {
                        if (show_comment && $info['comment']!="") {   //show signup's comment in block
                            $signupdesc = wordwrap(stripcslashes($info['comment']),hyphen_length,"<br>",1);
                        }
                        if ($signupdesc == "" || show_name) {  //force show name OR nothing in description yet
                            if (strlen($signupdesc)>0) $signupdesc = " - " . $signupdesc;
                            if (show_full_name) {
                                $signupdesc = hyphenated($users[$data[$priority]['usn']]['firstname'] . " " . $users[$data[$priority]['usn']]['lastname']) . $signupdesc;
                            } else {
                                if (show_first_name && $users[$data[$priority]['usn']]['firstname'] != "") {
                                    $signupdesc = hyphenated($users[$data[$priority]['usn']]['firstname']) . $signupdesc;
                                } else {
                                    $signupdesc = hyphenated($users[$data[$priority]['usn']]['lastname']) . $signupdesc;
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
                                $signupdesc = hyphenated($users[$data[$priority]['usn']]['firstname'] . " " . $users[$data[$priority]['usn']]['lastname']) . $signupdesc;
                            } else {
                                if (public_show_first_name && $users[$data[$priority]['usn']]['firstname'] != "") {
                                    $signupdesc = hyphenated($users[$data[$priority]['usn']]['firstname']) . $signupdesc;
                                } else {
                                    $signupdesc = hyphenated($users[$data[$priority]['usn']]['lastname']) . $signupdesc;
                                }
                            }
                        }
                    }
                    $content .= $signupdesc;

                    //add WX flag and/or * if comment exists or not even hour start or end
                    if (strtolower($info['comment'])=="wx") {
                        $content .= " (WX)";
                        if ($info['starttime']%(hour_interval*60*60)>0) $content .= "*";
                    } else if (($info['comment']!="" & !show_comment) | $info['starttime']%(hour_interval*60*60)>0) $content .= "*";

                    $content .= '</b></em>';

                    //add comments as mouse-over alt display
                    $mouseover = 'Comment: ' . strtr(stripslashes($info['comment']),array('<br>'=>''));

                    //does block continue partially into next time block? draw something to point that out
                    if ($info['endtime']%(hour_interval*60*60)>0 & date("m/d/Y",$info['endtime'])==date("m/d/Y",$datestamp)) 
                        $content .= "<font size=\"-1\">(end +" . date("G:i",mktime(0,$info['endtime']%(hour_interval*60*60)/60)) . ")</font>";

                    echo '<td colspan="' . $data[$priority]['rowspan'] . '" background="' . $mycolor . '.gif" ';
                    if ($thisuserid || public_show_comment) {
                        echo 'onmouseover = "javascript:window.status=\'' . $mouseover . '\';" ';
                        echo 'onmouseout  = "javascript:window.status=\'\';" ';
                        if (!empty($info['comment'])) echo 'title="' . stripcslashes($info['comment']) . '" ';
                    }
                    if (!empty($link)) echo 'onclick="javascript:window.location=\'' . $link . '\';" ';
                    echo '><font size="-1">';
                    echo $content . "</a>&nbsp;</font></td>\n";

                } else {
                    //-- Empty time block, give "select this time block" options --

                    $mycolor = $blankcolor;
                    $mycolorcode = $blankcolorcode;
                    if ($data[$priority]) {
                        $content = '';
                        $link = '';
                        if ($thisuserid) {
                            if (($_POST['submit'] != "edit") && ($datestamp >= (zulutime()+timezone + allow_signup_offset) | $administrator)) {
                                if ($_POST['vi'] == 'week') {
                                    $otherforminfo = 'UID=' . $UID . '&ta=' . $tabledate . '&co=' . make_get_str($_POST['comment']) . '&re=' . $resource . '&vi=' . $_POST['vi'] . '&';
                                    $showstart = True;
                                } else {
                                    $otherforminfo = 'UID=' . $UID . '&ta=' . $tabledate . '&co=' . make_get_str($_POST['comment']) . '&re=' . $todisp[$dow-1] . '&vi=' . $_POST['vi'] . '&';
                                    $showstart = (in_array($todisp[$dow-1],explode("+",$_POST['resource'])) | in_array("All",explode("+",$_POST['resource'])));
                                }
                                if (in_array("starttime", array_keys ($_POST)) & $showstart) {

                                    //determine if this is the start and/or end time block
                                    $startoffset = round(($datestamp/60/60/hour_interval) - ($_POST['starttime']/60/60/hour_interval));
                                    $endoffset   = round(($datestamp/60/60/hour_interval) - ($_POST['endtime']/60/60/hour_interval));
                                    $isstarttime = $startoffset>-1 & $startoffset<=0;
                                    $isendtime   = $endoffset>=-1 & $endoffset<0;
                                    $minblocks   = round(min_signup_time/hour_interval);

                                    if ($isstarttime & !in_array($_POST['submit'],array("info","Delete","No-Show","edit"))) {
                                        if (!$isendtime) {
                                            //actual <start> cell
                                            if ($minblocks==1) {
                                                $link = schedule_prog . '?' . $otherforminfo . 'st=' . $_POST['starttime'];
                                                $content .= '<a href="' . schedule_prog . '?' . $otherforminfo . 'st=' . $_POST['starttime'];
                                                $content .= '&en=' . ($datestamp+(hour_interval*60*60)) . '">';
                                            }
                                                $content .= '&lt;<em><font size="-1">START</font></em>&gt;';
                                            } else {
                                                //actual <start> AND <end> cell
                                                $content .= '&lt;<em>START / END</em>&gt;';
                                            }
                                    } elseif ($isendtime & !in_array($_POST['submit'],array("info","Delete","No-Show","edit"))) {
                                        //actual <end> cell
                                            $content .= '&lt;<em><font size="-1">END</font></em>&gt;';
                                    } elseif ($startoffset+1>=$minblocks) {
                                        //cells AFTER and immediately BEFORE <end>  (NEW END TIME)
                                            $link = schedule_prog . '?' . $otherforminfo . 'st=' . $_POST['starttime'] . '&en=' . ($datestamp+(hour_interval*60*60));
                                            $content .= '<a href="' . schedule_prog . '?' . $otherforminfo . 'st=' . $_POST['starttime'];
                                            $content .= '&en=' . ($datestamp+(hour_interval*60*60)) . '">';
                                    } elseif ($_POST['endtime']!=0 && $startoffset>0 && abs($endoffset)>=$minblocks) {
                                        //cells immediately AFTER <start> which are valid start times (NEW START TIME)
                                        $link = schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp . '&en=' . $_POST['endtime'];
                                        $content .= '<a href="' . schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp . '&en=' . $_POST['endtime'] . '"';
                                        $content .= 'style="{text-decoration: none}">';
                                        $content .= '&rarr;';
                                    } elseif ($startoffset<=0 & abs($endoffset)>=$minblocks) {
                                        //cells BEFORE <start> (NEW START TIME)
                                            $link = schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp . '&en=' . $_POST['endtime'];
                                            $content .= '<a href="' . schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp . '&en=' . $_POST['endtime'] . '">';
                                    } else {
                                        //INVALID TIMES because block would be too short
                                        $content .= "&rarr;";
                                    }
                                } else {
                                    $link = schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp;
                                    $content .= '<a href="' . schedule_prog . '?' . $otherforminfo . 'st=' . $datestamp . '">';
                                }
                            }
                        }
                        $content .= "</a>&nbsp;</td>\n";
                        echo '<td rowspan="' . $data[$priority] . '"' . $mycolorcode . ' background=' . $mycolor . ' height=22 ';
                        if (!empty($link)) echo 'onclick="javascript:window.location=\'' . $link .'\';" ';
                        echo '>' . $content;
                    }
                } //end empty cell IF
            } //hour loop
        } //end priority FOR
        echo "</tr>\n";
    }


    $otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&vi=' . $_POST['vi'] . '&';
    echo '<tr valign="top" align="center">';
    if ($_POST['vi'] == 'week') {
        echo '<td nowrap bgcolor=#dddddd><font size="-1">';
        echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($tabledate + (24*60*60*($myweeklength))) . '">&darr;Next</a></font></td>';
    } else {
        echo '<td nowrap bgcolor=#dddddd><font size="-1">';
        echo '<a href="' . schedule_prog . '?' . $otherforminfo . 'ta=' . ($tabledate + (24*60*60)) . '">&darr;Next</a></font></td>';
    }
    foreach (array_keys($week[1]) as $hour) {
        echo "<td bgcolor=#dddddd width=60><font size=\"-1\">";
        if (($hour - floor($hour)) == 0)
            echo date(timeformat,mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate),date("Y",$tabledate)));
        else
            echo "&nbsp;&nbsp;&nbsp;" .date(":i",mktime(floor($hour),($hour-floor($hour))*60,0,date( "m",$tabledate),date("d",$tabledate),date("Y",$tabledate)));
        echo "</font></td>\n";
    }
    echo '</tr>';

}

echo "</tr>\n";

} //end of foreach resource loop

echo "</table>\n";

//==============================================================================
//JMS 3/16/02
//  -move next and prev. in day view up to date line
//JMS 4/1/02
// -added full support of viewing only selected resources
// -fixed "wrong year" bug (use ".../Y" in date tests)
//JMS 11/14/02
// -added complete am/pm clock support
//CBW 11/23/02
// -fixed bug in resource listing when no resources exist (after initial installation)
//CBW 1/12/03
// -fixed bug with "resourceblock" field in userlist.csv file which used to
//      be limited to 15 resources.  The integer field was changed to a
//      "b-format" string which should be limitless (within reason)
//JMS 7/30/03
// -fixed 1/2-hour start bug which kept signup being edited from being shown as selected (week view only)
//JMS 8/18/03
// -revised handling of offset signups (length not even interval of hour_interval)
// -made hour_interval a configurable setting (with 0=automatic mode)
//JMS 10/01/03
// -fixed wrong-resource selected bug caused when limit_day_view is True
//JMS 11/01/03
// -added bottom-of-table next/prev and date/resource row
// -added public_lastname_view option
//JMS 12/11/03
// -fixed empty-resource bug
// -added comment_only option for time blocks
//JMS 12/13/03
// -converted checksec calls to variables
// -fixed logic of highlighting currently selected signup
// -added edit_other_users option
//JMS 12/30/03
// -separated logic of "myweeklength" from number of columns
//JMS 1/5/04
// -added euro-date support
//JMS 1/9/04
// -added different color for recurring signups
//JMS 1/27/04
// -added more name/comment display options
//JMS 2/5/04
// -added resource comment display
//JMS 2/6/04
// -added full suite of signup lableling configuration options
// -added better "zero resources" support
// -added better do-not-show resource
//JMS 2/7/04
// -fixed IE/table problems
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
?>
