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

if (file_exists('index.html') && !file_exists('data')) {
    //not installed yet...
    include('index.html'); 
    return(null);
}

require_once('functions.php');

?>

<html>
<head>
<title>Scheduling Service</title>
<meta name="description" content="Shareware ORS (Online Resource Scheduler) at ORS.sourceforge.net is a full featured
web interface for scheduling resources (aircraft, boats, rooms, time-shares) by members of an
organization. It provides a graphical interface, e-mail notification of changes and account
administration. See ors.sourceforge.net">
<meta name="keywords" content="online scheduling, on-line scheduling, online resource scheduler, ORS,
aircraft scheduling, boat scheduling, room scheduling, time-share scheduling, office scheduling,
lab equipment scheduling, php, apache">
<meta http-equiv="cache-control" content="private">

    <script type="text/javascript" src="jscalendar-1.0/calendar.js"></script>
    <script type="text/javascript" src="jscalendar-1.0/calendar-setup.js"></script>
    <script type="text/javascript" src="jscalendar-1.0/lang/calendar-en.js"></script>
    <style type="text/css"> @import url("jscalendar-1.0/calendar-blue.css"); </style>
	<script type="text/javascript">
	  function ispast(date, y, m, d) {
	    today = new Date();
	    return(date<Math.floor(today/24/60/60/1000)*24*60*60*1000);
	  }
	</script>

<link rel="stylesheet" href="ors.css" type="text/css">
<style>
    .signuptable #titlerow { background-color: <? echo menu_color;?>; }
    .actionsmenu { background-color: <? echo menu_color;?>; }
</style>

</head>
<?php

echo '<body background="' . page_background . '" nosave>';

$do_not_noshow = array('Wax','Maintenance');        //users which can not be "no-showed"

//====================================================
//Beginning of main routine
//====================================================

//copy all items from get and sanitize all values
foreach ($_GET as $key=>$value) $_POST[$key]=$value;
foreach ($_POST as $key=>$value) $_POST[$key]=sanitize($value);

$_POST['REMOTE_ADDR'] = $_SERVER['REMOTE_ADDR'];

//convert each two-letter "code" over to the actual field
// (we use shortened codes in the table to reduce file size)
foreach (array(
                'st'=>'starttime',
                'en'=>'endtime',
                'co'=>'comment',
                're'=>'resource',
                'ta'=>'tabledate',
                'si'=>'signup')

         as $code => $actual)

        if (in_array($code, array_keys($_POST))) {
                $_POST[$actual] = $_POST[$code];        //copy to real field
                unset($_POST[$code]);                                //and remove shorhand field
        }


//convert array-based resource list to + separated list
if (is_array($_POST['resource'])) {
        $temp = $_POST['resource'][0];
        unset($_POST['resource'][0]);
        if (count($_POST['resource']) > 0)
                foreach ($_POST['resource'] as $item) $temp .= "+" . $item;
        $_POST['resource'] = $temp;
        unset($temp);
}
$_POST['resource'] = str_replace(" ", "+", $_POST['resource']);  //in case input was space separated string

if ($_POST['vi'] == "") $_POST['vi'] = default_view;

if (!schedulelock($_POST['UID'])) {
        echo "<h3><center>Sorry, Scheduling is currently busy. Please check back shortly.</center></h3>";
        return(false);
}

$warning = array();                                //empty warning

//non-secure Logout command
if (in_array("submit", array_keys ($_POST))
        & in_array("UID", array_keys ($_POST))
        & $_POST['submit']=="logout") {
                logoutuid($_POST['UID']);
                $_POST['UID'] = "";
                $_POST['submit'] = "";
}
if ($_POST['tabledate'] == "") unset($_POST['tabledate']);
if ($_POST['resource'] == "") {
     if (default_resource_all && count($resourcelist)!=1) $_POST['resource'] = "All"; else $_POST['resource'] = 1;
}

//use remote authorization if no UID found (must be enabled in configuration)
$_POST['REMOTE_USER']=$REMOTE_USER;

//Can we identify this user based on the posted vars (UID or lastname/password)
$userinfo   = useridlookup($_POST);
$thisuserid = $userinfo['usn'];
$UID        = $userinfo['UID'];
$_POST['thisuserid'] = $thisuserid;        //copy to use in functions

//Read in necessary files
$resourcelist = getresources();                        //get list of valid resources
$users        = getusers();
$members      = getmemberlist();

$selectedres = explode("+",$_POST['resource']); //make array of selected resources

$administrator = checksec($users[$thisuserid]['sec'],100);
$calendarauth  = checksec($users[$thisuserid]['sec'],10);
$signuppermitted  = checksec($users[$thisuserid]['sec'],1);

//Give top-of-page form (as appropriate for logged-in or not)
?>
<form name="SignupForm" <? echo 'method="' . submit_method . '" action="' . schedule_prog . '?"'; ?> >
<?

if (!$thisuserid)        { //no valid username/password has been provided
    //Strip private resources from the resource list (those with "hidepublic" field set)
    unset($resourcetemp);
    foreach ($resourcelist as $res) if (!$res['hidepublic']) $resourcetemp[$res['resource']]=$res;
    $resourcelist = $resourcetemp;
    ?>

  <table border="1" cellspacing="0" cellpadding="4" bgcolor="<? echo login_form_color; ?>">
    <tr align="center"><td>
        <?
            if (!file_exists(header_image)) {
                echo "<b><font size=\"+1\" color=\"#000099\">Online Resource Scheduler</font></b>&nbsp;&nbsp;\n";
            }
            if (exit_URL != "") echo '<a href="' . exit_URL . '">';
            if (file_exists(header_image)) {
                echo '<img src="' . header_image . '" border=0>';
            }
            if (exit_URL != "") echo '<font color="#990000" size="-1"><em>Home</em></font></a>'; ?>

   </td></tr>
    <?
        if (enable_login) {
            if ($_POST['forgottenpassword']==1) {
                //E-mail login link to user who forgot their password
                echo "<tr><td><font color=\"#990000\">";
                forgottenpassword($_POST);
                echo "</font></td></tr>";
            }

            if (in_array("pwsupplied", array_keys ($_POST)) & $_POST['username'] != "") {
                echo "<tr><td><font color=\"#990000\"><b>That Username / Password combination is not valid</b></font>";
                echo "<br>Forgotten your password? <a href=\"" . schedule_prog . "?forgottenpassword=1&username=". $_POST['username'] . "\"><em>Yes! please e-mail me a link to log in</em></a>.";
                echo "</td></tr>";
            }
                ?>
            <tr align="right">
                      <td>Username:
                        <input type="text" name="username"
                        <?
                        if (in_array("username", array_keys ($_POST)))
                                echo "value=\"" . $_POST['username'] . "\"";
                        ?>
                        >
                        &nbsp;&nbsp;&nbsp;Password:
                        <input type="password" name="password">
                        <input type="submit" name="Submit" value="Submit">
                        <a target="blank" href = "help/index.html" ><font color="#000066" size="-1"><em>Help!</em></font></a>
                      </td></tr>
        <? } else {
                echo "<tr><td><h3><center>Sorry, Scheduling is currently down for maintenance. Please check back later.</center></h3></td></tr>";
                }
                ?>

                   </table>
                   <input type="hidden" name="pwsupplied" value="1">
                <?

        logaction();  //start by checking for daily actions


}
if ($thisuserid)        { //username/password is valid

        if (in_array("phpinfo", array_keys ($_POST))) phpinfo();

        if (in_array("UpdateView", array_keys ($_POST)))        //updateview gets translated for particular "modes"
                if (in_array($_POST['mode'], array("As User", "edit", "info", "No-Show"))) $_POST['submit']=$_POST['mode'];

        //Translate "alternate text" for special submit requests
        if (checkout_desc != "" && $_POST['submit']==checkout_desc)          $_POST['submit'] = 'Check Out';
        if (return_desc != "" && $_POST['submit']==return_desc)              $_POST['submit'] = 'Return Resource';
        if (return_desc != "" && $_POST['submit']=="Confirm " . return_desc) $_POST['submit'] = 'Confirm Resource Return';

        //Test for "isconfirmed" information ONLY set by java-script confirm dialog
        if ($_POST['isconfirmed']!="") $_POST['submit'] = $_POST['isconfirmed'];

        //Provide warning if account will expire soon or if it has already expired
        $userexpiredate = $users[$thisuserid]['expire'];
        if (strcmp($userexpiredate, "Expired") == 0) {
             $warning[] = errormsg_acntexpired;
        }elseif (strcmp($userexpiredate, "") != 0) {
             $timetoexpire = strtotime($userexpiredate) - time();
             if (($timetoexpire < warn_account_expire) && ($timetoexpire > 0)) $warning[] = "Warning: Your account will expire in ".besttime($timetoexpire+24*60*60).".  ".warnmsg_acntexpire;
        }

        //Provide warning if account is inactive
        $lastactivity = $users[$thisuserid]['lastactivity'];
        if (stristr($lastactivity, "inactive") != FALSE) $warning[] = errormsg_acntinactive;

        //Provide warning if account is restricted
        if ($users[$thisuserid]['restricted']) {
           $warning[] = errormsg_acntrestricted;
        }

        //Copy form date/time into starttime and endtime
        if (in_array("sdate", array_keys ($_POST)) & in_array("stime", array_keys ($_POST)))  {
            if (!empty($_POST['sdate']) && $_POST['stime'] == "") {
                $_POST['stime'] = day_start;
            }
            if ($_POST['stime'] != "") {
                $validdate = validatedate($_POST['sdate']);
                if ($validdate == -1) {
                    $warning[] = "Bad Start Date";
                    unset($_POST['starttime']);
                }else {
                   $_POST['sdate'] = date(dateformat . "/y",$validdate);
                   $d = @strtotime(date("m/d/y",$validdate) . ' ' . $_POST['stime']);
                   if ($d>0 & (($d >= zulutime()+timezone + allow_signup_offset) | $administrator)) {
                       $d = round($d/60/60/signup_interval)*60*60*signup_interval;
                       $_POST['starttime'] = $d;
                   } else {
                       if ($d<=0) $warning[] = "Bad Start Date";
                       unset($_POST['starttime']);
                   }
                }
            }
        }

        if (in_array("edate", array_keys ($_POST)) & in_array("etime", array_keys ($_POST)))  {
            if (!empty($_POST['edate']) && $_POST['etime'] == "") {
                $_POST['etime'] = min(23,day_end);
            }
            if ($_POST['etime'] != "") {
                if ($_POST['edate'] == "" & $_POST['sdate'] != "") $_POST['edate'] =$_POST['sdate'];
                    //use start date if no end date(but we DO have an end time!)
                $validdate = validatedate($_POST['edate']);
                if ($validdate == -1) {
                    $warning[] = "Bad End Date";
                    unset($_POST['endtime']);
                }else {
                    $_POST['edate'] = date(dateformat . "/y",$validdate);
                    $d = @strtotime(date("m/d/y",$validdate) . ' ' . $_POST['etime']);
                    if ($d>0 & (($d >= zulutime()+timezone + allow_signup_offset) | $administrator) ) {
                        $d = round($d/60/60/signup_interval)*60*60*signup_interval;
                        $_POST['endtime'] = $d;
                    } else {
                        if ($d<=0) $warning[] = "Bad End Date";
                        unset($_POST['endtime']);
                    }
                }
            }
        }

        //interpret any submitted checkout and checkin dates and times
        if (in_array("outdate", array_keys ($_POST)) & in_array("outtime", array_keys ($_POST)))  {
            if ($_POST['outtime'] != "") {
                $validdate = validatedate($_POST['outdate']);
                if ($validdate == -1) {
                    $warning[] = "Bad Start Date";
                    unset($_POST['checkout']);
                }else {
                   $_POST['outdate'] = date(dateformat . "/y",$validdate);
                   $d = @strtotime(date("m/d/y",$validdate) . ' ' . $_POST['outtime']);
                   if ($d>0 & (allow_checkout_edit | $administrator)) {
                       $_POST['checkout'] = $d;
                   } else {
                       if ($d<=0) $warning[] = "Bad Start Date";
                       unset($_POST['checkout']);
                   }
                }
            }
        }

        if (in_array("indate", array_keys ($_POST)) & in_array("intime", array_keys ($_POST)))  {
            if ($_POST['intime'] != "") {
                if ($_POST['indate'] == "" & $_POST['outdate'] != "") $_POST['indate'] =$_POST['outdate'];
                    //use start date if no end date(but we DO have an end time!)
                $validdate = validatedate($_POST['indate']);
                if ($validdate == -1) {
                    $warning[] = "Bad End Date";
                    unset($_POST['checkin']);
                }else {
                    $_POST['indate'] = date(dateformat . "/y",$validdate);
                    $d = @strtotime(date("m/d/y",$validdate) . ' ' . $_POST['intime']);
                    if ($d>0 & (allow_checkout_edit | $administrator)) {
                        $_POST['checkin'] = $d;
                    } else {
                        if ($d<=0) $warning[] = "Bad End Date";
                        unset($_POST['checkin']);
                    }
                }
            }
        }

        //Clear date&time manual inputs (we just used them above and are done with them now)
        if (in_array("sdate", array_keys ($_POST)) & in_array("stime", array_keys ($_POST)))  {
            unset($_POST['sdate']);
            unset($_POST['stime']);
        }
        if (in_array("edate", array_keys ($_POST)) & in_array("etime", array_keys ($_POST)))  {
            unset($_POST['edate']);
            unset($_POST['etime']);
        }

        //clear from inputs if empty
        if ($_POST['starttime'] == "" | $_POST['starttime'] <= 0) unset($_POST['starttime']);
        if ($_POST['endtime']   == "" | $_POST['endtime']   <= 0) unset($_POST['endtime']);

        switch ($_POST['submit']) {
        case "OK" :                 //same as clear
        case "Cancel" :         //same as clear
        case "Clear" :
                //if "clear" selected and starttime and endtime wern't there, then clear tabledate (resets back to today)
                if (!in_array("starttime", array_keys ($_POST)) & !in_array("endtime", array_keys ($_POST))
                     & $_POST['submit']=="Clear") {
                        unset($_POST['tabledate']);
                        unset($_POST['tabledatetext']);
                }
                $_POST = cleanup($_POST);
        }

        //check for "early morning" times and convert to valid time
        if (in_array("starttime", array_keys ($_POST))) {
                $startinfo = getdate($_POST['starttime']);
        }
        if (in_array("endtime", array_keys ($_POST)))  {
                $endinfo = getdate($_POST['endtime']);
        }

        //check for bad start/end times
        if (in_array("starttime", array_keys ($_POST)) &
            in_array("endtime", array_keys ($_POST))) {
                //check for inverted start/end times and swap if so
                if ($_POST['starttime'] > $_POST['endtime']) {
                        $temp = $_POST['starttime'];
                        $_POST['starttime'] = $_POST['endtime'];
                          $_POST['endtime'] = $temp;
                }
        }

        //No calendar authority? No priority setting!
        if (!$calendarauth) unset($_POST['priority']);

        //check for REALLY SHORT signups
        if ($_POST['starttime'] != "" & !in_array($_POST['submit'],array("info","Delete","Confirm Delete","No-Show","Return Resource","Confirm No-Show")))
                //check for REALLY SHORT ILLEGAL signups
                if ($_POST['endtime'] != "" & ($_POST['endtime'] - $_POST['starttime']) < 60*60*min_signup_time) {
                        $warning[] = "Signups shorter than " . besttime(min_signup_time*60*60) . " are not permitted. Give new end date.";
                        echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                        unset($_POST['endtime']);
                }

        //check for REALLY LONG signups
        if ($_POST['starttime'] != "" & !in_array($_POST['submit'],array("info","Delete","Confirm Delete","No-Show","Return Resource","Confirm No-Show"))) {
                //check for REALLY LONG ILLEGAL signups
                if ($_POST['endtime'] != "" & ($_POST['endtime'] - $_POST['starttime']) > min(180*60*60*24,longest_signup)) {
                        $warning[] = "Signups longer than " . besttime(min(180*60*60*24,longest_signup)) . " are not permitted. Give new end date.";
                        echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                        unset($_POST['endtime']);
                }
                //check for REALLY LONG signups (still legal)
                elseif ($_POST['endtime'] != "" & ($_POST['endtime'] - $_POST['starttime']) > warn_long_signup)
                        $warning[] = "Signup longer than " . besttime(warn_long_signup) . " - please verify dates.";

                //check for start date REALLY FAR OFF - Dissallow
                if ((disallow_future_signup != 0)&&($_POST['starttime'] > zulutime()+timezone+disallow_future_signup) & !$administrator){
                        $warning[] = "Signup Prohibited: Start time further off than " . besttime(disallow_future_signup) . " - please verify dates.";
                        unset($_POST['starttime']);
                        unset($_POST['endtime']);
                }else {
                       //check for start date REALLY FAR OFF - Give Warning
                       if ($_POST['starttime'] > zulutime()+timezone+warn_future_signup)
                            $warning[] = "Start time further off than " . besttime(warn_future_signup) . " - please verify dates.";
                }

        }

        //sort out selected resources into double-bookable and non-double-bookable
        $selectedndb = array();
        $selecteddb = array();
        foreach ($selectedres as $oneresource) {
            if ($resourcelist[$oneresource]['doublebook']==1) {
                $selecteddb[] = $oneresource;
            } else {
                $selectedndb[] = $oneresource;
            }
        }
        $selectedres = array_merge($selectedndb,$selecteddb);  //put resources into new order (non-double-bookable first)

        //both start and end times? 
        if (in_array("starttime", array_keys ($_POST)) & in_array("endtime", array_keys ($_POST))) {

            if ($_POST['avonly']==1) {
                // Limit to available resources (if requested)
                $allpriorities = array();
                $use = array();
                foreach ($resourcelist as $res=>$oneresource) {
                    $priority = findpriority($_POST['starttime'],$_POST['endtime'],$oneresource['resource']);
                    if (hard_available)
                        $require = 1;  //a "free resource" is one where a primary signup is available
                    else
                        $require = num_signups;    //a "free resource" is one where ANY signups are available

                    if ($priority>$require && !in_array($res,$selectedres)) {
                        $resourcelist[$res]['donotshow'] = True;
                    }
                }
            }

            if (in_array($_POST['submit'],array("Submit","As User",""))) {
                // calc. priority for this signup if signup times were "submitted"
                if (!in_array("All",$selectedres) && (count($selectedndb)==1 || (count($selectedndb)==0 && count($selectedres)>0)) ) {
                    if ($calendarauth & $_POST['priority'] != "") {
                        $priority = findpriority($_POST['starttime'],$_POST['endtime'],$selectedres[0],$_POST['priority'],true);
                    } else {
                        $priority = findpriority($_POST['starttime'],$_POST['endtime'],$selectedres[0]);
                    }
                    if (($priority == 0 | $priority > num_signups) & ($_POST['submit'] == "" | $_POST['submit'] == "Submit" ))
                        $warning[] = "No alternate signups available- Change times or resource.";
                } else $warning[] = "Please choose only one resource";
                
            } else $priority = 0;                


        } else $priority = 0;  //not enough info to calc. priority (or some other reason we don't want to)

        //if we have start and end and the user can't signup as themselves (not authorized), make the submit
        // be "as user" since that is what they'll have to do anyway!
        if (in_array("starttime", array_keys ($_POST)) & in_array("endtime", array_keys ($_POST)) &
                (!in_array("submit", array_keys ($_POST)) | $_POST['submit'] == "Submit" | $_POST['submit'] == "")
                 & !$signuppermitted) $_POST['submit'] = "As User";

        //set some display flags, modified by various submit modes
        $no_edit   = False;                //true = do not allow user to edit values
        $show_name = False;                //true = show signup name

        //special submit requests
        switch ($_POST['submit']) {

        case "edit" :
            if (($_POST['usn']!=$thisuserid) && !$calendarauth && !edit_other_users) {
                    //not permitted to change other people's signups? just give info instead of editing
                    $_POST['submit'] = "info";
            } else {
                    if ($_POST['usn']>0) {
                        $signup = getusersignups ($_POST['usn'],false);
                        $signup = $signup[$_POST['signup']];
                        $show_name = True;
                    } elseif ($_POST['usn']==0) {
                        if (false) { //disabled
                            $temp = explode(' ',$_POST['signup']);
                            $signup['starttime'] = $temp[0];
                            $signup['endtime']   = $temp[0] + $temp[1]*60*60;
                            $signup['priority']  = $temp[2];
                            $signup['status']    = "";
                        }
                    }
                    if (count($signup) > 0) {
                            $_POST['starttime']  = $signup['starttime'];
                            $_POST['endtime']    = $signup['endtime'];
                            $_POST['comment']    = $signup['comment'];
                            $_POST['status']     = $signup['status'];
                            $_POST['checkout']   = $signup['checkout'];
                            $_POST['checkin']    = $signup['checkin'];
                            $_POST['signupname'] = $_POST['usn'];
                            $warning[] = "Editing Previous Signup";
                            if (manage_status && $_POST['status'] == "Pending Approval") { //********
                                $warning[] = 'This signup is awaiting approval';
                            }
                    } else {
                            $signup = "";
                            $warning[] = "This recurring signup can only be edited on the recurring signup page.";
                            echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                            $_POST['submit'] = "";
                            $_POST['signup'] = Null;
                            $_POST['usn']    = Null;
                            $show_name = False;
                    }
                    break;
            }

        case "No-Show" : //first time through, this is like "info" except they're given "confirm No-Show" option
        case "Delete"  : //first time through, this is like "info" except they're given "confirm delete" option
        case "Return Resource" :  //yea.. just like those except "Confirm Resource Return"
        case "info" :  //almost same as edit (no "update" or "delete" buttons provided)

            if ($_POST['submit']=="No-Show") $warning[] = "Notice: This action will be logged";

            $signup = getusersignups ($_POST['usn'],false);
            $signup = $signup[$_POST['signup']];

            if (count($signup) > 0) {
                $_POST['starttime']  = $signup['starttime'];
                $_POST['endtime']    = $signup['endtime'];
                $_POST['comment']    = $signup['comment'];
                $_POST['status']     = $signup['status'];
                $_POST['checkout']   = $signup['checkout'];
                $_POST['checkin']    = $signup['checkin'];
                $_POST['signupname'] = $_POST['usn'];

                if (manage_status && $_POST['status'] == "Pending Approval") { //********
                    $warning[] = 'This signup is awaiting approval';
                }

                $prioritycheck = (getsignuppriority ($_POST['usn'], $_POST['signup'])==1);
                if ($_POST['submit']=="Delete" && $prioritycheck && ($_POST['starttime'] - (zulutime()+timezone) <= warn_late_delete) && !$administrator)
                     $warning[] = "WARNING: You are deleting this resource less than " . besttime(warn_late_delete) . " in advance. <br>" . warnmsg_late_delete . "<br>Please enter your reason for cancellation below.";

                $no_edit = True;
                $show_name = True;

            } else {
                $signup = "";
                $warning[] = "Sorry, this signup can not be viewed or edited.";
                $_POST['submit'] = "";
            }
            break;

        case "Confirm Delete" :
            if ($_POST['usn']!=$thisuserid && !$calendarauth && !edit_other_users) {
                $warning[] = "You are not authorized to modify other people's signups";
                echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
            } else {
                $thissignup = getusersignups($_POST['usn'],false);
                $thissignup = $thissignup[$_POST['signup']];

                $prioritycheck = (getsignuppriority ($_POST['usn'], $_POST['signup'])==1);
                //check if deleting a reservation too close to the reservation start time, send email to Warning Monitor

                if ($prioritycheck && ($thissignup['starttime'] - (zulutime()+timezone) < warn_late_delete) && !$administrator){
                    $message = "WARNING: Resource deleted within ".besttime(warn_late_delete)." \n\n";
                    $message .=  $users[$_POST['usn']]['firstname'] . " " . $users[$_POST['usn']]['lastname']." deleted the following reservation ". besttime(($thissignup['starttime'] - (zulutime()+timezone)))." before its start time:\n";
                    $message .=  "Resource = ".$resourcelist[$thissignup['resource']]['long']."\n";
                    $message .=  "Start time = ".date(timeformat,$thissignup['starttime'])."\n";
                    $message .=  "End time   = ".date(timeformat,$thissignup['endtime'])."\n";
                    $message .=  "Comment = ".$thissignup['comment']."\n\n";
                    $message .= "Reason for Cancellation: ".$_POST['ReasonCancel']."\n";

                    $addheaders = "From: " . email_from . "\n";
                    $to      = warning_monitor;
                    $subject = "WARNING - Late Resource Cancellation: ".date(dateformat . "/Y",zulutime()+timezone);

                    sendemail ($to, $subject, $message, $addheaders);
                }


                foreach ($thissignup as $key=>$data) $_POST[$key] = $data;
                logaction('deleted signup for',$thisuserid,$_POST);
                emailmonitors('delete',$_POST);
                if ($_POST['usn'] != $thisuserid) message("superdelete",$_POST);
                promote ( deletesignup ($_POST) );
            }
            $_POST = cleanup($_POST);
            $priority = 0;
            break;

        case "Confirm No-Show" :
            //new end time is NOW
            $thissignup = getusersignups($_POST['usn']);
            $thissignup = $thissignup[$_POST['signup']];
            if (count($thissignup)>0) foreach ($thissignup as $key=>$data) $_POST[$key] = $data;

            logaction('no-showed',$thisuserid,$_POST);
            emailmonitors('No-show',$_POST);
            if ($_POST['usn'] != $thisuserid) message("noshow",$_POST);
            $warning[] = "No-Showed " . $users[$_POST['usn']]['firstname'] . " " . $users[$_POST['usn']]['lastname'];

            promote ( deletesignup ($_POST) );

            $_POST = cleanup($_POST);
            $priority = 0;
            break;

        case "Approve" :   //********
            //indicate that the given signup user has been checked out
            $_POST['status'] = "";
            setsignupstatus($_POST['usn'],$_POST['signup'],$_POST['status']);
            
            //get new signup info
            $signup = getusersignups ($_POST['usn'],false);
            $_POST = array_merge($_POST,$signup[$_POST['signup']]);
            $show_name = True;

            if (manage_status) $warning[] = "Signup status is now 'approved'.";            
            logaction($_POST['submit'],$thisuserid,$_POST);
            $_POST = cleanup($_POST);
            $priority = 0;
            $show_name = false;
            break;

        case "Confirm Resource Return" :
        case "Check Out" :
            if ($_POST['usn']!=$thisuserid && !$calendarauth && !edit_other_users){
                $warning[] = "You are not authorized to modify other people's signups";
                echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                $_POST = cleanup($_POST);
                $priority = 0;
                break;
            } else {
                switch ($_POST['submit']) {
                    case "Check Out" :
                        //indicate that the given signup user has been checked out
                        $_POST['status'] = 1;
                        setsignupstatus($_POST['usn'],$_POST['signup'],$_POST['status']);
                        
                        //get new signup info
                        $signup = getusersignups ($_POST['usn'],false);
                        $_POST = array_merge($_POST,$signup[$_POST['signup']]);
                        $show_name = True;

                        if (manage_status) $warning[] = "Signup status is now 'started'.";
                        break;
                    case "Confirm Resource Return" :
                        //indicate that the given signup has been returned
                        $_POST['status'] = 0;
                        setsignupstatus($_POST['usn'],$_POST['signup'],$_POST['status']);

                        //get new signup info
                        $signup         = getusersignups ($_POST['usn'],false);
                        $_POST = array_merge($_POST,$signup[$_POST['signup']]);
                        $show_name      = True;

                        if (manage_status) $warning[] = "Signup status is now 'done'.";
                        break;
                }
                logaction($_POST['submit'],$thisuserid,$_POST);

                if ($_POST['submit']=="Confirm Resource Return" && (!manage_status || hard_return)) { 
                    //modify endtime on return
                    $d = zulutime()+timezone+1;
                    $_POST['endtime'] = floor($d/60/60/signup_interval)*60*60*signup_interval;
                    if ($_POST['endtime'] <= $_POST['starttime']) {
                        $warning[] = "Unable to end before signup has started.";                    
                        $_POST['submit'] = "edit";
                        $priority = 0;
                        $show_name = False;
                        break;
                    } else {
                        //make sure end time is at least one signup block long
                        $_POST['endtime'] = max($_POST['endtime'],($_POST['starttime']+signup_interval*60*60));
                        $_POST['submit'] = "update";    //falls through to update below to save time
                        $show_name = False;
                    }
                } else {
                    if (manage_status) {
                        //Don't modify endtime just allow user edit
                        $warning[] = "Adjust 'effective times' if necessary.";
                        $_POST['submit'] = "edit";
                    }
                    $priority = 0;
                    break;
                }
            }

        case "Update" :
            if (count($selectedres) > 1 | in_array("All",$selectedres)) {
                    $warning[] = "Choose only one resource";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $_POST['submit']="edit";
                    $priority = 0;
                    break;
            }
            if ($_POST['usn'] == "") $_POST['usn'] = $thisuserid;

            if (checkpermissions($users[$thisuserid]['resourceblock'],"1",$_POST['resource'])
                     & !$calendarauth){
                    $warning[] = "You are not authorized to sign up for this resource";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';

            } elseif (checkpermissions($users[$_POST['usn']]['resourceblock'],"1",$_POST['resource'])
                     & !$calendarauth){
                    $warning[] = "This user is not authorized to sign up for this resource";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';

            } elseif ($_POST['usn']!=$thisuserid && !$calendarauth && !edit_other_users){
                    $warning[] = "You are not authorized to modify other people's signups";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';

            } else {
                    $newusn = $_POST['signupname'];
                    if ($newusn == "") {
                            $warning[] = "Please select a user for this signup.";
                            echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                            $_POST['submit']="edit";
                            $priority = 0;
                            break;
                    } elseif (!checksec($users[$newusn]['sec'],1)) {        //check for user not allowed to signup
                            if ($newusn == $thisuserid)
                                $warning[] = "You are not allowed to sign-up for any resource";
                            else
                                $warning[] = "Specified user not allowed to sign-up for any resource";
                            echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                            $_POST['submit']="edit";
                            $priority = 0;
                            break;
                    }

                    $oldsignup = getusersignups($_POST['usn'],false);
                    $oldsignup = $oldsignup[$_POST['signup']];
                    $oldsignup['priority'] = getsignuppriority ($_POST['usn'],$_POST['signup']);

                    //use original starttime if old starttime was in past (no matter what the user entered)
                    // OR starttime is empty (often because it was cleared because user entered time in the past)
                    if ($_POST['starttime'] == "" | (!allow_poststart_mod & $oldsignup['starttime'] < zulutime()+timezone + allow_signup_offset & !$administrator)) {
                            //if still in the future and not checked out yet, give warning
                            if ((!manage_status || $_POST['status']=="") && $oldsignup['starttime'] >= zulutime()+timezone) $warning[] = "Invalid start time - using original";
                            $_POST['starttime'] = $oldsignup['starttime'];
                    }

                    // endtime is empty (often because it was cleared because user entered time in the past)
                    if ($_POST['endtime'] == "") {
                            $_POST['endtime'] = $oldsignup['endtime'];
                            $warning[] = "Invalid end time - using original";
                            // Disabled --// $priority = -1;                //indicates break-out as edit below
                    }

                    if ($priority==-1) {  //start or end time was invalid
                            $_POST['submit'] = "edit";
                            $priority = 0;
                            break;
                    }

                    //verify that we haven't snuck "underneith" someone else (look for NEW overlaps if priority isn't forced by administrator)
                    if ($_POST['priority'] == "") {
                            $oldoverlaps = findoverlaps($oldsignup, true);
                            $_POST['priority'] = $oldsignup['priority'];        //temporary.. we'll clear in a moment
                            $newoverlaps  = findoverlaps($_POST, true);        //findoverlaps wants SOMETHING for priority so we say "based on the old priority"
                            $_POST['priority'] = "";                        //told you so (btw, the only way we're in here is if this was blank anyway!)

                            $overlaperr = false;        //assume the best
                            if (count($newoverlaps) > count($oldoverlaps)) $overlaperr = true;
                            elseif (count($oldoverlaps) > 0) {
                                    foreach($oldoverlaps as $oitem)
                                            if (count($newoverlaps) > 0) foreach ($newoverlaps as $key=>$nitem)
                                                    // delete all items which match
                                                    if ($oitem['usn'] == $nitem['usn'] & $oitem['signup'] == $nitem['signup']) {
                                                            unset($newoverlaps[$key]);
                                                            break;
                                                    }
                                    if (count($newoverlap)>0) $overlaperr = true;        //something in new which wasn't in old? error!
                            }
                            if ($overlaperr) {
                                    $warning[] = "Can't extend under other signups - Add as new";
                                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                                    $_POST['submit'] = "edit";
                                    $priority = 0;
                                    break;
                            }
                    }

                    $result = update($_POST,$newusn,$_POST['priority']);

                    if (!$result['success']) {
                            $warning[] = "Signup Rejected for the following reason(s):";
                            $warning   = array_merge($warning,$result['warning']);
                            echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                            $_POST['submit'] = "edit";
                            $_POST['signup'] = $result['signup'];                //didn't get added, but assigned new signup
                            $priority = 0;
                            $show_name = True;
                            if ($_POST['usn']!=$thisuserid && !$calendarauth && !edit_other_users) {
                                $no_edit = True;
                            }
                            $_POST['signupname'] = $_POST['usn'];
                            break;
                    } else {

                            //if there is a warning for this item, give it now if warningpopup is selected
                            if ($resourcelist[$_POST['resource']]['popupwarning'] && $resourcelist[$_POST['resource']]['warnings']!="") {
                                echo '<script language="JavaScript">alert("';
                                echo $resourcelist[$_POST['resource']]['long'] . ' Warning:\n';
                                echo stripcslashes($resourcelist[$_POST['resource']]['warnings']);;
                                echo '")</script>';
                            }

                            //if we're allowing a non administrator to move an already-past start time to a forward time, leave a "ghost trail" behind
                            if (allow_poststart_mod & $_POST['starttime'] > zulutime()+timezone &  $oldsignup['starttime'] < zulutime()+timezone + allow_signup_offset & !$administrator) {
                                    $ghost = $oldsignup;
                                    $ghost['usn'] = $_POST['usn'];
                                    $ghost['endtime'] = floor((zulutime()+timezone)/(signup_interval*60*60))*signup_interval*60*60;
                                    addsignup($ghost);
                            }

                            $_POST['newsignup']   = $result['signup'];
                            $_POST['priority']    = getsignuppriority ($newusn,$result['signup']);

                            if ($oldsignup['resource']!=$_POST['resource'] | $newusn!=$_POST['usn']) {
                                //moved to new resource, handle logging differently
                                $temp = array_merge($_POST,$oldsignup);
                                emailmonitors('delete',$temp);  //send delete message to old resource monitors
                                logaction('delete',$thisuserid,$temp);

                                $_POST['usn'] = $newusn;
                                emailmonitors('signup',$_POST); //send signup message to NEW resource monitors                                    
                                logaction('signup',$thisuserid,$_POST);
                            } else {
                                //change within same resource, log normally
                                logaction('updated',$thisuserid,$_POST);
                                emailmonitors('update',$_POST);
                            }

                            //Add message to users
                            if ($newusn != $thisuserid) $warning[] = "Signed up as user: " . $users[$newusn]['firstname'] . " " . $users[$newusn]['lastname'];
                    }
            }
            $_POST = cleanup($_POST);
            $priority = 0;
            break;

        case "Special" :                //acts like As User but forces priority (only calendar authorized users can use)
            if (!$calendarauth)
                $_POST['submit'] = "Submit";        //not authorized? don't allow
            else {
                $priority = 1;                                //fake priority 1
                $_POST['priority'] = 1;
                $show_name = True;
            }
            break;
        case "As User" :                //first call to as user
            $_POST['usn'] = "";
            if (allow_signup_as_others  || $administrator || (calendarauth_as_others && $calendarauth)) {
                //calendar authorized or admin
                $show_name = True;
            } else {
                $show_name = False;
                $_POST['submit'] = "Submit";        //not authorized? don't allow
            }                
            break;

        case "Sign-up As User" :                //subsequent call to as user
            if ($_POST['usn'] == "") {
                $_POST['usn'] = $_POST['signupname'];
                if ($_POST['usn'] == "")  {
                    $warning[] = "Please select a user for this signup.";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $_POST['submit'] = "As User";
                    $show_name = True;
                    break;
                } elseif (!checksec($users[$_POST['usn']]['sec'],1) ) {        //check for user not allowed to signup
                    if ($_POST['usn'] == $thisuserid)
                        $warning[] = "You are not allowed to sign-up for any resource";
                    else
                        $warning[] = "Speceified user not allowed to sign-up for any resource";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $_POST['usn'] = "";
                    $_POST['submit'] = "As User";
                    $show_name = True;
                    break;
                }
            }

        case "Sign-up" :
            if (!in_array("starttime", array_keys ($_POST)) | !in_array("endtime", array_keys ($_POST))) {
                $_POST['submit'] = "";
                break;
            }
            if (count($selectedndb) > 1 | in_array("All",$selectedres)) {
                $warning[] = "Choose only one resource";
                echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                $_POST['submit'] = "";
                break;
            }

            //First, verify that we can actually do all these signups
            $failure = false;
            foreach ($selectedres as $oneresource) {
            
                $_POST['resource'] = $oneresource;
                $priority = findpriority($_POST['starttime'],$_POST['endtime'],$oneresource);

                if ($_POST['usn'] == "") $_POST['usn'] = $thisuserid;

                if (checkpermissions($users[$thisuserid]['resourceblock'],"1",$_POST['resource'])
                     & !$calendarauth){
                    $warning[] = "You are not authorized to sign up for " . $resourcelist[$_POST['resource']]['long'];
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $failure = true;
                } elseif (checkpermissions($users[$_POST['usn']]['resourceblock'],"1",$_POST['resource'])
                     & !$calendarauth){
                    $warning[] = "This user is not authorized to sign up for " . $resourcelist[$_POST['resource']]['long'];
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $failure = true;
                } elseif (!allow_signup_as_others & $_POST['usn'] != $thisuserid & !$calendarauth){
                    unset($_POST['usn']);
                    unset($_POST['signup']);
                    unset($_POST['submit']);
                    $warning[] = "You are not authorized to sign up for other people";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    $failure = true;
                }
                if ($failure) break; //stop now if anything went wrong
            }
            if ($failure) break; //stop now if anything went wrong

            $_POST['resource'] = implode('+',$selectedres);
            foreach (array_merge($selecteddb,$selectedndb) as $oneresource) {
        
                $_POST['resource'] = $oneresource;
                $priority = findpriority($_POST['starttime'],$_POST['endtime'],$oneresource);

                if ($_POST['usn'] == "") $_POST['usn'] = $thisuserid;

                if ($priority > num_signups & ($_POST['priority']!="" | $calendarauth))        {        //check for priority > num_signups
                    if (!demote($_POST)) {
                        $warning[] = "Insert signup failed";
                        echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                        $failure = true;
                        break;
                    }
                    $priority = $_POST['priority'];
                }
                if (!checksec($users[$_POST['usn']]['sec'],1) )        {        //check for user not allowed to signup
                    if ($_POST['usn'] == $thisuserid)
                        $warning[] = "You are not allowed to sign-up for any resource";
                    else
                        $warning[] = "Specified user not allowed to sign-up for any resource";
                    echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                    unset($_POST['usn']);
                    unset($_POST['signupname']);
                    $failure = true;
                    break;
                }

                //If the account is expired or restricted, determine if signup is authorized
                If ((strcmp($users[$_POST['usn']]['expire'],"Expired") == 0) || $users[$_POST['usn']]['restricted']) {

                    //If hard_account_expiration is set, prohibit future signup attempts
                    If (hard_account_expiration) {
                         unset($_POST['usn']);
                         unset($_POST['signup']);
                         unset($_POST['submit']);
                         $warning[] = "Your account is expired or restricted. Please contact an administrator for more help.";
                         echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                        $failure = true;
                         break;
                    //If hard_account_expiration is unset, only delete solo signups
                    } else {
                        //Check if this signup's resource is double-bookable
                        if (!$resourcelist[$_POST['resource']]['doublebook']) {
                            //Resource is not double-bookable, therefore check if there
                            //   is another signup for same time period with a double-bookable resource
                            $signupOK = FALSE;
                            $usersignups = getusersignups($_POST['usn']);
                            foreach ($usersignups as $signupcheck){
                                //Find other signups with double-bookable resources
                                if ($resourcelist[$signupcheck['resource']]['doublebook']) {
                                    if (($_POST['starttime'] >= $signupcheck['starttime'])
                                    && ($_POST['endtime'] <= $signupcheck['endtime'])) {
                                         //Matching double-bookable signup found.  Leave signup alone.
                                         $signupOK = TRUE;
                                         break;
                                    }
                                }
                            }
                            //If no matching double-bookable signup found, then delete signup
                            if ($signupOK == FALSE) {
                                 $warning[] = ($users[$_POST['usn']]['restricted'] ? errormsg_acntrestricted : errormsg_acntexpired);
                                 echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                                 unset($_POST['usn']);
                                 unset($_POST['signup']);
                                 unset($_POST['submit']);
                                $failure = true;
                                 break;
                            }
                        }
                    }
                }

                $_POST['priority'] = $priority;
                if (!$calendarauth && manage_status && require_approval) $_POST['status'] = "Pending Approval";  //********
                $signup = addsignup ($_POST);

                if (!$calendarauth & hard_enforce_rules) {
                    $ruletest = validatesignups($_POST['usn'],false,$signup);
                    if (count($ruletest) > 0) {
                        $warning[] = "Reservation rejected: This Sign-up causes a rule violation (or others already exist).";
                        $warning   = array_merge($warning,$ruletest);
                        echo '<script language="JavaScript">alert("' . $warning[count($warning)-1] . '")</script>';
                        $_POST['submit'] = "";
                        deletesignup (array('usn'=>$_POST['usn'],'signup'=>$signup));
                        $failure = true;
                        break;
                    }
                }

                $_POST['signup'] = $signup;
                message("primaries",$_POST);
                emailmonitors('signup',$_POST);

                //if there is a warning for this item, give it now if warningpopup is selected
                if ($resourcelist[$_POST['resource']]['popupwarning'] && $resourcelist[$_POST['resource']]['warnings']!="") {
                    echo '<script language="JavaScript">alert("';
                    echo $resourcelist[$_POST['resource']]['long'] . ' Warning:\n';
                    echo stripcslashes($resourcelist[$_POST['resource']]['warnings']);;
                    echo '")</script>';
                }

                if ($thisuserid == $_POST['usn']) {
                    $_POST['usn'] = "";
                    $_POST['signup'] = $signup;
                    logaction('Signup',$thisuserid,$_POST);
                } else {
                    $_POST['signup'] = $signup;
                    logaction('Signup As User',$thisuserid,$_POST);
                    message("asuser",$_POST);
                    if ($_POST['usn'] != $thisuserid) $warning[] = "Signed up as user: " . $users[$_POST['usn']]['firstname'] . " " . $users[$_POST['usn']]['lastname'];
                }

                if ($failure) break; //stop if anything went wrong

                if (count($selectedres)>1) $warning[] = "Signed up for " . $resourcelist[$_POST['resource']]['long'];
            }
            $_POST['resource'] = implode('+',$selectedres);

            if ($failure) break; //stop if anything went wrong
            //all signups worked? clear info
            $_POST = cleanup($_POST);
            $priority = 0;

            //if "clear" selected and starttime and endtime wern't there, then clear tabledate (resets back to today)
            if (!in_array("starttime", array_keys ($_POST)) & !in_array("endtime", array_keys ($_POST))
                 & $_POST['submit']=="Clear") {
                    unset($_POST['tabledate']);
                    unset($_POST['tabledatetext']);
            }
            $_POST = cleanup($_POST);
            $priority = 0;
            break;
        }

// - - - - - - - - - - - - - - - - - - - - - - - - - - -
// Draw main form and actions menu

?>

    <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >
    <input type="hidden" name="mode" value="<? echo $_POST['submit']; ?>">

<? if ($no_edit | in_array($_POST['submit'],array("edit"))) { ?>
    <input type="hidden" name="usn"
      <?php echo "value=\"" . $_POST['usn'] . "\""; ?>
>    <input type="hidden" name="signup"
      <?php echo "value=\"" . $_POST['signup'] . "\""; ?>
>
<? } ?>

<table border="0" cellspacing="3" cellpadding="0" class=signuptable>
  <tr>
    <td colspan="2">
      <div id=titlerow>
        <? if ($thisuserid) {
           ?>
             User:
           <?
           echo $users[$thisuserid]['firstname'] . ' ' . $users[$thisuserid]['lastname'];
           if ($administrator) echo "<span id=titleauthorized>&nbsp;(Admin.)</div>";
           elseif ($calendarauth) echo "<span id=titleauthorized>&nbsp;(Calendar Auth.)</div>";
        } ?>
      </div>
    </td>
  </tr>
<? //- - - - - - - Actions menu - - - - - - - ?>
  <tr class=actionsmenu>
    <td colspan = 2>
      <? if ($thisuserid) { ?>
        <li id=title>Actions:</li>

        <li>
        <a href = <? echo '"' . list_signups_prog . '?UID=' . $UID . '&usn=' . $_POST['usn'] . '&submit=listuser"'; ?>  >
        User Signups</a>
        </li>

        <li>
        <a href = <? echo '"' . list_signups_prog . '?UID=' . $UID . '&resource=' . $_POST['resource'] . '&submit=listresource"'; ?>  >
        Resource Signups</a>
        </li>

        <? if (allow_memberlist_view || $administrator) { ?>
        <li>
        <a href = <? echo '"' . list_members_prog . '?UID=' . $UID . '&submit=memberlist"'; ?>  >
        Member List</a>
        </li>
        <? } ?>

        <li>
        <a href = <? echo '"' . util_prog . '?UID=' . $UID . '"'; ?>  >
        Utilities Menu</a>
        </li>

        <li>
        <a target="blank" href = "help/index.html"  >
        Help!</a>
        </li>
        
        <li id="last">
        <a href = <? echo '"' . schedule_prog . '?UID=' . $UID . '&submit=logout"'; ?>  >
        Log Out</a>
        </li>
      <? } ?>
    </td>
  </tr>
        
<? //- - - - - - - Show Warnings - - - - - - - ?>

<tr><td colspan=2>
<?
// look for signup violations
$violations = validatesignups($thisuserid,false);

// display warnings and violation notices
if (count($warning) > 0 | count($violations)>0) {
        echo "<div id=\"warning\">";
        if (count($warning) > 0) {
            echo $warning[0];
            unset($warning[0]);
            if (count($warning) > 0) foreach ($warning as $onewarning) echo "<br>" . $onewarning;
        } elseif (count($violations)>0) {
            echo 'NOTE: Your existing sign-ups break one or more rules. <a href = "' . list_signups_prog . '?UID=' . $UID . '&usn=' . $_POST['usn'] . '&submit=listuser">[Click for details]</a>';
        }
        echo "</div>";
}

?>
</td></tr>

<? //- - - - - - - Name and Priority Entries - - - - - - - ?>

 <tr><td nowrap>
 <? if ($show_name) { ?>
     <b>Name:</b>
     <?
        if (!$administrator && ($no_edit || (!$calendarauth && !allow_signup_as_others) || ($calendarauth && !calendarauth_as_others) ) ) {
                //if not authorized to edit name, display it ONLY
                echo $users[$_POST['signupname']]['firstname'] . " " . $users[$_POST['signupname']]['lastname'] . "&nbsp;&nbsp;&nbsp;&nbsp;";
                echo '<input type="hidden" name="signupname" value="' . $_POST['signupname'] . '">';
        } else {
                if ($_POST['usn'] != "") 
                    usermenu($_POST['usn'],"signupname",true,1);
                else 
                    usermenu($thisuserid,"signupname",true,1);
                echo '';
        }
        if (!$no_edit && in_array($_POST['submit'],array('As User','Special','edit')) && $calendarauth) {
                echo "&nbsp;<b>Signup ";
                if (automanage_alternates) {
                    echo "Priority";
                } else {
                    echo "Position";
                }
                echo ":</b>";
                ?>
                  <input type="text" name="priority" size="3" maxlength="2"
                    <? echo 'value="' . $_POST['priority'] . '"'; ?>
                  >
                <?     
        }
}
if (manage_status && editable_status && $show_name && !$no_edit && $calendarauth && $_POST['status']!="" && in_array($_POST['submit'],array('As User','Special','edit'))) {
    echo '<br>&nbsp;<b>Status:&nbsp;</b>';
    echo '<select name="status" size="1">';
    echo "<option value=\"\"";
    if ($_POST['status']=="") echo " selected";
    echo ">&lt;none&gt;</option>\n";
    foreach (array(1=>"Started",0=>"Done","Pending Approval"=>"Pending Approval") as $val=>$desc) { //******
        echo "<option value=\"" . $val . "\"";
        if ($_POST['status']==$val) echo " selected";
        echo ">$desc</option>\n";
    }
    echo '</select>';
} else {
    echo '<input type="hidden" name="status" value="' . $_POST['status'] . '">';
}
if ($no_edit) {
    echo "<br><b>Resource:</b></font></font>&nbsp;&nbsp;&nbsp;" . $resourcelist[end($selectedres)]['long'];
    echo '<input type="hidden" name="resource" value="' . $_POST['resource'] . '">';
}
?> </font></td>

<? //- - - - - - - Resource menu - - - - - - - 

if (!$no_edit) {
    if (count($resourcelist)<=1) {
        $oneitem = end($resourcelist);
        echo '<input type="hidden" name="resource" value="' . $oneitem['resource'] . '">';
    } else {
        ?>
        <td rowspan=6 id="resources">
        Resource:<br>
        <select name="resource[]" 
           language="JavaScript" onchange="document.SignupForm.UpdateView.click()"
           size="5" multiple>
        <?

        $numshown = 0;
        foreach (sortby($resourcelist,'order') as $oneitem) {
            if ((checkpermissions($users[$thisuserid]['resourceblock'],"0",$oneitem['resource'])
               | $calendarauth | show_blocked_resources)
               && ($oneitem['donotshow']!=True || $_POST['starttime']=="" || $_POST['endtime']=="")){
                    echo "<option value=\"" . $oneitem['resource'] . "\"";
                    if (in_array($oneitem['resource'],$selectedres)) echo " selected";
                    echo ">${oneitem['long']}</option>\n";
                    $numshown++;
            }   
        }
        if ($numshown>1) {
            if (in_array("All",$selectedres))
                echo "<option value=\"All\" selected>All</option>";
            else         
                echo "<option value=\"All\">All</option>";
        }
        echo '</select>';
    
        if (in_array("starttime", array_keys ($_POST)) & in_array("endtime", array_keys ($_POST))) {
            echo '<br><center><font size=-1><input type="checkbox" name="avonly" Value="1"';
            if ($_POST['avonly']==1) echo ' checked';
            echo ' language="JavaScript" OnClick="document.SignupForm.UpdateView.click()">List Available Only</font></center>';
        }
        ?>
        </td>
        <?
    }
}

?>

</tr>

<? //- - - - - - - Start Date/Time Entries - - - - - - - ?>

    <? //decide if we should show the check-out time
    $show_checkouttime = (manage_status && $_POST['checkout']>0 && $_POST['status']!="");
    ?>
  <tr id=daterow>
    <td>
      <div id=datecell>
        <? 
        if (!$show_checkouttime) {
            echo "Start Date: ";
        } else {
            if (checkout_desc == "") {
                echo "<span id=warning>Checkout</span> Date:";        
            } else {                            
                echo "<span id=warning>" . checkout_desc . "</span> Date:";        
            }
        }
        if ($show_checkouttime) {
            if (($calendarauth || allow_checkout_edit) && !$noedit) {
                echo '<input type="text" name="outdate" size="10"';
                if (in_array("checkout", array_keys ($_POST))) {echo ' value="' . date(dateformat . "/y",$_POST['checkout']) . '"'; }
                echo '>';
            } else {
                echo date(dateformat . "/y",$_POST['checkout']);
            }
        } elseif ($no_edit) {
            echo date(dateformat . "/y",$_POST['starttime']);
        } else {
            echo '<input type="text" name="sdate" id="sdate" size="10"';
            if (in_array("starttime", array_keys ($_POST))) { echo ' value="' . date(dateformat . "/y",$_POST['starttime']) . '"'; }
            echo '>';

            //add calendar script
            ?>
            <script type="text/javascript">
                function updateview () {
                    document.SignupForm.tabledate.value = Date.parse(document.SignupForm.tabledatetext.value)/1000;
                }
                Calendar.setup({
                    inputField     :    "sdate",          // id of the input field
                    ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                    weekNumbers    :    false,            //do not show week numbers
                    showOthers     :    true,             //show other months
                    dateStatusFunc :    ispast,           //function to turn off past dates
                });
            </script>
            <?

        }
        ?>
      </div>
      <div id=timecell>
        Time:
        <?
        if ($show_checkouttime) {
            echo '<input type="hidden" name="starttime" value="' . $_POST['starttime'] . '">';
            if (($calendarauth || allow_checkout_edit) && !$noedit) {
                echo '<input type="text" name="outtime" size="10"';
                if (in_array("checkout", array_keys ($_POST))) {echo ' value="' . date(timeformat,$_POST['checkout']) . '"'; }
                echo '>';
            } else {
                echo date(timeformat,$_POST['checkout']);
            }
        } elseif ($no_edit) {
            echo date(timeformat,$_POST['starttime']);
        } else {
            echo '<input type="text" name="stime" id="stime" size="10"';
            if (in_array("starttime", array_keys ($_POST))) { echo ' value="' . date(timeformat,$_POST['starttime']) . '"'; }
            echo '>';
        } 
        ?>
      </div>
    </td>
  </tr>

<? //- - - - - - - End Date/Time Entries - - - - - - - ?>

  <tr id=daterow>
    <td>
    <? //decide if we should show the check-in time
    $show_checkouttime = (manage_status && $_POST['checkin']>0 && "".$_POST['status']=="0");
    ?>
      <div id=datecell><?
        if (!$show_checkouttime) {
            echo "End Date: ";
        } else {
            if (return_desc == "") {
                echo "<span id=warning>Return</span> Resource Date:";        
            } else {                            
                echo "<span id=warning>" . return_desc . "</span> Date:";        
            }
        }
        if ($show_checkouttime) {
            echo '<input type="hidden" name="endtime" value="' . $_POST['endtime'] . '">';
            if (($calendarauth || allow_checkout_edit) && !$noedit) {
                echo '<input type="text" name="indate" size="10"';
                if (in_array("checkin", array_keys ($_POST))) {echo ' value="' . date(dateformat . "/y",$_POST['checkin']) . '"'; }
                echo '>';
            } else {
                echo date(dateformat . "/y",$_POST['checkin']);
            }
        } elseif ($no_edit) {
            echo date(dateformat . "/y",$_POST['endtime']);
        } else {
            echo '<input type="text" name="edate" id="edate" size="10"';
            if (in_array("endtime", array_keys ($_POST))) { echo 'value="' . date(dateformat . "/y",$_POST['endtime']) . '"'; }
            echo '>';

            //add calendar script
            ?>
            <script type="text/javascript">
                Calendar.setup({
                    inputField     :    "edate",          // id of the input field
                    ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                    weekNumbers    :    false,            //do not show week numbers
                    showOthers     :    true,             //show other months
                    dateStatusFunc :    ispast,           //function to turn off past dates
                });
            </script>
        <?
        }
        ?>
      </div>
      <div id=timecell>
        Time:
        <?
        if ($show_checkouttime) {
            if (($calendarauth || allow_checkout_edit) && !$noedit) {
                echo '<input type="text" name="intime" size="10"';
                if (in_array("checkin", array_keys ($_POST))) {echo ' value="' . date(timeformat,$_POST['checkin']) . '"'; }
                echo '>';
            } else {
                echo date(timeformat,$_POST['checkin']);
            }
        } elseif ($no_edit) {
            echo date(timeformat,$_POST['endtime']);
        } else {
            echo '<input type="text" name="etime" size="10"';
            if (in_array("endtime", array_keys ($_POST))) {echo ' value="' . date(timeformat,$_POST['endtime']) . '"'; }
            echo '>';
        }
        ?>
      </div>
    </td>
  </tr>
<? //- - - - - - - Comment Entry - - - - - - - ?>

      <tr>
        <td>
            <span id=comment><? echo comment_text . ":";?></span>
        
            <?
              if ($no_edit) {
                echo stripcslashes($_POST['comment']);
            } else {
                if (!long_comment) {
                    echo '<input type="text" name="comment" size="25" maxlength="80"';
                    echo 'value="' . stripcslashes($_POST['comment']) . '">';
                } else {
                    echo '<textarea valign="top" name="comment" cols="35" rows="4">';
                    echo stripcslashes($_POST['comment']) . '</textarea>';
                }
            }
            ?>
        </td>
      </tr>
      <?
  //- - - - - - - Contact Information - - - - - - -

  //show contact info if the user has calendar authority
  if ($_POST['usn']!=$thisuserid && $_POST['signup']!=""
      && (allow_memberlist_view | $calendarauth) && show_owner_contact){  
  
      $contactinfo = listusercontact($_POST['usn'],TRUE,FALSE);
      if ($contactinfo!="") {
        ?>
        <tr>
          <td id=contactinfo>
            Contact Info:<br>
            <? echo $contactinfo; ?>
          </td>
        </tr>
        <?
      }
  }
  
//- - - - - - - Instructions/Comments - - - - - - - 

?>
<tr>
    <td>
    <table border="0" cellspacing="3" cellpadding="0" width="100%">
    <tr>
      <td>
     <font color="#CC6600">
 <?php
          if ($priority > 0 & $priority <= num_signups) {
                  echo "Confirm Sign-up";
                  if (automanage_alternates) {
                      echo "<br>";
                      if ($priority == 1) echo "as primary";
                      if ($priority > 1) echo "as alternate " . ($priority -1);
                  }
         }

        if (!in_array("starttime", array_keys ($_POST))) {
            echo "Select Start Date/Time";
        } elseif (!in_array("endtime", array_keys ($_POST))) {
            echo "Select End Date/Time";
        } elseif ($_POST['submit'] == "Delete") {
            echo "Delete this Signup?";
        } elseif ($_POST['submit'] == "Return Resource") {
            echo "End this signup now?";
        } elseif ($_POST['submit'] == "No-Show") {
            echo "This person did not show up?";
        } else {
                //other messages here?
        }
?>
      </font> </td>
      <td align="right">
<?
//  - - - - - - - Buttons to Offer - - - - - - -

        //hidden flag to indicate that JAVASCRIPT has already confirmed whatever action they wanted
        // (this is used to prepend "Confirm " to the submit string in the next call)
        echo '<input type="hidden" name="isconfirmed" value="">';
        
        if (in_array("starttime", array_keys ($_POST)) & in_array("endtime", array_keys ($_POST))) {
                //if we have start and end times BOTH, decide what buttons to provide
                
                //Editing a previous signup?
                if ($_POST['submit'] == "edit") {
                    if (manage_status && $_POST['status'] == "" && $_POST['starttime']<zulutime()+timezone+checkout_interval) {
                        if (checkout_desc == "") {
                            echo '&nbsp;<input type="submit" name="submit" value="Check Out">';
                        } else {
                            echo '&nbsp;<input type="submit" name="submit" value="' . checkout_desc . '">';
                        }
                    }

                    if (manage_status && $calendarauth && $_POST['status'] == "Pending Approval") { //********
                        echo '&nbsp;<input type="submit" name="submit" value="Approve">';
                    }

                    if ((manage_status && $_POST['status'] == "1") || 
                        (($_POST['starttime']<zulutime()+timezone) && ($_POST['endtime']>zulutime()+timezone))) {
                        //If signup has already started (but not ended yet)
                        //Offer return resource?
                        if ((!manage_status && enable_return) || (manage_status && $_POST['status'] != "0")) {
                            if (return_desc == "") {
                                echo '&nbsp;<input type="submit" name="submit" value="Return Resource" ';
                                echo "onClick=\"if (confirm('Are you sure you want to return this resource and end the signup now?')) {SignupForm.isconfirmed.value='Confirm Resource Return'; return true}; return false\">";
                            } else {
                                echo '&nbsp;<input type="submit" name="submit" value="' . return_desc . '" ';
                                echo "onClick=\"if (confirm('Are you sure you want to end the signup now?')) {SignupForm.isconfirmed.value='Confirm Resource Return'; return true}; return false\">";
                            }
                        }
                    }


                    if ($_POST['starttime']<zulutime()+timezone && (allow_past_edit || $_POST['endtime']>zulutime()+timezone)) {
                        //If signup has already started (but not ended yet) (or we're allowing editing of past signups)
                        if (enable_noshow && !in_array($users[$usn]['firstname'] . $users[$usn]['lastname'], $do_not_noshow) && $_POST['status'] == "") {
                            echo '&nbsp;<input type="submit" name="submit" value="No-Show">';
                        }
                        
                        echo '&nbsp;<input type="submit" name="submit" value="Update">';
                        if ($administrator) {
                            echo '&nbsp;<input type="submit" name="submit" value="Delete" ';
                            echo "onClick=\"if (confirm('Are you sure you want to delete this signup?')) {SignupForm.isconfirmed.value='Confirm Delete'; return true}; return false\">";
                        }
                    } else {
                        echo '&nbsp;<input type="submit" name="submit" value="Update">';
                        if (warn_late_delete==0) {
                            echo '&nbsp;<input type="submit" name="submit" value="Delete" ';
                            echo "onClick=\"if (confirm('Are you sure you want to delete this signup?')) {SignupForm.isconfirmed.value='Confirm Delete'; return true}; return false\">";
                        } else {
                            echo '&nbsp;<input type="submit" name="submit" value="Delete">';
                        }
                    }
                    echo '&nbsp;<input type="submit" name="submit" value="Cancel">';

                //Request to Return a Resource (aka end a signup NOW)
                } elseif ($_POST['submit'] == "Return Resource") {
                    if (return_desc == "") {
                        echo '<input type="submit" name="submit" value="Confirm Resource Return">';
                    } else {
                        echo '<input type="submit" name="submit" value="Confirm ' . return_desc . '">';
                    }
                    echo ' <input type="submit" name="submit" value="Cancel">';

                //Request to Delete a Signup?
                } elseif ($_POST['submit'] == "Delete") {
                    echo '<input type="submit" name="submit" value="Confirm Delete">';
                    echo ' <input type="submit" name="submit" value="Cancel">';
                    $prioritycheck = (getsignuppriority ($_POST['usn'], $_POST['signup'])==1);
                    if ($prioritycheck && ($_POST['starttime'] - (zulutime()+timezone) <= warn_late_delete) && !$administrator)
                        echo '<tr><b>Reason for Deletion:</b><input type="text" name="ReasonCancel">';

                //Currently viewing a signup's information (not editing)
                } elseif ($_POST['submit'] == "info") {
                    echo '<input type="submit" name="submit" value="OK">';
                    if ($_POST['starttime']<zulutime()+timezone & $_POST['endtime']>zulutime()+timezone)
                        if (enable_noshow && !in_array($users[$usn]['firstname'] . $users[$usn]['lastname'], $do_not_noshow))
                            echo '&nbsp;<input type="submit" name="submit" value="No-Show">';

                //Request to Log signup as No-Show
                } elseif ($_POST['submit'] == "No-Show") {
                    echo '<input type="submit" name="submit" value="Confirm No-Show">';
                    echo ' <input type="submit" name="submit" value="Cancel">';

                //Calculated priority is available, or we're doing a "special" edit (calendar auth w/o signup ability)
                } elseif (($priority <= num_signups & $priority > 0) | $_POST['submit'] == "As User" | $_POST['submit'] == "Special") {
                    if ($_POST['submit'] == "As User" | $_POST['submit'] == "Special") {
                        echo '<input type="submit" name="submit" value="Sign-up As User">';
                        echo ' <input type="submit" name="submit" value="Clear">';
                    } else {
                        if ($signuppermitted || !$calendarauth)
                            echo '<input type="submit" name="submit" value="Sign-up">';
                        if (allow_signup_as_others || $administrator || ($calendarauth && calendarauth_as_others)) 
                            echo ' <input type="submit" name="submit" value="As User">';
                        echo ' <input type="submit" name="submit" value="Clear">';
                    }

                } else  {  //no signups available
                    echo ' <input type="submit" name="submit" value="Submit">';
                    if ($calendarauth) 
                        echo ' <input type="submit" name="submit" value="Special">';
                    echo ' <input type="submit" name="submit" value="Clear">';

                }
        } else {
                echo '<input type="submit" name="submit" value="Submit">';
                echo ' <input type="submit" name="submit" value="Clear">';
        }

?>
      </td>
    </tr>
    </table>
  </tr>
</table>
</td>
  </tr>
</table>

<?
}

if ((public_calendar & enable_login) | $thisuserid)        {        //don't show table if enable_login is false (unless already logged in)

// - - - - - - - - - - - - - - - - - - - - - - - - - - -
// Draw sign-up table

// Throughout this code the string "otherforminfo" is reassigned to pass appropriate
// values back when a click on the table is done. This string takes different forms depending
// on what is being clicked. Be cautious to make certain you change ALL instances of
// otherforminfo. You may also need to add items to the "edit" and "info" calls when
// a user clicks on a previous signup.

if ($no_edit | in_array($_POST['submit'],array("edit"))) {
        unset($_POST['starttime']);
        unset($_POST['endtime']);
        unset($_POST['comment']);
}        //if editing or viewing info, the above items should NOT be used in drawing the table

// - - - - - - - - - - - - - - - -
// draw table header (w/controls)

echo '<table border="0" cellspacing="0" cellpadding="0" bordercolor="#cccccc" bgcolor="#ffffff">';

//convert any valid tabledatetext into actual tabledate
$newtabledate = validatedate($_POST['tabledatetext']);
if ($newtabledate != -1) 
    $tabledate = $newtabledate;
elseif (in_array("tabledate", array_keys ($_POST)))
    $tabledate = $_POST['tabledate'];
else
    $tabledate = zulutime()+timezone;

//Show weekends?
if (noweekends) {
    if (date("w",$tabledate)==6) $tabledate += 2*60*60*24;  //sat->next monday
    if (date("w",$tabledate)==0) $tabledate -= 2*60*60*24;  //sun->last friday
}

echo '<input type="hidden" name="tabledate" value="' . $tabledate . '">';
echo "<tr>\n";

echo '<td nowrap bgcolor="' . calendar_head_color . '">';
if ($_POST['vi'] == 'week') {
    echo '&nbsp;Week of:';
} else {
    echo '&nbsp;Date:';
}

echo '&nbsp;<input type="text" name="tabledatetext" id="tabledatetext" size="16" maxlength="25" value="' . date("M d, Y",$tabledate) . '"';
if ($_POST['submit'] != "edit") echo ' language="JavaScript"';
echo ">";

//add calendar script
?>
<script type="text/javascript">
    function updateview () {
    <? if ($_POST['submit'] != "edit") { //PHP to disable auto-refresh if in edit mode ?>
        document.SignupForm.tabledate.value = Date.parse(document.SignupForm.tabledatetext.value)/1000;
        document.SignupForm.UpdateView.click();
    <? } ?>
    }

    Calendar.setup({
        inputField     :    "tabledatetext",  
        ifFormat       :    "%b %e, %Y",      
        timeFormat     :    "24",
        showOthers     :    true,             
        onUpdate       :    updateview,
        electric       :    false,            
    });
</script>
<?


//Return to today button
$otherforminfo = 'UID=' . $UID . '&st=' . $_POST['starttime'] . '&en=' . $_POST['endtime'] . '&co=' . make_get_str($_POST['comment']) . '&re=' . $_POST['resource'] . '&vi=' . $_POST['vi'] . '&tabledatetext=today';
echo "&nbsp;";
echo "<acronym title=\"View Today\"><a href=\"" . schedule_prog . '?' . $otherforminfo . "\"";
echo " onmouseover=\"javascript:window.status='View today';\" onmouseout=\"javascript:window.status='';\"";
echo '><img src="today.gif" border=0 align="center" alt="[View Today]"';
echo '></a></acronym>';

//Veiw menu:
?>
        &nbsp;View:
        &nbsp;<select name="vi" language="JavaScript" onChange="document.SignupForm.UpdateView.click()">
                <option value="day"   <? if ($_POST['vi']=="day") echo " selected";   ?> >Day</option>
                <option value="week"  <? if ($_POST['vi']=="week") echo " selected";  ?> >Week</option>
                <option value="month" <? if ($_POST['vi']!="day" & $_POST['vi']!="week") echo " selected"; ?> >Month</option>
        </select>
<?

if (!$thisuserid) {
    if (count($resourcelist)==1) {
        $oneitem = end($resourcelist);
        echo '<input type="hidden" name="resource" value="' . $oneitem['resource'] . '">';
    } elseif (count($resourcelist)>1) {
        echo '&nbsp;&nbsp;Resource:&nbsp;<select name="resource" ';
        echo ' language="JavaScript" onchange="document.SignupForm.UpdateView.click()"';
        echo ' >';

        if (($_POST['resource'] == NULL) && (!default_resource_all)) $_POST['resource'] = 1;
        foreach (sortby($resourcelist,'order') as $oneitem) {
                echo "<option value=\"" . $oneitem['resource'] . "\"";
                if (in_array($oneitem['resource'],$selectedres)) echo " selected";
                echo ">${oneitem['long']}</option>\n";
        }
        if (count($resourcelist)>1) {
            if (in_array("All",$selectedres))
                    echo "<option value=\"All\" selected>All</option>";
            else    echo "<option value=\"All\">All</option>";
        }
        echo '</select>';
    }
}

echo '&nbsp;&nbsp;<input type="submit" name="UpdateView" value="Update View">';

echo '<font size="-1">&nbsp;&nbsp;[* = click for details]&nbsp;&nbsp;&nbsp;&nbsp;Now:' . date(timeformat,zulutime()+timezone) . "</font>";
echo "</td></tr>\n";

echo "<tr><td>\n";

//Use appropriate table display routine
if ($_POST['vi']=="day" | $_POST['vi']=="week")
    if (vertical_view) {
        include('dayview.php');
    } else {
        include('dayview_horiz.php');
    }    
else
        include('monthview.php');

echo "</td></tr></table>\n";

// end of draw table - - - - - - - - - - - - - - - - - - - - - - - - - - -
}

scheduleunlock();
?>

</form>


<p>
<? printfooter(false,false); ?>
</body>
</html>
<?//==============================================================================
//JMS 8/18/05
//TBD: *** Add config for:
//    + autofill times when not submitted
//    + calendar css
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!