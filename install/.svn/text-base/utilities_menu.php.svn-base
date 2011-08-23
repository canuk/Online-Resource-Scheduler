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

<html>
<head>
<title>Scheduling Service Utilities</title>
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

</head>

<?php
require_once('functions.php');

echo '<body background="' . page_background . '" nosave>';

//====================================================
//Beginning of main routine
//====================================================

//copy all items from get and sanitize all values
foreach ($_GET as $key=>$value) $_POST[$key]=$value;
foreach ($_POST as $key=>$value) if ($key!="htmltext") $_POST[$key]=sanitize($value);

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

if (!schedulelock($_POST['UID'])) {
        echo "<h3><center>Sorry, Scheduling is currently busy. Please check back shortly.</center></h3>";
        return(false);
}

$warning = "";                                //empty warning

//if doing an installation, force validateusers to be run to make sure everything is in order in the userfiles
if ($_POST['forceconfig']=='AskedByInstall') validateusers();

//use remote authorization if no UID found (must be enabled in configuration)
$_POST['REMOTE_USER']=$REMOTE_USER;

$userinfo   = useridlookup($_POST);
$thisuserid = $userinfo['usn'];
$UID        = $userinfo['UID'];

$_POST['thisuserid'] = $thisuserid;        //copy to use in functions

$resourcelist = getresources();                        //get list of valid resources
$users        = getusers();

$administrator = checksec($users[$thisuserid]['sec'],100);
$calendarauth  = checksec($users[$thisuserid]['sec'],10);

if (!$thisuserid)        {
    logaction();
    ?>

  <form name="SignupForm" <? echo 'method="' . submit_method . '" action="' . util_prog . '?"'; ?> >

  <table border="1" cellspacing="0" cellpadding="4" bgcolor="<? if (defined('login_form_color')) echo login_form_color; else echo "#ffffff"; ?>">
    <tr align="center"><td>
        <? if (!file_exists(header_image)) echo "<b><font size=\"+1\" color=\"#000099\">Online Resource Scheduler</font></b>&nbsp;&nbsp;\n";
           if (exit_URL != "") echo '<a href="' . exit_URL . '">';
           if (file_exists(header_image)) {
               echo '<img src="' . header_image . '" border=0>';
           }
           if (exit_URL != "") echo '<font color="#990000" size="-1"><em>Home</em></font></a>'; ?>
   </td></tr>

    <?
        if ($_POST['forgottenpassword']==1) {
            //E-mail login link to user who forgot their password
            echo "<tr><td><font color=\"#990000\">";
            forgottenpassword($_POST);
            echo "</font></td></tr>";
        }

        if (in_array("pwsupplied", array_keys ($_POST)) & $_POST['username'] != "") {
            echo "<tr><td><font color=\"#990000\"><b>That Username / Password combination is not valid</b></font>";
            if ($_POST['forceconfig']!="") { ?>
                    <br><b><font color="#990000">NOTE:</font> You must log in as an administrator to finish the installation.</b>
            <? }
            echo "<br>Forgotten your password? <a href=\"?forgottenpassword=1&username=". $_POST['username'] . "\"><em>Yes! please e-mail me a link to log in</em></a>.";
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
   </table>

   <? if ($_POST['forceconfig']!="") echo '<input type="hidden" name="forceconfig" value="'.$_POST['forceconfig'].'">'; //remember this for when they manage to log in ?>
   <input type="hidden" name="pwsupplied" value="1">
   </>

<?
}

if (($thisuserid>0) & !enable_login & !$administrator) {
        //if login disabled, give message if user is NOT administrator
        echo "<h3><center>Sorry, Scheduling is currently under construction. Please check back later.</center></h3>";
}

if (($thisuserid>0) & (enable_login | $administrator)) {
        //if login disabled, only allow login if user is administrator

        //special request to force configuration page open?
        if ($administrator & $_POST['forceconfig']=='AskedByInstall') {

                $_POST['submit']         = 'Submit';
                $_POST['mode']           = 'configureprogram';
                $_POST['ConfigPassword'] = configpassword;

        }

        //no submit mode AND username WAS provided implies list user
        //(NOTE: without username, assume they want the maintenance menu)

        if (!in_array("submit", array_keys ($_POST)) & in_array("mode", array_keys ($_POST)))
                $_POST['submit'] = $_POST['mode'];


        //Deal with ambiguious submit buttons by appending "mode" if present
        // (some submit buttons have identical names and the only way to tell which form they came from
        //  is to look at "mode". Here we append mode and look for that in our switch below)

        if (in_array($_POST['submit'],array("Cancel","Update","Submit")) & in_array("mode", array_keys ($_POST)))
                $_POST['submit'] = $_POST['submit'] . $_POST['mode'];


        //- - - - - - - - - - - Actions Menu - - - - - - - - - - - -
        ?>

        <table border=0><tr><td valign="top"> <!outer table>

        <table border=0 cellspacing=3 cellpadding=0><!actions table>
          <tr valign="middle">
            <td bgcolor="<? if (defined('menu_color')) echo menu_color; else echo "#ffffff"; ?>">
              <table border=0 cellspacing=0 cellpadding=0 width="100%">
              <tr><td align="left">
                <b><font color="#660000">
                User:
                <? echo $users[$thisuserid]['firstname'] . ' ' . $users[$thisuserid]['lastname'];  ?>
                  </font></b>
                <?
                        if ($administrator) echo "<font size=-2>&nbsp;(Admin.)</font>";
                        elseif ($calendarauth) echo "<font size=-2>&nbsp;(Calendar Auth.)</font>";
                 ?>

              </td><td align="right">
                <font face="Arial, Helvetica, sans-serif" size=-1>
                <a <? echo 'href="' . schedule_prog . '?UID=' . $UID . '"'; ?> >
                <font color="#000066" size="-1">Return&nbsp;to&nbsp;Calendar&nbsp;View</font></a>&nbsp;

                /&nbsp;
                <a href = <? echo '"' . schedule_prog . '?UID=' . $UID . '&submit=logout"'; ?> >
                <font color="#000066" size="-1">Log&nbsp;Out</font></a>&nbsp;
                </font>
              </td></tr></table>

          </td></tr>

        <tr>
        <td valign="top" bgcolor="<? if (defined('menu_color')) echo menu_color; else echo "#ffffff"; ?>">
        <font face="Arial, Helvetica, sans-serif" size=-1>
        &nbsp;<b>Actions:</b>&nbsp;

        <a href = <? echo '"' . list_signups_prog . '?UID=' . $UID . '&submit=listuser"'; ?> >
        <font color="#000066" size="-1">List&nbsp;User&nbsp;Signups</font></a>&nbsp;

        /&nbsp;
        <a href = <? echo '"' . list_signups_prog . '?UID=' . $UID . '&submit=listresource"'; ?> >
        <font color="#000066" size="-1">List&nbsp;Resource&nbsp;Signups</font></a>&nbsp;

        <? if (allow_memberlist_view || $calendarauth) { ?>
        /&nbsp;
        <a href = <? echo '"' . list_members_prog . '?UID=' . $UID . '"'; ?> >
        <font color="#000066" size="-1">List&nbsp;Members</font></a>&nbsp;
        <? } ?>

        <? if (allow_email_any || $calendarauth) { ?>
        /&nbsp;
        <a href = <? echo '"' . email_announcement_prog . '?UID=' . $UID . '"'; ?> >
        <font color="#000066" size="-1">E-Mail&nbsp;Announcement</font></a>&nbsp;
        <? } ?>

        <? if ($thisuserid>1) { ?>
                /&nbsp;
                <a href = <? echo '"' . user_management_prog . '?UID=' . $UID . '&usn=' . $thisuserid . '&submit=editmember&listmembers=1"'; ?> >
                <font color="#000066" size="-1">Edit&nbsp;My&nbsp;Account</font></a>&nbsp;
        <? } ?>

        <? if ($calendarauth) { ?>

                <br>

                &nbsp;<b>Maintenance:</b>&nbsp;

                <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=listinfringements"'; ?> >
                <font color="#000066" size="-1">Sign-up&nbsp;Infringements</font></a>&nbsp;

                <? if (recurring_enabled && (($administrator && !recurring_calauth) || ($calendarauth && recurring_calauth))) { ?>

                    /&nbsp;
                    <a href = <? echo '"' . recurring_prog . '?UID=' . $UID . '"'; ?> >
                    <font color="#000066" size="-1">Recurring&nbsp;Signups</font></a>&nbsp;

                <? } ?>
                <? if (!$administrator && $calendarauth) { ?>
                        /&nbsp;
                        <a href = <? echo '"' . resource_management_prog . '?UID=' . $UID . '&submit=listwarnings"'; ?> >
                        <font color="#000066" size="-1">Resource&nbsp;Warnings</font></a>&nbsp;
                <? } ?>                

                <? if ($administrator) { ?>

                        /&nbsp;
                        <a href = <? echo '"' . user_management_prog . '?UID=' . $UID . '"'; ?> >
                        <font color="#000066" size="-1">User&nbsp;Management</font></a>&nbsp;
                        /&nbsp;
                        <a href = <? echo '"' . resource_management_prog . '?UID=' . $UID . '&submit=listresources"'; ?> >
                        <font color="#000066" size="-1">Resource&nbsp;Management</font></a>&nbsp;
                        /&nbsp;
                        <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=more"'; ?> >
                        <font color="#000066" size="-1">More...</font></a>&nbsp;

                <? } ?>

        <? }} ?>

        </font>
        </td></tr></table>
        <td valign="top">
        </td></tr></table>

<?

if (!$thisuserid) {
    scheduleunlock();
    printfooter();
    return(false);
}

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
//JMS 2/19/04
// -removed .htaccess logic
//JMS 3/29/04
// -always do dailyaction test with non-logged in user
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
