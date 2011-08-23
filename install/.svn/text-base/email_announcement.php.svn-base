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

if (!isset($_POST['mode']))
    $mode = false;
else 
    $mode = $_POST['mode'];

if (!allow_email_any && !$calendarauth) {

    echo "<br><br><b>You are not authorized to make announcements.</b><br><br><br>";

} elseif ($mode == "Cancel Announcement") {
    $_POST['submit'] = "";

} else {

    // Shorten the submit variable value
    if ($mode == "Confirm Announcement" || $mode == "Edit Announcement") {
        $sendToAll   = unserialize(base64_decode($_POST['sendToAll']));
        $anncSubject = unserialize(base64_decode($_POST['anncSubject']));
        $anncBody    = unserialize(base64_decode($_POST['anncBody']));
        $anncUsers   = unserialize(base64_decode($_POST['anncUsers']));
        $sendAsMe    = unserialize(base64_decode($_POST['sendAsMe']));
        if (!is_array($anncUsers))
            $anncUsers = array();
        $anncResources = unserialize(base64_decode($_POST['anncResources']));
        if (!is_array($anncResources))
            $anncResources = array();
    } else {
        if ($_POST['sendToAll'] == "Send to ALL USERS")
            $sendToAll = true;
        elseif ($_POST['sendToAll'] == "Send Message to Checked Users")
            $sendToAll = false;
        $anncSubject   = $_POST['anncSubject'];
        $anncBody      = $_POST['anncBody'];
        $anncResources = $_POST['anncResources'];
        $sendAsMe      = $_POST['sendAsMe'];
        if (!is_array($anncResources))
            $anncResources = array();

        $anncUsers = $_POST['anncUsers'];
        if (!is_array($anncUsers))
            $anncUsers = array();
    }

    // Make user-submitted strings usable for mailing
    $anncSubject = stripcslashes($anncSubject);
    $anncBody = stripcslashes($anncBody);

    $validated = false;
    $errors = array();
    if ($mode == "validateform" || $mode == "confirmmessage" || $mode == "Confirm Announcement") {
        // Check to see if a user is trying to do something that they are not allowed
        if (!allow_mass_email && !$calendarauth) {
            if ($sendToAll)
                $errors['permission'] = "You are not authorized to send a message to all users.";
            else if (!$sendToAll && count($anncResources)>0 )
                $errors['permission'] = "You are not authorized to send a message to all users<br>of a given resource.";
            else if (!$sendToAll && count($anncUsers)>1 )
                $errors['permission'] = "You are not authorized to send a message to more than one user.";
        }

        // Ensure that we got all the information we needed from the user
        if ($anncSubject == "")
            $errors['anncSubject'] = "An announcement subject is required.";
        if ($anncBody == "")
            $errors['anncBody'] = "An announcement body is required.";
        if (!$sendToAll &&
            (count($anncResources)==0 && count($anncUsers)==0))
            $errors['recipients'] = "No users were selected to receive the message.";

        // Redisplay form if errors exist
        if (count($errors)) {
            $mode = "printform";
        } else {
            // If user just filled out form, send them to confirmation form
            if ($mode == "validateform")
                $mode = "confirmmessage";
            elseif ($mode == "Confirm Announcement")
                $mode = "sendmessage";
            $validated = true;
        }
    }

    if ($mode == "confirmmessage" && $validated == true) {
        echo 'Please confirm that you want to send the following message to ';
        if ($sendToAll)
            echo '<b>ALL USERS</b>:';
        else 
            echo 'the users you selected:<hr>';

        ?>
        <table border="0">
        <tr><td align="right" valign="top"><b>To:</b>&nbsp;</td><td><?
            if ($sendToAll)
                echo "ALL scheduler users<br>\n";
            else {
                if (count($anncUsers)>0)
                    foreach ($anncUsers as $user) echo getcontact($user,'firstname') . " " . getcontact($user,'lastname') . "<br>\n";
                if (count($anncResources)>0)
                    foreach ($anncResources as $resource) echo "All users of " . $resourcelist[$resource]['long'] . "<br>\n";
            }
            ?></td></tr>
        <tr><td align="right" valign="top"><b>Subject:</b>&nbsp;</td><td><pre><? print wordwrap($anncSubject,80); ?></pre></td></tr>
        <tr><td align="right" valign="top"><b>Body:</b>&nbsp;</td><td><pre><? print wordwrap($anncBody,80); ?></pre></td></tr>
        </table>
        <form name="ConfirmAnnouncement" <? echo 'method="' . submit_method . '" action="' . email_announcement_prog . '"'; ?> >
        <input type="hidden" name="UID" <? echo "value=\"$UID\""; ?>>
        <input type="hidden" name="submit" value="emailannouncement">
        <input type="hidden" name="anncSubject" value="<? echo base64_encode(serialize($anncSubject)); ?>">
        <input type="hidden" name="anncBody" value="<? echo base64_encode(serialize($anncBody)); ?>">
        <input type="hidden" name="sendToAll" value="<? echo base64_encode(serialize($sendToAll)); ?>">
        <input type="hidden" name="anncUsers" value="<? echo base64_encode(serialize($anncUsers)); ?>">
        <input type="hidden" name="anncResources" value="<? echo base64_encode(serialize($anncResources)); ?>">
        <input type="hidden" name="sendAsMe" value="<? echo base64_encode(serialize($sendAsMe)); ?>">
        <input type="submit" name="mode" value="Confirm Announcement">&nbsp;
        <input type="submit" name="mode" value="Edit Announcement">&nbsp;
        <input type="submit" name="mode" value="Cancel Announcement">
        </form>

        <?
    } else {
    
        if ($mode == "sendmessage" && $validated = true) {
            if (!$sendToAll) {
                $mailingusns = array();
    
                if (allow_mass_email || $calendarauth) {
                    foreach($anncResources as $resource) {
                        foreach($users as $user) {
                            if (!checkpermissions($user['resourceblock'],"1",$resource['resource']))
                                array_push($mailingusns,$user['usn']);
                        }
                    }
                }

                foreach($anncUsers as $user)
                        array_push($mailingusns,$user);
                $mailingusns = array_unique($mailingusns);

                // Filter out users that cannot be contacted via email
                $mailableusns = array();
                foreach($mailingusns as $mailingusn) {
                    if (getcontact($mailingusn) !== "")
                        array_push($mailableusns,$mailingusn);
                }
                $mailableusns = array_unique($mailableusns);
            } elseif ($sendToAll && (allow_mass_email || $calendarauth)) {
                // Build a list of unique users that have e-mail addresses
                $mailableusns = array();
                foreach($users as $user) {
                    if (getcontact($user['usn']) != "")
                        array_push($mailableusns,$user['usn']);
                }
                $mailableusns = array_unique($mailableusns);
            }

            // Based on the list of mailable users, build an address list
            $anncAddresses = array();
            foreach($mailableusns as $mailableusn ) {
                    array_push($anncAddresses,getcontact($mailableusn));
            }
            $anncAddresses = array_unique($anncAddresses);

            // Get a single email address from the current user to place in the X-Apparently-From header
            $senderemail = false;
            if (getcontact($thisuserid) !== "") {
                if (getcontact($thisuserid,'email1') !== "")
                    $senderemail = getcontact($thisuserid,'email1');
                else $senderemail = getcontact($thisuserid,'email2');
            }

            $addheaders = "From: " . email_from . "\n";
            if ($senderemail) {
                $temp = '"' . getcontact($thisuserid,'lastname') . ", " . getcontact($thisuserid,'firstname') .
                                '" <' . $senderemail . ">";
                if (announce_as_user) $addheaders = "From: " . $temp . "\n";
                $addheaders = $addheaders . "X-Apparently-From: " . $temp . "\n";
            }

            if (scheduling_URL!="") $anncBody .= "\n\n\n----------------------\nSent via scheduling system at\n  " . scheduling_URL;

            foreach($anncAddresses as $address)
                sendemail($address, $anncSubject, $anncBody, $addheaders);
            echo "Your announcement has been sent to " . count($anncAddresses) . " user(s).";

        } else {

            echo "<b>Send an E-Mail Announcement:</b><br><br>";

            if (count($errors)) {
                echo '<font color="red"><b>Your message could not be sent for the following reasons:<br>';
                foreach($errors as $error)
                        echo $error . '<br>';
                echo '</b></font><br>';
            }
            ?>

            <form name="Announcement" <? echo 'method="' . submit_method . '" action="' . email_announcement_prog . '"'; ?> >
            <input type="hidden" name="UID" <? echo "value=\"$UID\""; ?> >
            <input type="hidden" name="submit" value="emailannouncement">
            <input type="hidden" name="mode" value="validateform">

            <?
            echo '<b>Compose Message:</b>';
            echo '<table border="0">';
            echo '<tr><td align="right">Subject:</td><td><input type="text" size="80" maxlength="80" name="anncSubject" ';
            echo 'value="' . $anncSubject . '" style="font-family:monospace;"></td></tr>';
            echo '<tr><td align="right" valign="top">Message:</td><td><textarea rows="10" cols="80" name="anncBody">' . $anncBody . '</textarea></td></tr>';
            echo '</td></tr></table><br>';

            echo '<b>Message Recipient(s):</b>';
            echo '<table border="1" cellspacing="0" cellpadding="5">';
            echo '<tr>';
            if (allow_mass_email || $calendarauth) {
                echo '<td><b>Users of Resource:</b><br>';
                echo '<select name="anncResources[]" size="5" multiple>';
                // Preserve state of any checkboxes that the user checked
                foreach (sortby($resourcelist,'order') as $resource) {
                    echo '<option value="' . $resource['resource'] . '"';
                    if (in_array($resource['resource'],$anncResources))
                        echo " selected";
                    echo '>';
                    echo $resource['long'] . '</option>' . "\n";
                }
                echo '</select>';
                echo '</td><td>';
            } else {
                echo '<td colspan="2">';
            }

            // Get a list of users with at least one email address
            $mailableusns = array();
            foreach(sortbyname($users) as $user) {
                if (getcontact($user['usn']) != "")
                    array_push($mailableusns,$user['usn']);
            }

            echo "<b>Specific Users:</b><br>\n";
            // Preserve state of any checkboxes that the user checked
            echo '<select name="anncUsers[]" size="5"';
            if (allow_mass_email || $calendarauth) echo ' multiple ';
            echo '>';
            foreach($mailableusns as $anncusn) {
                echo '<option value="' . $anncusn . '"';
                if (in_array($anncusn,$anncUsers))
                    echo " selected";
                echo '>';
                echo $users[$anncusn]['lastname'] . ', ' . $users[$anncusn]['firstname'] . '</option>' . "\n";
            }
            echo '</select><br>';

            echo '</td>';

            if (allow_mass_email || $calendarauth) {
                echo '<td align="center" valign="top" rowspan="2"><b>All Users</b><p>';
                echo '<input type="checkbox" name="sendToAll" value="Send to ALL USERS"';
                if ($sendToAll) echo "checked";
                echo '> Send to ALL USERS<br>';

                //echo '<input type="submit" name="sendToAll" value="Send to ALL USERS">';
                echo '</td>';
            }

            echo '</tr><tr><td align="center" colspan="2">';
            if (allow_mass_email || $calendarauth) {
                echo "<font size=-1>(Use control-click to select multiple resources/users)</font><br>\n";

                //echo '<input type="submit" name="sendToSome" value="Send to Selected Users"><br>';

                echo '</td></tr><tr><td align=center colspan=3>';

            } else {
                //echo '<input type="submit" name="sendToSome" value="Send to Selected User"><br>';
            }
            echo '<input type="submit" name="preview" value="Preview"><br>';
            echo '</td></tr></table><br>';
            echo '</form>';
        }
    }     //confirm message
}      //authorization test

scheduleunlock();
printfooter();

//--------------------------------------
//
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!