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

    case "Updateusermanagement" : // save changes
    case "Add New User" : 		  // save changes IF user passes validation
    case "Update User" :           // save changes IF user passes validation
    case "Cancelusermanagement" : // cancel edit of user history
    case "Cancel New User" :      // cancel edit of new user account
    case "usermanagement" :       // show user management main screen
    case "editmember" :           // edit a member's account info
    case "addmember" :            // add a new member
    case "addeditmember" :        // enter account info for new member
    case "deleteuser" :           // delete a specified member
    case "confirmdeleteuser" :    // delete a specified member
    case "restrictuser" :         // restrict a user for future signups
    case "confirmrestrictuser" :  // restrict a user for future signups
    case "unrestrictuser" :       // remove user account restrictions
    case "activateuser"   :       // mark user as active (remove inactive status)
    case "Modify User" :
    case "bulkaddition" :         // give dialog for bulk addition of users
    case "Bulk-Add Users" :              // actually add users in bulk
    case "" :

        echo "<b>User Management:</b>";
        if (demomode) {echo "<br><font size=-1>Demo mode - saving disabled</font>";}

        if ($_POST['submit'] == "Cancelusermanagement") {
                $_POST['usn'] = "";
                $_POST['username'] = "";
        }

        //NOT an administrator?
        if (!$administrator) {
                //(1) DO NOT let them modify anybody except themselves

                $_POST['usn'] = $thisuserid;

                //(2) DO NOT let them do any of the following commands

                if (in_array($_POST['submit'],array('Add New User','bulkaddition','Bulk-Add Users',
                        'addusermanagement','add','deleteuser','confirmdeleteuser','restrictuser','unrestrictuser')))
                        $_POST['submit'] = 'Modify User';        //convert to simple "list" command

        }

        if ($_POST['submit']=="editmember" && empty($_POST['usn']) && $thisuserid==1) {
            //do not edit administrator account!
            $_POST['submit']='Modify User';
            ?>
            <p><font size=+1 color="#990000">Administrator account information is set in the
            <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=configureprogram#administratoraccountinformation"'; ?> >Configuration Settings page</a>
            </font></p>
            <?
        }

        if ($_POST['submit']=="confirmdeleteuser") {
            $usn      = $_POST['usn'];
            removeuser($usn);
            $users = getusers();
        }


        if ($_POST['submit'] == "Bulk-Add Users") {
            echo "<p>";

            //specify requied and valid field names
            $required = array('lastname','pin','username');
            $valid    = array('lastname','firstname','username','pin','email1','email2','homephone','workphone','ratings');

            //Split submitted lines out into individual arrays
            $lines = explode('\r\n',$_POST['bulkusers']);
            $badlines = array();
            foreach ($lines as $ind=>$oneline) {
                if ($ind==0) {
                    $keys = explode(',',$oneline);
                    foreach ($keys as $ind=>$key) $keys[$ind] = trim($key);
                } else {
                    $temp = explode(',',$oneline);
                    if (count($temp)==1 && empty($temp[0])) continue;
                    
                    foreach ($keys as $keyind=>$key) {
                        $val = trim($temp[$keyind]);

                        //check for empty required fields
                        if (in_array($key,$required) && empty($val)) {
                            $badlines[] = "Empty value for required field (&quot;$key&quot; in line " . ($ind+1) . ")";
                        }
                    
                        $values[$ind-1][$key] = $val;
                    }

			        if (lookupusn($values[$ind-1]['username']) != $usn && lookupusn($values[$ind-1]['username']) != "")
                        $badlines[] = "Username already in use (line " . ($ind+1) . ")";
                }
            }

            $badkeys  = array_diff($keys,array_intersect($valid,$keys));


            if (count(array_intersect($required,$keys))<count($required)) {

                echo "<font color=#990000><b>Error: Missing necessray fields</b></font>";
                $_POST['submit'] = "bulkaddition";

            } elseif (count($badkeys)>0) {

                echo "<font color=#990000><b>Error: Invalid fields supplied (";
                foreach ($badkeys as $ind=>$key) {
                    echo "&quot;" . $key . "&quot;";
                    if ($ind+1<count($badkeys)) echo ",";
                }
                echo ")</b></font>";
                $_POST['submit'] = "bulkaddition";        

            } elseif (!empty($badlines)) {

                echo "<font color=#990000><b>There were problems with your request. See errors below.</b></font><br>";

                $_POST['submit'] = "bulkaddition";        
            
            } else {
                foreach ($values as $newuser) {

                    if (empty($newuser)) continue; //skip if no info (blank line)

                    $newuser['username']  = strtolower($newuser['username']);
                    $newuser['firstname'] = ucfirst($newuser['firstname']);
                    $newuser['lastname']  = ucfirst($newuser['lastname']);

                    $usn = assignusn($newuser['username'],$newuser['lastname'],$newuser['firstname'],defalut_user_sec);
                    $_POST['usn'] = $usn;
                    $users = getusers();

                    foreach ($newuser as $key=>$item) $users[$usn][$key] = $item;
                    putusers($users);

                    echo "Created user " . $newuser['firstname'] . " " . $newuser['lastname'] . " ("  . $newuser['username'] . ")<br>";
                    $users = getusers();

                }
            }
        }

        if ($_POST['submit'] == "bulkaddition") {

            if (empty($_POST['bulkusers']))
                $_POST['bulkusers'] = "lastname, firstname, username, pin\r\n";
        
            ?>
            <p><b>Bulk Addition of Users</b><br>
            Users can be created in bulk by entering the relavent information in the edit box below. <br>
            The first line must be a "key" line containing the fields (separated by commas) that will be provided for each user.

            <p><b>Valid field names are:</b> <font color="#990000"><b>lastname</b></font>, firstname, <font color="#990000"><b>username</b></font>, <font color="#990000"><b>pin</b></font>, email1, email2, homephone, workphone, ratings
            <br>(Fields in <font color="#990000"><b>red</b></font> must be supplied)</p>

            Each subsequent line represents the information for one user. The information should be entered separated by commas.
            <b>Note:</b> Fields may not contain commas.

            <?
                if (!empty($badlines)) {
                    echo "<p>";
                    foreach ($badlines as $badline)
                        echo "<font color=#990000><b>Error: $badline</b></font><br>";
                }
            ?>      

            <form name="SignupForm" <? echo 'method="' . submit_method . '" action="' . user_management_prog . '"'; ?> >
            
                <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >
                <input type="hidden" name="usn" <?php echo "value=\"$usn\""; ?> >
                <input type="hidden" name="mode" value="usermanagement">
                <textarea valign="top" name="bulkusers" cols="50" rows="8"><? echo stripcslashes($_POST['bulkusers']); ?></textarea>

                <br>
                <input type="Submit" name="submit" value="Bulk-Add Users">

            </form>
            <?
            break;
        }

		if ($_POST['submit'] == "Add New User" || $_POST['submit'] == "Updateusermanagement") {
			$numerrors = 0;
			// Only need to worry about changing of bad fields with administrative users.
			if ($administrator) {
	            $usn = $_POST['usn'];

				//if (($_POST['lastname'] != "Administrator") // Administrator doesn't have a first name, so we allow this.
                // && ($_POST['firstname'] == "")) {
				//	$errors['firstname'] = "First name is required.";
				//	$numerrors++;
				//}

				if ($_POST['lastname'] == "") {
					$errors['lastname'] = "Last name is required.";
					$numerrors++;
				}
                if ($_POST['username'] == "") {
					$errors['username'] = "Username is required.";
					$numerrors++;
				} elseif (lookupusn($_POST['username']) != $usn && lookupusn($_POST['username']) != "") {
			        $errors['username'] = "This username is already in use.";
			        $numerrors++;
		        }

				if ($_POST['expire'] != "") {
                   	$validdate = validatedate($_POST['expire']);
                   	if ($validdate != -1)
		                $_POST['expire'] = date(dateformat . "/y",$validdate);
                   	else {
		                $errors['expire'] = "The expiration date entered was invalid.";
		                $_POST['expire'] = "";
                        $numerrors++;
                    }
                }
            }
            
            //check password for validity            
            $newpassword1 =  $_POST['newpassword1'];
            $newpassword2 =  $_POST['newpassword2'];
            if ((strcasecmp($newpassword1, "") != 0) || (strcasecmp($newpassword2, "") != 0)){  //was either typed?
                   if (!strcasecmp($newpassword1, $newpassword2))  {
                           if (strlen($newpassword1) < min_password_size) {
                               $errors['password'] = "Password must contain at least ".min_password_size." characters.";
                               $numerrors++;
                           }
                   } else {
                       $errors['password'] = "The entered passwords do not match.  Please re-enter new password.";
                       $numerrors++;
                   }
            } elseif ($_POST['submit'] == "Add New User") { 
                //adding new user, require password
                $errors['password'] = "A valid password is required for a new user.";
                $numerrors++;
            }

			// Send user back to data entry form in proper state
			if ($numerrors > 0) {
				if ($_POST['submit'] == "Add New User")
					$_POST['submit'] = "addeditmember";
				if ($_POST['submit'] == "Updateusermanagement")
					$_POST['submit'] = "editmember";
				$errorprefix = "<font color=\"red\"><b>";
				$errorsuffix = "</font></b>";
				reset($errors);
				while (list($errorfield, $errortext) = each($errors))
					$errors[$errorfield] = $errorprefix . $errortext . $errorsuffix;
		
			} else {

                $usn = $_POST['usn'];
                
                if ($administrator) {
                    // new user? assign USN
                    if ($usn=="") {
                        $usn = assignusn($_POST['username'],$_POST['lastname'],$_POST['firstname'],defalut_user_sec);
                        $_POST['usn'] = $usn;
                        $users = getusers();
                    }
            
                    $users[$usn]['username']  = strtolower($_POST['username']);
                    $users[$usn]['firstname'] = ucfirst($_POST['firstname']);
                    $users[$usn]['lastname']  = ucfirst($_POST['lastname']);
                }
                $users[$usn]['ratings']   = $_POST['ratings'];
                $users[$usn]['homephone'] = $_POST['homephone'];
                $users[$usn]['workphone'] = $_POST['workphone'];
                $users[$usn]['email1']    = $_POST['email1'];
                $users[$usn]['email2']    = $_POST['email2'];

                if ($administrator) {
                    //Set Permissions
                    //force administrator mode ON if they're modifying themself
                    // (don't let an administrator lock themselves out)
                    if ($usn == $thisuserid) $_POST['administrator'] = "1";

                    //administrators ALWAYS have calendar authority
                    if ($_POST['administrator']!="") $_POST['calendar'] = "1";
                    $sec = $_POST['allowsignup']*1 + $_POST['calendar']*10 + $_POST['administrator']*100 + $_POST['emaillogs']*1000;
                    $users[$usn]['sec'] = $sec;

                    //create a code which blocks out all items and subtract selected items from that code
                    $resourceblock = "b" . str_repeat("1",max(array_keys($resourcelist)));
                    if (count($_POST['resourceblock'])>0) {
                       foreach ($_POST['resourceblock'] as $value) {
                            //set resource permissions (allow)
                            $resourceblock[$value] = "0";
                        }
                    }
                    if (count($_POST['resourcemonitor'])>0) {
                       foreach ($_POST['resourcemonitor'] as $value) {
                            //set resource monitor permission
                            $temp = $resourceblock[$value] + 2;
                            $resourceblock[$value] = $temp;
                        }
                    }
                    $users[$usn]['resourceblock'] = $resourceblock;

                    if ((strcmp($_POST['expire'],"") != 0) || (strcmp($users[$usn]['expire'],"") != 0)) {
                         $validdate = validatedate($_POST['expire']);

                         if ($_POST['expire'] == "") {
                            //If the expire date is cleared, then clear the expiration field
                            $users[$usn]['expire'] = "";
                         } elseif ((strcmp($_POST['expire'],"Expired") == 0) && (strcmp($users[$usn]['expire'],"Expired") == 0)) {
                            //If the expire date is set as "Expired" and so is the user input then don't make any changes to this field
                         }else {
                            //Save the expiration date (if valid), else provide error message
                            if ($validdate != -1) {
                                $users[$usn]['expire'] = date(dateformat . "/y",$validdate);
                            } else {
                                echo "<BR><font color = 'red'><b>ERROR: Expiration date field invalid.  Please re-edit user account to update expiration.<BR>All account fields have been updated except expiration date.</b></font>";
                            }
                         }
                    }

                }

                $newpassword1 =  $_POST['newpassword1'];
                if (strcasecmp($newpassword1, "") != 0){  //don't update password if they didn't enter one
                   $newpassword1 = strtolower($newpassword1);                   //Make passwords case-INsensitive
                   $users[$usn]['pin'] = our_crypt($newpassword1);
                }

                putusers($users);

                //log event
                $data = $users[$usn];
                logaction("Updated Account Info for",$thisuserid,$data);
                
                //- - - - - - - - - - 
                //display that the user was edited/created

                echo "<table border=1 cellpadding=2 cellspacing=0><tr><td bgcolor=\"#aaffaa\">";
                echo "<p><b>User " . $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] . " (" . $users[$usn]['username'] . ") ";
                if ($_POST['submit'] == "Add New User")
                    echo "created";
                else
                    echo "updated";
                echo ".</b>";

                //Called by list_members? return to that now
                if (!empty($_POST['listmembers'])) {
                    echo "</td></tr></table>";
                    $noheader = true;
                    include(list_members_prog);
                    return(true);
                }
                
                echo "<br><br>";

                $fontsize="-1";
                ?>
                <table border=0 cellpadding="0" cellspacing="0">
                <tr align="left" valign="bottom">
                <td nowrap><b><font size="<?echo $fontsize; ?>">Username</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Last Name</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">First Name</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>"><? echo info_fieldname; ?></font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Home Phone</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Work Phone</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Email 1</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Email 2</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
                <td nowrap><b><font size="<?echo $fontsize; ?>">Expire</font></b></td>
                </tr>
                <tr align="left">
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>">
                    <? echo $users[$usn]['username']; ?>
                    </a></font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>">
                    <? echo $users[$usn]['lastname']; ?>
                    </a></font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>">
                    <? echo $users[$usn]['firstname']; ?>
                    </a></font></td>
                <td></td>
                <td valign="top"><font size="-2"> <? echo $users[$usn]['ratings']; ?> </font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>"> <? echo $users[$usn]['homephone']; ?> </font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>"> <? echo $users[$usn]['workphone']; ?> </font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>">
                            <? echo "<a href=\"mailto:{$users[$usn]['email1']}\">{$users[$usn]['email1']}</a>"; ?>
                    </font></td>
                <td></td>
                <td valign="top" nowrap><font size="<?echo $fontsize; ?>">
                <? echo "<a href=\"mailto:{$users[$usn]['email2']}\">{$users[$usn]['email2']}</a>"; ?>
                </font></td>
                <td></td>
                <td valign="top"><font size="-2"> <? echo $users[$usn]['expire']; ?> </font></td>
                </tr></table>
                <br>
                <font size="-1">
                <?
                $lastactivity = $users[$usn]['lastactivity'];
                if (stristr($lastactivity, "inactive") != FALSE) {
                       echo "User account is <b>INACTIVE</b> ";
                }else  echo "User account is <b>active</b> ";
                echo ($users[$usn]['restricted'] ? "and <b>RESTRICTED</b> " : "and <b>unrestricted</b> ");
                
                if ($administrator) {
                        echo "and ";
                        echo "<ul><li>";
                        if (checksec($sec,1)) echo "Can"; else echo "<b>Cannot</b>"; echo " sign-up resources";
                        echo "</li><li>";
                        if (checksec($sec,10)) echo "Has"; else echo "<b>Does not have</b>"; echo " calendar authority";
                        echo "</li><li>";
                        if (checksec($sec,100)) echo "Is"; else echo "<b>Is not</b>"; echo " an administrator";
                        echo "</li><li>";
                        if (checksec($sec,1000)) echo "Does"; else echo "<b>Does not</b>"; echo " receive e-mailed logs";
                        echo "</li></ul>";

                        echo "User has access to the following resources:<p><ul>";
                        $access = array();
                        foreach (sortby($resourcelist,'order') as $oneitem)
                                if (checkpermissions($users[$usn]['resourceblock'],"0",$oneitem['resource']) | checkpermissions($users[$usn]['resourceblock'],"2",$oneitem['resource'])) $access[] = $oneitem;

                        echo "<table border=1 cellpadding=2 cellspacing=0>";
                        if (count($access)>0) {
                            ?>
                            <tr>
                                <td valign=bottom><font size=-1><b>Resource</b></font></td>
                                <td valign=bottom align=center><font size=-1><b>Signup<br>Permission</b></font></td>
                                <td valign=bottom align=center><font size=-1><b>E-mail<br>Monitor</b></font></td>
                            </tr>
                            <?
                            foreach ($access as $oneitem) {
                                echo "<tr><td><font size=-1>" . $oneitem['long'] . "</font></td><td align=center><font size=-1>";
                                if (!checkpermissions($users[$usn]['resourceblock'],"1",$oneitem['resource'])) echo "X";
                                echo "</font></td><td align=center><font size=-1>";
                                if (checkpermissions($users[$usn]['resourceblock'],"2",$oneitem['resource'])) echo "X";
                                echo "</font></td></tr>";
                            }
                        } else echo "<tr><td>None</td></tr>";
                        echo "</table>";

                }
                echo "</font>";
                echo "</td></tr></table>";
            }
        }

        $usn = $_POST['usn'];
        $sec = $users[$usn]['sec'];

        if ($_POST['submit']=="restrictuser") {
             $users[$usn]['restricted'] = TRUE;
             echo "<br><font color='red'>WARNING:</font>  When a user's account is restricted, existing future signups may be deleted,<br>";
             echo "and restrictions will be imposed upon new signup attempts.<br>";
             echo "<br><b>Are you sure that you want to restrict the user account for <font color='red'>".$users[$usn]['firstname']." ".$users[$usn]['lastname']." (".$users[$usn]['lastname'].")</font>?<br>";
             echo '<a href="' . user_management_prog . '?usn=' . $usn . '&UID=' . $UID . '&submit=confirmrestrictuser"><font size="-1">[Restrict User Account]</font></a>&nbsp;';
             echo '<a href="' . user_management_prog . '?usn=' . $usn . '&UID=' . $UID . '&submit=Cancelusermanagement"><font size="-1">[Cancel]</font></a>&nbsp;';
        }

        if ($_POST['submit']=="confirmrestrictuser")  {
                require_once('maintfunctions.php');
                reconcileexpiredaccounts ($usn);
                $users = getusers();
                $users[$usn]['restricted'] = TRUE;
                logaction('has restricted ',$thisuserid,array('usn'=>$usn,'member'=>$users[$usn]['username']));
                putusers($users);
        }

        if ($_POST['submit']=="unrestrictuser") {
                $users = getusers();
                $users[$usn]['restricted'] = FALSE;
                logaction('has removed restrictions for ',$thisuserid,array('usn'=>$usn,'member'=>$users[$usn]['username']));
                putusers($users);
        }

        if ($_POST['submit']=="activateuser") {
                recordactivity($usn);
                $users = getusers();
                logaction('has reactivated ',$thisuserid,array('usn'=>$usn,'member'=>$users[$usn]['username']));
                echo '<p><b>Inactivity flag has been removed from '.$users[$usn]['firstname'].' '.$users[$usn]['lastname'].' ('.$users[$usn]['lastname'].')</b><p>';
        }

        if ($_POST['submit']=='deleteuser' & !save_deleteduser_signups)
                echo '<p><font color="#990000"><b>Warning: Selected user will be deleted and all of his/her signups will be erased!</b></font><p>';


        if ($_POST['submit'] == "editmember" || $_POST['submit'] == "addeditmember") {
			if ($_POST['submit'] == "editmember") {
                 if ($usn=="") {
                    $usn = $thisuserid;
                    $_POST['usn'] = $usn;
                 }
                 $firstname = $users[$usn]['firstname'];
                 $lastname  = $users[$usn]['lastname'];
                 $username  = $users[$usn]['username'];
                 if ($username == "") $username = strtolower($lastname);
                 $ratings   = $users[$usn]['ratings'];
                 $homephone = $users[$usn]['homephone'];
                 $workphone = $users[$usn]['workphone'];
                 $email1    = $users[$usn]['email1'];
                 $email2    = $users[$usn]['email2'];
                 $resourceblock = $users[$usn]['resourceblock'];
                 $sec           = $users[$usn]['sec'];
                 $expire        = $users[$usn]['expire'];
                 $lastactivity  = str_replace("inactive","",$users[$usn]['lastactivity']);
                 $restricted    = $users[$usn]['restricted'];
            } else {
                 $username  = "";
                 $firstname = "";
                 $lastname  = "";
                 $ratings   = "";
                 $homephone = "";
                 $workphone = "";
                 $email1    = "";
                 $email2    = "";
                 $expire    = "";
                 $lastactivity  = "";
                 $restricted    = "";
                 $sec           = defalut_user_sec;
                 $resourceblock = "b" . str_repeat("1",max(array_keys($resourcelist)));
                 foreach ($resourcelist as $oneitem)  {     //set default access permissions
                      if ($oneitem['defaultaccess']==1) $resourceblock[$oneitem['resource']] = "0";
            	 }
			}

            if ($numerrors > 0) {
            	// For future possibilities, separate administrative validation from plain users
            	if ($administrator) {
            		$firstname = $_POST['firstname'];
            		$lastname  = $_POST['lastname'];
            		$username  = $_POST['username'];
            		if ($username == "") $username = $lastname;
            		$expire    = $_POST['expire'];
                    $resourceblock = "b" . str_repeat("1",max(array_keys($resourcelist)));
                    if (count($_POST['resourceblock'])>0) {
                           foreach ($_POST['resourceblock'] as $value) {
                                //set resource permissions (allow)
                                $resourceblock[$value] = "0";
                            }
                    }
                    if (count($_POST['resourcemonitor'])>0) {
                        foreach ($_POST['resourcemonitor'] as $value) {
                            //set resource monitor permission
                            $temp = $resourceblock[$value] + 2;
                            $resourceblock[$value] = $temp;
                        }
                    }
                    $sec = $_POST['allowsignup']*1 + $_POST['calendar']*10 + $_POST['administrator']*100 + $_POST['emaillogs']*1000;
						
				}
				$ratings   = $_POST['ratings'];
                $homephone = $_POST['homephone'];
                $workphone = $_POST['workphone'];
                $email1    = $_POST['email1'];
                $email2    = $_POST['email2'];
				
				echo "<br><br><font size=\"+1\" color=\"red\"><b>Sorry, there were errors in your submission.</b></font>";
            }

            $fontsize = "-1";

            ?>
            <form name="SignupForm" <? echo 'method="' . submit_method . '" action="' . user_management_prog . '"'; ?> >

            <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >
            <input type="hidden" name="usn" <?php echo "value=\"$usn\""; ?> >
            <table>
            <?

                if ($_POST['submit'] == "editmember") {
                     echo '<input type="hidden" name="mode" value="usermanagement">';
                     ?>
                     <tr bgcolor="#eeeeff"><td><? echo "> > > > >" ?></td><td colspan="2"><b>USER ACCOUNT INFO</b></td></tr>
                     <? if ($administrator) { ?>
                             <tr><td></td><td> Username: </td><td> <input type="text" name="username" size="15" maxlength="50" value="<? echo $username ?>">&nbsp;<? echo $errors['username'] ?></td></tr>
                             <tr><td></td><td> First Name: </td><td> <input type="text" name="firstname" size="15" maxlength="50" value="<? echo $firstname ?>">&nbsp;<? echo $errors['firstname'] ?></td></tr>
                             <tr><td></td><td> Last Name: </td><td> <input type="text" name="lastname" size="15" maxlength="50" value="<? echo $lastname ?>">&nbsp;<? echo $errors['lastname'] ?></td></tr>
                     <? } else { ?>
                             <tr><td></td><td>Username:</td><td> <? echo $username ?> </td></tr>
                             <tr><td></td><td>First Name:</td><td> <? echo $firstname ?> </td></tr>
                             <tr><td></td><td>Last Name:</td><td> <? echo $lastname ?> </td></tr>
                     <? }
                }
                else {  //if adding a new member
                     ?>
                     <input type="hidden" name="mode" value="addmember">
                     <tr bgcolor="#eeeeff"><td><? echo "> > > > >" ?></td><td colspan="2"><b> <? echo "ENTER NEW USER ACCOUNT INFO" ?> </b></td></tr>
                     <tr><td></td><td> <? echo "Username:" ?> </td><td> <input type="text" name="username" size="15" maxlength="50" value="<? echo $username ?>">&nbsp;<? echo $errors['username'] ?></td></tr>
                     <tr><td></td><td> <? echo "First Name:" ?> </td><td> <input type="text" name="firstname" size="15" maxlength="50" value="<? echo $firstname ?>">&nbsp;<? echo $errors['firstname'] ?></td></tr>
                     <tr><td></td><td> <? echo "Last Name:" ?> </td><td> <input type="text" name="lastname" size="15" maxlength="50" value="<? echo $lastname ?>">&nbsp;<? echo $errors['lastname'] ?></td></tr>
                     <?
                }

                // for all account info entry
                ?>
                <tr><td></td><td> <? echo info_fieldname . ": " ?> </td><td><input type="text" name="ratings" size="15" maxlength="50" value="<? echo $ratings ?>">
                <? if (!nouserhide) { ?>
                    (enter "hide" to hide in Member List)
                <? } ?>
                </td></tr>
                <tr><td></td><td> <? echo "Home Phone: " ?> </td><td><input type="text" name="homephone" size="15" maxlength="50" value="<? echo $homephone ?>"></td></tr>
                <tr><td></td><td> <? echo "Work Phone: " ?> </td><td><input type="text" name="workphone" size="15" maxlength="15" value="<? echo $workphone ?>"></td></tr>
                <tr><td></td><td> <? echo "Email 1: " ?> </td><td><input type="text" name="email1" size="50" maxlength="70" value="<? echo $email1 ?>"></td></tr>
                <tr><td></td><td> <? echo "Email 2: " ?> </td><td><input type="text" name="email2" size="50" maxlength="70" value="<? echo $email2 ?>"></td></tr>
                <tr></tr>
                <?
                //if ($_POST['submit'] == "editmember"){
                    ?>
                    <tr bgcolor="#eeeeff"><td><? echo "> > > > >" ?></td><td colspan="2"><b>USER PASSWORD</b></td></tr>
                    <tr><td></td><td>New Password:</td><td> <input type="password" name="newpassword1" size="15" maxlength="50" >&nbsp;<? echo $errors['password'] ?></td></tr>
                    <tr><td></td><td>New Password (again):</td><td> <input type="password" name="newpassword2" size="15" maxlength="50" ></td></tr>
                    <?
                //}

                echo '<tr bgcolor="#eeeeff"><td>> > > > ></td><td colspan="2"><b>ACCOUNT EXPIRATION</b></td></tr>';
                if (!$administrator) {
                    echo "<tr><td></td><td colspan=2>".expire_explanation."</td></tr>";
                    $exptmp = (strcmp($expire, "") == 0 ? "Never" : $expire);
                    echo "<tr><td></td><td>Expiration Date:</td><td>".$exptmp."</td></tr>";
                    echo "<tr><td></td><td>Most recent Activity:</td><td>".(strcmp($lastactivity,"")==0 ? "None" : str_replace("inactive","",$lastactivity))."</td></tr>";
                    echo "<tr><td></td><td>".($restricted == 1 ? '<b>User Account is RESTRICTED</b>' : 'User Account is Unrestricted')."</td></tr>";
                }else {
                    ?><tr><td></td><td> <? echo "Expiration Date: " ?> </td><td><input type="text" name="expire" id="expire" size="15" maxlength="50" value="<? echo $expire ?>">&nbsp;<? echo $errors['expire'] ?></td></tr>
                    <script type="text/javascript">
                        Calendar.setup({
                            inputField     :    "expire",          // id of the input field
                            ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                            weekNumbers    :    false,            //do not show week numbers
                            showOthers     :    true,             //show other months
                        });
                    </script>
                    <?
                    echo "<tr><td></td><td>Most Recent Activity:</td><td>".(strcmp($lastactivity,"")==0 ? "None" : str_replace("inactive","",$lastactivity))."</td></tr>";
                    echo "<tr><td></td><td>Account Restrictions:</td><td>".($restricted == 1 ? '<b>Future signups are Restricted</b>' : 'No Restrictions')."</td></tr>";

                }


                if ($administrator) {
                    ?>
                    </table>
                    <table>
                    <tr bgcolor="#eeeeff"><td><? echo "> > > > >" ?></td><td colspan="2"><b> <? echo "USER PERMISSIONS" ?> </b></td></tr>
                    <tr><td></td><td colspan="2"><input type="checkbox" name="allowsignup" value="1" <? if (checksec($sec,1)) echo "checked"; ?> > User can sign-up resources</td></tr>
                    <tr><td></td><td colspan="2">
                    <? if (($usn == $thisuserid || $usn == 1 ) && checksec($sec,10)) { //don't allow disable if this is ME or Administrator
                        echo '(X)<input type="hidden" name="calendar" value="1">';
                    } else { ?>
                        <input type="checkbox" name="calendar" value="1"  <? if (checksec($sec,10)) echo "checked"; ?> >
                    <? } ?> 
                    User has calendar authority</td></tr>
                    <tr><td></td><td colspan="2">
                    <? if (($usn == $thisuserid || $usn == 1 ) && checksec($sec,100)) { //don't allow disable if this is ME or Administrator
                        echo '(X)<input type="hidden" name="administrator" value="1">';
                    } else { ?>
                        <input type="checkbox" name="administrator" value="1"  <? if (checksec($sec,100)) echo "checked"; ?> > 
                    <? } ?> 
                    User is administrator</td></tr>
                    <tr><td></td><td colspan="2"><input type="checkbox" name="emaillogs" value="1"  <? if (checksec($sec,1000)) echo "checked"; ?> > User receives e-mailed logs</td></tr>
                    <?

                    echo '<tr><td></td><td colspan="2">&nbsp;<b>Permitted Resources:</b><br></tr></td>';
                    echo '<tr><td></td><td>';
                    echo '<table border=1 cellpadding=3 cellspacing=0><tr><td valign=bottom align=center><b>Resource</b></td>';
                    echo "<td valign=bottom align=center><a href=\"javascript:alert('When checked, this user will be allowed to sign up for the specified resource');\"><b>Signup<br>Permission</b></td>";
                    echo "<td align=center valign=bottom><a href=\"javascript:alert('When checked, this user will receive e-mail whenever someone signs up for the specified resource');\"><b>E-mail<br>Monitor</b></a></td></tr>";

                    foreach (sortby($resourcelist,'order') as $oneitem) {
                        ?> <tr><td align=center> <? echo $oneitem['long'] ?> </td>
                        <td align=center><input type="checkbox" name="resourceblock[]" value="<? echo $oneitem['resource'] . "\"";
                        if (!checkpermissions($resourceblock,"1",$oneitem['resource'])) echo " checked";
                        echo "></td>";
                        ?> <td align=center><input type="checkbox" name="resourcemonitor[]" value="<? echo $oneitem['resource'] . "\"";
                        if (checkpermissions($resourceblock,"2",$oneitem['resource'])) echo " checked";
                        echo "></td></tr>\n";
                    }

                    echo "</table>\n";
                    echo '</td></tr>';

                }
                if ($_POST['submit'] == "editmember") {
                    ?><tr><td></td><td colspan="2">
                    <input type="submit" name="submit" value="Update">&nbsp;<input type="submit" name="submit" value="Cancel">
                    <? if (!$administrator) {
                        ?><input type="hidden" name="listmembers" value="1"><?
                    } elseif (!empty($_POST['listmembers'])) {
                        ?><input type="checkbox" name="listmembers" value="1" checked>Return to Member List when done<?
                    } ?>
                    </td></tr>
                    </table><br><?
                } else {
                    ?>
                    <tr><td></td><td colspan="2"><input type="submit" name="submit" value="Add New User">&nbsp;<input type="submit" name="submit" value="Cancel New User"></td></tr>
                    </table><br>
                    <?
                }
        }

        //--main member/user listing
        if (!in_array($_POST['submit'],array("editmember","addeditmember","restrictuser"))){ //main user management menu

                //Called by list_members? return to that now
                if (!empty($_POST['listmembers'])) {
                    $noheader = true;
                    include(list_members_prog);
                    return(true);
                }

                if ($administrator) {
                    echo '<p>';
                    echo '<a href="' . user_management_prog . '?UID=' . $UID . '&submit=addeditmember"><font size="-1">[Add New User]</font></a>';
                    echo '&nbsp;&nbsp;&nbsp;<a href="' . user_management_prog . '?UID=' . $UID . '&submit=bulkaddition"><font size="-1">[Bulk User Addition]</font></a>';
                }
                echo "\n";

                // not admin? show only self in list
                if (!$administrator) $users=array($users[$thisuserid]);

                // delete user command? show only user to delete in list
                if ($_POST['submit'] == "deleteuser" ) $users = array($users[$_POST['usn']]);

                ?>
                <table border=0 cellspacing=0 bgcolor="#ffffff"><tr align="center">
                <td>&nbsp;</td>
                <td nowrap><b><font size="-1">&nbsp;Name (Username)&nbsp;</font></b></td>
                <td colspan=4 nowrap><font size="-1"><b>&nbsp;Permissions<br>(<a href="#key">Key</a>)&nbsp;</b></font></td>
                <td><b><font size="-1">&nbsp;Account&nbsp;<br>&nbsp;Expiration&nbsp;</font></b>
                <td><b><font size="-1">&nbsp;Account&nbsp;<br>&nbsp;Activity&nbsp;</font></b>
                <td><b><font size="-1">&nbsp;Permitted&nbsp;<br>&nbsp;Resources&nbsp;</font></b></td>
                <td><b><font size="-1">&nbsp;Monitoring&nbsp;<br>&nbsp;Resources&nbsp;</font></b></td>
                <td><b>&nbsp;Actions&nbsp;</b></td>
                <?

                $rulelinecount = 0;
                foreach(sortbyname($users) as $user) {

                        $otherforminfo = user_management_prog . '?usn=' . $user['usn'] . '&UID=' . $UID . '&submit=';

                        $rulelinecount++;

                        if ($rulelinecount==3) {
                            $rulelinecount = 0;
                            $linecolor = '#eeeeee';
                        } else {
                            $linecolor = '#ffffff';
                        }

                        echo "<tr align=\"center\" bgcolor=\"" . $linecolor . "\">";

                        echo "<td valign='top' nowrap>";
                        if ($user['usn']>1) {
                            echo '<a href="' . $otherforminfo . 'editmember">';
                        } else { ?>
                                <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=configureprogram#administratoraccountinformation"'; ?> >
                        <? } ?> 
            	        <img src="white.gif" height=4 width=1 border=0><br><img src="pencil.gif" align="top" border=0></a><img scr="white.gif" height=1 width=8 border=0>
            	        <?
                        echo "</td>";
                        echo "<td nowrap align=\"left\" valign=top>";
                        echo "<font size='-1'>";
                        echo $user['lastname'];
                        if ($user['firstname'] != "") echo ", " .$user['firstname'];
                        if ($user['username'] != "") {
                            echo " (" .$user['username'] . ")";
                        } else {
                            echo " (" .$user['lastname'] . ")";
                        }
                        echo "&nbsp;</font></td>";
                        echo "<td valign=top>&nbsp;";
                        echo "<font size='-1'>";
                        if (checksec($user['sec'],1)) {
                            echo "S";
                            if ($user['restricted']) echo "/";
                        }
                        if ($user['restricted']) echo "<b><font color=\"#990000\">R</font></b>";
                        echo "&nbsp;</font></td>";
                        echo "<td valign=top>&nbsp;";
                        echo "<font size='-1'>";
                        if (checksec($user['sec'],10)) echo "C";
                        echo "&nbsp;</font></td>";
                        echo "<td valign=top>&nbsp;";
                        echo "<font size='-1'>";
                        if (checksec($user['sec'],100)) echo "A";
                        echo "&nbsp;</font></td>";
                        echo "<td valign=top>&nbsp;";
                        echo "<font size='-1'>";
                        if (checksec($user['sec'],1000)) echo "E";
                        echo "&nbsp;</font></td>";

                        $expiredate = $user['expire'];
                        if (strcmp($expiredate, "") == 0) $expiredate = "<i>Never</i>";
                        echo "<td valign=top align = \"center\">";
                        echo "<font size='-1'>";
                        echo $expiredate."&nbsp;</font></td>";

                        $lastactivity = $user['lastactivity'];
                        echo "<td valign=top align = \"center\">";
                        echo "<font size='-1'>";
                        if (stristr($lastactivity, "inactive") != FALSE)
                            echo "<i>Inactive</i>";
                        else
                            echo $lastactivity."&nbsp;";
                        echo "</font></td>";

                        echo "<td valign=top align=\"left\"><font size=\"-1\">&nbsp;";
                        $temp = sortby($resourcelist,'order');
                        foreach ($temp as $index=>$oneitem)
                            if (checkpermissions($user['resourceblock'],"0",$oneitem['resource']))
                                echo $oneitem['initials'] . ", ";
                        echo "&nbsp</font></td>";

                        echo "<td valign=top align=\"left\"><font size=\"-1\">&nbsp;";
                        foreach ($temp as $index=>$oneitem)
                            if (checkpermissions($user['resourceblock'],"2",$oneitem['resource']))
                                echo $oneitem['initials'] . ", ";
                        echo "&nbsp</font></td>";

                        echo "<td valign=top align=\"left\">&nbsp;";

                        //Provide action buttons
                        if (($_POST['submit'] == "deleteuser") && ($user['usn'] == $_POST['usn'])) {
                            echo '<a href="' . $otherforminfo . 'confirmdeleteuser"><b>[Confirm Delete]</b></a>&nbsp;';
                            echo '<a href="' . $otherforminfo . 'usermanagement"><font size="-1">[Cancel]</font></a>&nbsp;';
                        } elseif ($user['usn']>1) {
                            echo '<a href="' . $otherforminfo . 'editmember"><font size="-1">[Edit]</font></a>&nbsp;';
                            if ($administrator) { //only admin allowed to delete or change restrictions
                                echo '<a href="' . $otherforminfo . 'deleteuser"><font size="-1">[Delete]</font></a>&nbsp;';
                                if ($user['restricted']){
                                    echo '<a href="' . $otherforminfo . 'unrestrictuser"><font size="-1">[Unrestrict]</font></a>&nbsp;';
                                } else {
                                    echo '<a href="' . $otherforminfo . 'restrictuser"><font size="-1">[Restrict]</font></a>&nbsp;';
                                }
                                if (stristr($user['lastactivity'], "inactive") != FALSE) {
                                    echo '<a href="' . $otherforminfo . 'activateuser"><font size="-1">[Activate]</font></a>&nbsp;';
                                }
                            }
                        } else { //for usn=1 (administrator acct)
                            echo '<a href =';
                            echo '"' . util_prog . '?UID=' . $UID . '&submit=configureprogram#administratoraccountinformation"';
                            echo ' ><font size="-1">[Edit]</font></a>';
                        }
                        echo "&nbsp;</td></tr>\n";
                }
                ?>
                </table>
                <br><font size="-1">
                <a name="key"></a>
                <b>Key:</b><br>
                <li>S = Sign-up Allowed</li>
                <li><b><font color="#990000">R</font></b> = Account is Restricted</li>
                <li>C = Has Calendar Authority</li>
                <li>A = Is Administrator</li>
                <li>E = Receives E-mailed Logs</li>
                </font>
                <?
        }
        ?></form> <?
        break;

}

scheduleunlock();
printfooter();

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
// -adjusted actions available for different types of users
//JMS 1/5/04
// -added support for euro-date
//JMS 3/29/04
// -removed requirement for first name
//JMS 7/29/04
//  -added hide of user via ratings = "hide"
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!