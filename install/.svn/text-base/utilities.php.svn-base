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


if ($calendarauth) {
    switch($_POST['submit']) {
        case "listinfringements" :                //List of users with rule infringements

                echo '<b>Sign-up Infringements:</b>';
                echo "\n";

                echo "<p>The following users have infringements: (click on name to view details)\n";

                echo "<ul>\n";
                foreach ($users as $usn=>$user) {

                        $infringements = validatesignups($usn,false);

                        if (count($infringements)>0) {
                                echo '<li>';
                                echo '<b><a href="' . list_signups_prog . '?usn=' . $usn . '&UID=' . $UID . '&submit=listuser">';
                                echo $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] . "</a></b></li>";
                                echo "<ul>\n";
                                foreach ($infringements as $item) echo "<li><font color=#dd0000>" . $item . "</font></li>\n";
                                echo "</ul>\n";
                        }
                }
                echo "</ul>\n";
    }
}

if ($administrator) {

    switch($_POST['submit']) {

        //- - - - - - -

        case "dailyactions" :

                require_once('maintfunctions.php');
                dailyactions();
                echo '<b>Performed Daily Actions</b>';
                echo "\n<br>";

                       $_POST['submit']="";
                break;

        //- - - - - - -

		case "validateusers" :

                validateusers();
                echo '<b>Validated Users</b>';
                echo "\n<br>";

                       $_POST['submit']="";
                break;

        //- - - - - - -

		case "unlockroot" :

                if (demomode) {echo "Demo mode - function disabled"; break;}
                chmod (root_dir, 0777);
                chmod (root_dir . "../backup/",0777);

                foreach (getfilelist(root_dir,true,true) as $filename) if (file_exists($filename)) @chmod($filename,0777);

                ?>
                <b>Data Folder Access Allowed</b>
                <p>
                <font color="#990000"><b>WARNING:</b> The scheduling data folder has just had it's access permissions set to world read/write.
                 This action can compromise the security of your scheduler and any other web files you have stored on this server. When you are done
                you must use "Lock Data Folder" to restore the security of this system!<p></font>

                <?
                       $_POST['submit']="";
                break;

        //- - - - - - -

		case "lockroot" :

                if (demomode) {echo "Demo mode - function disabled"; break;}
                chmod (root_dir, 0700);
                chmod (root_dir . "../backup/",0700);

                foreach (getfilelist(root_dir,true,true) as $filename) if (file_exists($filename)) @chmod($filename,0700);

                echo '<b>Data Folder Access Disallowed</b>';
                echo "\n<br>";

                       $_POST['submit']="";
                break;

        //- - - - - - -

		case "editannouncements" :

                ?>
                <b>Edit Announcements File:</b><p>

                The contents of the announcement file will be displayed when a user logs in. The current announcement file contains:<p>

                <? if (file_exists(announcementfile_name)) include(announcementfile_name); ?>

                <p>You may modify this file here. You may include HTML and PHP code as desired.
                <form name="announcements" method="post" action=<? echo '"' . util_prog . '"'; ?>>
                  <p>
                    <textarea name="htmltext" cols="80" rows="15"><? if (file_exists(announcementfile_name)) @readfile(announcementfile_name); ?></textarea>
                    <br>
                    <input type="submit" name="submit" value="Submit Announcements">
                    <input type="submit" name="Cancel" value="Cancel">
                    <input type="hidden" name="UID" value="<? echo $UID; ?>">
                  </p>
                  </form>
                <?

                break;

        case "Submit Announcements" :

                echo "<b>Edit Announcements File:</b><p>";

                if (demomode) {echo "Demo mode - saving disabled"; break;}

                $failure = writetextfile(announcementfile_name, array(stripcslashes($_POST['htmltext'])));
                if (!$failure) {
                        echo "Announcements Written Successfully:<p>";
                        include(announcementfile_name);
                } else {
                        ?>
                        <font color="#990000">UNABLE</font> to write Announcements (possible file permissions problem?)
                        <?
                }
                       $_POST['submit']="";
                break;

        //- - - - - - -

        case "timeshiftcorrection" : //correct time shift problems

                ?>
                <b>Time Shift Correction:</b><br>
                <?
                if (demomode) {echo "Demo mode - function disabled"; break;}
                
                require_once('maintfunctions.php');
                timeshiftcorrection();

                break;

        //- - - - - - -

        case "reconcile" : //reconcile calendar with user files

                ?>
                <b>Reconcile Calendar:</b><br>
                <?
                if (demomode) {echo "Demo mode - function disabled"; break;}

                require_once('maintfunctions.php');
                $result = reconcilecalendar();

                if ($result['missing'] > 0) echo "<p><b>Restored " . $result['missing'] . " missing entries</b></p>";
                if ($result['short'] > 0) echo "<p><b>Extended " . $result['short'] . " short entries</b></p>";
                if ($result['missing'] == 0 & $result['short'] == 0)
                        echo "<p>All entries over the next year were reconciled without exception</p>";
                else {
                        echo "<b>Changed users:</b><br>";
                        foreach ($result['changed'] as $user) echo "&nbsp;&nbsp;&nbsp;" . $user . "<br>\n";
                }

                       $_POST['submit']="";

                break;

       //- - - - - - -

        case "protectfile" :
        case "Protect File" :                //button submission

                echo '<b>Protect File:</b>';
                echo "\n<p>";

                if (demomode) {echo "Demo mode - function disabled"; break;}

                echo "This function will protect a file from external access by having the scheduler take ownership.<br>\n";
                echo "Please enter the name of file in the scheduler/data folder you want the scheduler to protect.<br>\n";
                echo "Sub-folder paths are acceptable (e.g. \"1/default.csv\"); Parent folders are not.<br>\n";
                echo "The file <b>must</b> have global write permission.<br>\n";

                $filename = $_POST['filename'];
                if ($_POST['filename']!="") {

                        echo "<br>";

                        $filename = str_replace('..',"",$filename);
                        while (substr($filename,0,1)=='\'' | substr($filename,0,1)=='/')
                                $filename = substr($filename,1);

                        $file1 = root_dir . $filename;
                        $file2 = $file1 . ".tmp";

                        $success = copy($file1, $file2);
                        if ($success) $success = unlink($file1);
                        if ($success) $success = rename($file2 , $file1);
                        if ($success) $success = chmod($file1, 0744);

                        if ($success)         echo "SUCCESS: Ownership of <b>$filename</b> has been taken by the scheduler...<br>\n";
                        else                 echo "<font color=#990000><b>Failed to take possession of $filename...</b></font><br>\n";

                }

                echo "<form name=\"takefile\" method=\"" . submit_method . "\" action=\"" . util_prog . "?submit=protectfile\">";
                echo "<input type=\"hidden\" name=\"UID\" value=\"" . $UID . "\">";
                echo "<input type=\"text\" name=\"filename\" value=\"\">";
                echo "<input type=\"submit\" name=\"submit\" value=\"Protect File\">";

                echo '</form>';

                break;

        //- - - - - - -
        case "resetconfig" :
        case "configureprogram" :
        case "Submitconfigureprogram" :

                if ($_POST['submit']=="resetconfig")
                    echo "<br><b><font color=\"#990000\">You have reset the values for the selected field(s). However, this configuration must be saved before the reset values are saved.</font></b><p>";

                $configpw = our_crypt(strtolower($_POST['ConfigPassword']));
                $validpw  = (configpassword == "" | strcasecmp($configpw,configpassword) == 0 | strcasecmp($configpw,our_crypt(configpassword)) == 0);


                if (!$validpw) {
                         if ($_POST['ConfigPassword']!="") echo "<p><b><font color=\"#990000\">The password you entered is incorrect...</font></b>";
                         $_POST['ConfigPassword'] = "";
                }
                //if there is a password and we haven't gotten it, prompt for it
                if (strcasecmp($_POST['ConfigPassword'],"") == 0 & strcasecmp(configpassword,"") != 0) {
                   echo "<form name=\"configpass\" method=\"" . submit_method . "\" action=\"" . util_prog . "?submit=configureprogram\">";
                   echo "<br>";
                   echo "<b>Please Enter the Configuration settings password:</b>";
                   echo "<input type=\"password\" name=\"ConfigPassword\" value=\"\">";
                   echo "<input type=\"hidden\" name=\"UID\" value=\"" . $UID . "\">";
                   echo "<input type=\"hidden\" name=\"mode\" value=\"configureprogram\">";
                   echo "<input type=\"submit\" name=\"submit\" value=\"Submit\">";
                   echo "</form>";
                }

                if ($validpw | demomode) {

                        //special request to force configuration page open?
                        if ($administrator & $_POST['forceconfig']=='AskedByInstall' & !demomode) {

                                ?>
                                <h3><font color="#990000">Final Installation Step:</font> After reviewing configuration settings, you must choose "Save Changes" to finish the installation process.</h3>
                                <?
                        } else {
                                echo '<font size=+1><b>Program Configuration:</b></font>';
                                echo "\n<p>";
                        }


                    if (demomode & !$validpw)
                        echo "<b>DEMO MODE:</b> Configuration changes disabled - enter password above to make changes.<p>";
                    else {
                        echo "<table border=1 cellspacing=0 cellpadding=4><tr><td>";
                        echo "You may modify the program configuration and signup rules here.<br>\n";
                        echo "After you make configuration changes, make sure to select 'Save Changes'.<br>\n";
                        echo "Be careful.  Do not edit these values unless you know what you are doing!\n";
                        echo "</td></tr></table>";
                    }

                    if ($validpw)
                        $otherforminfo = util_prog . '?UID=' . $UID . '&ConfigPassword=' . $_POST['ConfigPassword'] . '&submit=resetconfig&defaultname[]=';
                    else
                        $otherforminfo = util_prog . '?UID=' . $UID . '&submit=resetconfig&defaultname[]=';

                    if (count($_POST['defaultname'])==0) $_POST['defaultname'] = array();
                    //add all the "defaultname" names together at end of otherforminfo
                    foreach ($_POST['defaultname'] as $item) $otherforminfo .= $item . '&defaultname[]=';


                    echo "<form name=\"Update Configuration\" method=\"" . submit_method . "\" action=\"" . util_prog . "?submit=SaveChanges\">";

                    $configbuffer = readtextfile(config_file);
        
                    if ($validpw)
                        echo "<input type=\"submit\" name=\"submit\" value=\"Save Changes\"><br>";

                    echo "<p><a name='#top'></a><b>Topics</b>";

                    echo "<ul>\n";
                    foreach ($configbuffer as $line) {
                        $line = trim($line);
                        if (strstr($line, "****") != FALSE) {
                            $item = substr($line,4,strlen($line)-9);
                            echo "<li><a href=\" #" . strtolower(str_replace(' ','',$item)) . "\">" . $item . "</a><br>\n";
                        }
                        if (substr($line,0,1) == "-") {
                            $item = substr($line,1,strlen($line)-2);
                            echo "<ul><li><a href=\" #" . strtolower(str_replace(' ','',$item)) . "\">" . $item . "</a></ul>\n";
                        }
                    }
                    echo "</ul>\n";
                    
                    $i=0;
                    echo "<table border=\"0\" cellspacing=\"2\" cellpadding=\"2\">";
                    foreach ($configbuffer as $line4) {
                        echo "<tr>";
                        if (strstr($line4, "****") == FALSE) {
                            $line4 = str_replace("&,", "&@", $line4);
                            unset($linesplit);
                            $linesplit = explode(",", $line4);
                                if (count($linesplit) > 1) {
                                    foreach ($linesplit as $ind=>$linetmp) $linesplit[$ind]=str_replace("&@", ",", $linetmp);
                                    if (count($linesplit) == 3) $linesplit[3] = ".\n";

                                    if (substr(trim($linesplit[1]),0,1)=="n") {
                                        //make sure we've expanded any multipliers (comes from the default values)
                                        $defnum = explode("*", $linesplit[2]);
                                        $nval=1;
                                        foreach ($defnum as $defnum2) $nval = $nval*$defnum2;
                                        $linesplit[2] = $nval;
                                    }
                                    if (defined($linesplit[0])) {
                                        switch (trim($linesplit[1])) {
                                            case "b" :
                                                $wasset = (($linesplit[2]=="True" && constant($linesplit[0])==False) || ($linesplit[2]=="False" && constant($linesplit[0])==True));
                                                break;
                                            case "t" :  //same as p
                                            case "p" :
                                                $wasset = ($linesplit[2] != constant($linesplit[0]));
                                                break;
                                            default :
                                                $wasset = (0+$linesplit[2] != 0+constant($linesplit[0]));
                                                break;
                                        }
                                        if (!in_array($linesplit[0],$_POST['defaultname'])) {
                                                $linesplit[2] = constant($linesplit[0]);  //get constant value defined
                                        } else $wasset = false;
                                    } else $wasset = false;

                                    echo "<td valign=top bgcolor=\"#eeeeff\"><font size=\"-1\">[&nbsp;".$linesplit[0]."&nbsp;]</font></td>";
                                    switch (trim($linesplit[1])) {
                                    case "b" :
                                            echo "<td valign=top bgcolor=\"#eeeeee\">".$linesplit[3];
                                            if ($wasset) echo "<a href=\"" . $otherforminfo . $linesplit[0] . "\"><font size=\"-1\">[reset]</font></a>";
                                            echo "</td>";
                                            echo "<td valign=top bgcolor=\"#eeeeee\">";
                                            echo "<select name=\"config[]\">";
                                            echo "<option value=\"True\"";
                                            if ($linesplit[2] | (strcasecmp(trim($linesplit[2]),'true')==0)) echo " selected";
                                            echo ">True</option>";
                                            echo "<option value=\"False\"";
                                            if (!$linesplit[2] | (strcasecmp(trim($linesplit[2]),'false')==0)) echo " selected";
                                            echo ">False</option>";
                                            echo "</select>";
                                            echo "</td>";
                                            break;
                                    case "t" :
                                            echo "<td colspan=2 valign=top  bgcolor=\"#eeeeee\">";
                                            echo "<table border=0><tr>";
                                            echo "<td valign=top bgcolor=\"#eeeeee\">";
                                            echo "".$linesplit[3];
                                            if ($wasset) echo "<a href=\"" . $otherforminfo . $linesplit[0] . "\"><font size=\"-1\">[reset]</font></a>";
                                            echo "</td>";
                                            echo "<td valign=top>";
                                            echo "<textarea name=\"config[]\" cols=40>".trim($linesplit[2])."</textarea>";
                                            echo "</td>";
                                            echo "</tr></table>\n";
                                            echo "</td>";
                                            break;
                                    case "p" :
                                            echo "<td valign=top bgcolor=\"#eeeeee\">".$linesplit[3]." <b>Enter twice to confirm</b>";
                                            if ($wasset) echo "<a href=\"" . $otherforminfo . $linesplit[0] . "\"><font size=\"-1\">[reset]</font></a>";
                                            echo "</td>";
                                            echo "<td valign=top bgcolor=\"#eeeeee\">";
                                            echo "<input type=\"password\" name=\"config[]\" value=\"".trim($linesplit[2])."\">";
                                            echo "<input type=\"password\" name=\"passwd[" . $i . "]\" value=\"".trim($linesplit[2])."\">";
                                            echo "</td>";
                                            break;
                                    default :
                                        //make sure we've expanded any multipliers (comes from the default values)
                                        $defnum = explode("*", $linesplit[2]);
                                        $nval=1;
                                        foreach ($defnum as $defnum2) $nval = $nval*$defnum2;
                                        $linesplit[2] = $nval;

                                            //Translate #s down using seconds,minutes,hours correction
                                            $strng = "";
                                            $invert = False;
                                            $units = 0;  //seconds
                                            if ($linesplit[2] < 0) {
                                                $invert = True;
                                                $linesplit[2] = -$linesplit[2];
                                            }
                                            if ($linesplit[2] > 60 & ($linesplit[2]%60) == 0) {
                                                $strng = $strng . "*60";
                                                $linesplit[2] = $linesplit[2]/60;
                                                $units = 1;  //minutes
                                            }
                                            if ($linesplit[2] > 60 & ($linesplit[2]%60) == 0) {
                                                $strng = $strng . "*60";
                                                $linesplit[2] = $linesplit[2]/60;
                                                $units = 2;  //hours
                                            }
                                            if ($linesplit[2] > 24 & ($linesplit[2]%24) == 0) {
                                                $strng = $strng . "*24";
                                                $linesplit[2] = $linesplit[2]/24;
                                                $units = 3;  //days
                                            }

                                            echo "<td valign=top bgcolor=\"#eeeeee\">".$linesplit[3];
                                            if ($wasset) echo "<a href=\"" . $otherforminfo . $linesplit[0] . "\"><font size=\"-1\">[reset]</font></a>";
                                            echo "</td>";
                                            echo "<td valign=top nowrap bgcolor=\"#eeeeee\">";

                                            if (trim($linesplit[1])=='nt') { //numeric time
                                                $strng = $linesplit[2];
                                                if ($invert) $strng = "-" . $strng;
                                                $linesplit[2] = $strng;

                                                echo "<input type=\"text\" size=4 name=\"config[]\" value=\"".trim($linesplit[2])."\">";
                                                ?>
                                                <select name="units[]">
                                                    <option value=""          <? if ($units==0) echo "selected"; ?>>Seconds</option>
                                                    <option value="*60"       <? if ($units==1) echo "selected"; ?>>Minutes</option>
                                                    <option value="*60*60"    <? if ($units==2) echo "selected"; ?>>Hours</option>
                                                    <option value="*24*60*60" <? if ($units==3) echo "selected"; ?>>Days</option>
                                                </select>
                                                <?

                                            } else {
                                                
                                                $strng = $linesplit[2] . $strng;
                                                if ($invert) $strng = "-" . $strng;
                                                $linesplit[2] = $strng;

                                                echo "<input type=\"text\" name=\"config[]\" value=\"".trim($linesplit[2])."\">";
                                                
                                            }    
                                            echo "</td>";
                                            break;
                                    }
                                    echo "</tr>\n";

                                    $i++;
                                } else {
                                    $line4 = trim($line4);
                                    echo "<td colspan='3' bgcolor=\"#ffffff\">\n";
                                    echo "<a name=\"" . strtolower(str_replace(' ','',substr($line4,1,strlen($line4)-2))) . "\"></a>\n";
                                    echo "<b>" . $line4 . "</b>";
                                    echo "&nbsp;";
                                    if (strlen($line4)>0) {
                                        echo "&nbsp;&nbsp;&nbsp;&nbsp;<font size=-1><a href='#top'>[back to topic list]</a></font>";
                                        if ($validpw)
                                            echo "&nbsp;&nbsp;&nbsp;<input type=\"submit\" name=\"submit\" value=\"Save Changes\">";
                                    }
                                    echo "</td></tr>\n";
                                }
                           } else {
                                $line4 = trim($line4);
                                echo "<td colspan='3' bgcolor=\"#ffffff\" align=\"right\">\n";
                                if ($validpw)
                                    echo "<input type=\"submit\" name=\"submit\" value=\"Save Changes\">";
                                echo "</td></tr>\n";
                                echo "<tr><td colspan='3' bgcolor=\"#ffffff\">\n";
                                echo "<hr>";
                                echo "<a name=\"" . strtolower(str_replace(' ','',substr($line4,4,strlen($line4)-9))) . "\"></a>\n";
                                echo "<font size=+1><br><b>".$line4."</b></font>";
                                echo "&nbsp;&nbsp;&nbsp;&nbsp;<font size=-1><a href='#top'>[back to topic list]</a></font>";
                                if ($validpw)
                                    echo "&nbsp;&nbsp;&nbsp;<input type=\"submit\" name=\"submit\" value=\"Save Changes\">";
                                echo "</td>";
                           }
                    }
                    echo "</table><br>";
                    if ($validpw) {
                        echo "<input type=\"hidden\" name=\"ConfigPassword\" value=\"".$_POST['ConfigPassword']."\">";
                        echo "<input type=\"hidden\" name=\"UID\" value=\"" . $UID . "\">";
                        echo "<input type=\"submit\" name=\"submit\" value=\"Save Changes\">";
                    }
                    echo "</form>";

               }
				break;

        //- - - - - - -
        case "Save Changes" :
        case "SaveChanges" :

              $newvalues = $_POST['config'];
              $configbuffer = readtextfile(config_file);
              $i = 0;

              $fd = fopen (config_encoded, "w+");
              fputs($fd, "<" . "?php\n");

              $units = $_POST['units'];

              foreach ($configbuffer as $line1) {

                  if (strstr($line1, "****") == FALSE) {
                       $line1 = str_replace("&,", "&@", $line1);

                       $linesplit = explode(",", $line1);
                       if (count($linesplit) == 3) $linesplit[3] = ".\n";
                       foreach ($linesplit as $ind=>$linetmp) $linesplit[$ind]=str_replace("&@", "&,", $linetmp);
                       if (count($linesplit) >= 3) {
                          $newvalue     = $newvalues[$i];
                          $newvalue     = str_replace("****","",$newvalue);  //don't let user have field with four stars (makes line a comment!!)
                          $newvalue     = str_replace("\r\n","",$newvalue);

                          if (strcasecmp($linesplit[1], "p")==0) {
                                if ($_POST['passwd'][$i]!=$newvalues[$i]) {
                                        $newvalue = constant($linesplit[0]);  //revert back to current password if this one didn't match
                                        echo "<font color=\"#990000\">Warning:</font> Passwords for " . $linesplit[0] . " did not match. Old value retained.<br>";
                                } elseif ($newvalue != sanitize(constant($linesplit[0]))) {
                                        $newvalue = our_crypt(strtolower($newvalue));        //encrypt if new password
                                } else {
                                        $newvalue = constant($linesplit[0]);
                                }
                          }

                          if ($linesplit[1]=="b" && $newvalue=="") $newvalue = "False";

                          //test for out-of-limits values for numerical field
                          if ($linesplit[1]=="n" || $linesplit[1]=="nt") {
                            if ($newvalue=="") $newvalue = "0";
                            if ($linesplit[1]=="nt") {  //numerical time - grab units to go with it
                                list($key,$value) = each($units);
                                $newvalue .= $value;
                            }
                            $defnum = explode("*", $newvalue);
                            $nval=1;
                            foreach ($defnum as $defnum2) $nval = $nval*$defnum2;
                            if ($linesplit[4]!="") {
                                $defnum = explode("*", $linesplit[4]);
                                $mltply=1;
                                foreach ($defnum as $defnum2) $mltply = $mltply*$defnum2;
                                if ($nval<$mltply) $newvalue = $linesplit[4];
                            }
                            if ($linesplit[5]!="") {
                                $defnum = explode("*", $linesplit[5]);
                                $mltply=1;
                                foreach ($defnum as $defnum2) $mltply = $mltply*$defnum2;
                                if ($nval>$mltply) $newvalue = $linesplit[5];
                            }

                            //test for bad timezone entry (special case)
                            if ($linesplit[0]=="timezone") {
                                if ($newvalue>0 && $newvalue<25) //gave timezone in HOURS!
                                    $newvalue = $newvalue*60*60; //convert to MINUTES
                            }

                          }


                          if ($linesplit[1]=="t" | $linesplit[1]=="p")
                                $strng = 'define("'. $linesplit[0] . '","' . $newvalue . "\");\n";
                          else
                                $strng = 'define("'. $linesplit[0] . '",' . $newvalue . ");\n";

                          fputs($fd, $strng);

                          $i++;
                       }
                  }

              }
              fputs($fd, "?" . ">\n");
              fclose ($fd);
              logaction("Updated Configuration",$thisuserid);
              echo "<br><font size=+1 color='#990000'><b>Configuration Updated</b></font><br>";
              echo "Please note that some formatting and feature enable/disable changes may not take effect until the next menu selection <a href=\"" . util_prog . '?UID=' . $UID . "\"><em>(click here to refresh now)</em></a>.<p>";
              $_POST['submit']="";
              //break;  //commented out to force fall-through to next section


        //- - - - - - -

        case "more"  :
        case ""      :

                echo '<br><b>Other Maintenance Actions:</b>';
                //echo "<p>\n";

                ?>
                <table width="750" border="0" cellspacing="2" cellpadding="2">
                <td valign=top bgcolor="#eeeeff">
                <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=editannouncements"'; ?> >
                <font color="#000066">Edit Announcements</font></a></td><td valign=top bgcolor="#eeeeee"> - Modify general announcements displayed when users log in. </td>
                <tr></tr><td valign=top bgcolor="#eeeeff">
                <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=configureprogram"'; ?> >
                <font color="#000066">Program Configuration</font></a></td><td valign=top bgcolor="#eeeeee"> - Modify program settings. May require password. </td>

                <tr></tr><td valign=top bgcolor="#eeeeff">
                <a href = <? echo '"' . log_util_prog . '?UID=' . $UID . '&submit=reviewuselog"'; ?> >
                <font color="#000066">Review Use Log</font></a>
                <br><a href = <? echo '"' . log_util_prog . '?UID=' . $UID . '&submit=reviewactionlog"'; ?> >
                <font color="#000066">Review Action Log</font></a></td>
                <td valign=top bgcolor="#eeeeee"> - Show history of logins and currently logged-in users.
                <br> - Show history of actions and events. </td>

                <tr></tr><td valign=top bgcolor="#eeeeff">
                <a href = <? echo '"' . backup_util_prog . '?UID=' . $UID . '&submit=backup"'; ?> >
                <font color="#000066">Make Backup File</font></a>
                <br><a href = <? echo '"' . backup_util_prog . '?UID=' . $UID . '&submit=restore"'; ?> >
                <font color="#000066">Restore from Backup</font></a></td>
                <td valign=top bgcolor="#eeeeee"> - to be used for emergency restoration of schedule.
                <br> - to be used when restoring from emergency backup.</td>

                <tr></tr><td valign=top bgcolor="#eeeeff">
                <a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=unlockroot"'; ?> >
                <font color="#000066">Unlock Data Folder</font></a>
                <br><a href = <? echo '"' . util_prog . '?UID=' . $UID . '&submit=lockroot"'; ?> >
                <font color="#000066">Lock Data Folder</font></a>
                <td valign=top bgcolor="#eeeeee"> - allow read and write access to data folder (NOTE: Use with caution! Relock when done).
                <br> - disallow read and write access to data folder.</td>

                <font color="#ffffff">&nbsp;validateusers, reconcile, dailyactions, protectfile, timeshiftcorrection&nbsp;</font>
                </table>
                <?

        }
}

scheduleunlock();
printfooter();

//----------------------------------------------------------------------
// JMS 2/14/02 -modified usermenu call in listusersignups to include only sign-up enabled users
//   -sort signup lists (resource or user) by starttime, changed format of lists slightly
// JMS 2/19/02
//   -reload authlist after removing authorized user (in case we created "abandoned" user)
//JMS 2/21/02
// -sanitize ALL inputs (not just comment), added better get support w/odd comment strings
// -added full stripcslashes support for comments
//JMS 3/8/02
// -added memberlist
//JMS/CBW 7/23/02
// -add optional disable of display of memberlist
// -add resource blocking support (users not allowed to sign up for ceratin resources)
// -fix bug causing warning messages when no-resources were allowed for a user
// -add minimum password length test (and option)
// -add password case insensitivity
// -add password encryption for on-line memberlist control
// -add default settings for resource blocking (new users)
// -fix new-user sec settings (on-line memberlist control only)
// -add how-to help for resource blocking controls
// -add copy of resource access (i.e. individual user blocking settings) from one resource to another
// CBW  8/4/02
//  -added logic to reesource list/edit screens to handle two new fields:
//      maxsignup (defined in hours) & doublebook (boolean)
//  -added GUI interface to change program settings (used to be stored in
//      in the defaults file).  Settings are now stored in defaults & in
//      the config.php file (located in the data directory).
// JMS 8/11/02
//  -added better support for different types of default inputs (boolean pull-down menus, text boxes)
//  -renamed "SuperUser Access Password" to simply "Configuration Password"
// JMS 8/27/02
//  -fixed bugs associated w/empty config password
//  -reformatted configuration page slightly
// JMS 10/20/02
//  -revised configuration method to save to new config_enc file
//  -added reconcile calendar
// CBW 11/06/02
//  -added "Search by date" function to List Resource Signups
//  -Included "edit password" boxes to main user account edit screen
//JMS 11/12/02
// -Changed "access code" to "password"
// -fixed bug which hides header_image if no "exit_URL" was set
// -revised search by date form and functionality
//JMS 11/14/02
// -added complete am/pm clock support
// -removed nowrap from memberlist ratings column
// -added special install support to add .htaccess file and force open configuration settings (w/o password)
//  (remembers those special instructions if admin password wasn't correct so that it will still be brought up when
//   the admin does manage to log in)
// CBW 11/17/02
// -fixed bug when clearing an account expiration date
// -edited user account actions menu and added "expire" field to account listing
// -added "Deny public view" field to resource setup
// -added functionality for user account expiration
// CBW 11/23/02
// -added ability to force restrictions on an account (same actions as expired account).
//        This is typically used for inactive accounts.
// JMS 12/02/02
// -added set_time_limit(0) inside restore loop to avoid timeouts (won't work in SAFE mode, though!)
// -added include('maintfunctions.php') just prior to the call to reconcileexpiredaccounts
// -added &nbsp;s in user management table for appearance
// -modified user management table code to use ONE call to lookupmemindex for speed
// -limited [restrict] to only administrators (missing {} allowed it for everyone!)
// JMS 1/9/02
// -keep backup folder unlocked through confirmation (until actual restore is being performed)
// -fixed odd behavior with restricting users (last user appears restricted)
// -modified method of retreiving past signups for "show resources"
//CBW 1/12/03
// -fixed bug with "resourceblock" field in userlist.csv file which used to
//      be limited to 15 resources.  The integer field was changed to a
//      "b-format" string which should be limitless (within reason)
//JMS 2/23/03
// -added schedulelock to stop request (no lock available)
// -updated configuration reset notice
// -added "view backup" option on restore (prior to restore)
// -modifed action_log and action_log details for improved readability
//JMS 3/2/03
// -added forceconfig to call validateusers (used when installing)
// -added (administrator) and (calendar authority) tags to User: line in signup form
// -allow cal authority users to view "signup infringements"
// -added "edit announcements" utility
// -added "announcementfile_name" to items to be backed up and restored
// -added "must log in as admin" note when auto-login after install fails
//CBW 3/6/03
// -fixed blockresource bug
//JMS 3/14/03
// -modified display of program configuration page
// -fixed [reset to default] logic for boolean's in program configuration page
// -show announcement file contents as interpreted "include" when editing
// -return to "Other Actions" menu when certain functions are complete
// -added unlockroot and lockroot to other actions menu
//JMS 3/31/03
// -added demo-mode support
// -fixed security hole in configuration settings password (encryption bug)
//CBW 4/6/03
// -modified resource list to show single resource when editing
// -fixed add resource bug
// -updated appearance of the MORE utilities menu
// -added show_admin_infringe config setting to allow admin infringements
//     to be omitted from the infringement listing
//JMS 4/13/03
// -changed resource permissions editing to be checkbox style (instead of list)
// -revised "user updated" output (shortened and neatened up)
// -removed redundant "set permissions" code in update user
// -added resource monitoring
// -re-fixed add resource bug (force permissions either way (yes OR no) - used to be only on NO)
//JMS 4/25/03
// -added IP to userlog display
// -added configurable color for login form and top menus
// -unbolded Other Actions menu
// -added default color option (for better display when updating to new configurable color version)
// -fixed demo-mode configuration change (e.g. on install) complexity
// -fixed potential santizied password problem in configuration settings (encrypted password might contain
//   characters which get santizied out, thus making the encrypted password look different from the stored
//   password and change it to something unreachable)
//CBW 5/7/03
// -Fixed bug when updating user accounts (resource permissions)
//JMS 5/19/03
// -changed \limit to /limit when writing .htaccess file
// -changed resourceblock conversion (from old files) on add of resource to make "unspecified" resources
//   be permitted (used to make them "blocked" but an unspecified bit really means "permitted"!)
// -modified resource manamgement to avoid changes to resource monitor settings.
// -modified copy resource permissions settings as to copy BOTH resource blocking and resource monitoring features
// -fixed authorized user editing bug (changes were not saved to user's profile)
// -added default of "me" when admin requests edituser without a usn specified
//JAT 7/23/03
// - added requirement of first and last name when adding or editing a user
// - moved expiration date validation closer to form entry code so that users cannot be added or
//   modified until a valid expiration date is entered
// - if invalid data is entered in add/edit user form, user is sent back to form and the information they
//   entered is preserved (except user permissions and permitted resources)
//JAT 7/25/03
// - fixed bug that prevented user name and password from being changed simultaneously
// - fixed bug that prevented user name and permissions from being changed simultaneously
// - added preservation of user permissions and permitted resources form fields when user add/edit form
//   is reloaded due to an error (e.g. omission of first/last name)
//JMS 7/30/03
// - translate empty inputs for numerical configuration values to 0 and for boolean values to "False"
//JMS 8/2/03
// - added limits to be set on numerical configuration values (set in config.php)
// - added test for non-existant announcements file
//JMS 8/20/03
// - revised "ratings" to be user-definable fieldname (internally it is still "ratings")
//JMS 10/1/03
// - used include_once for all instances of include of maintfunctions
//JMS 10/4/03
// - moved expire, restricted and lastactivity into users from memberlist
// - replaced all tests for admin or calendarauth with variables set at start ($administrator and
//    $calendarauth) to make it easier to read and code.
// - clearified logic which tests if administrator's name is being changed (not allowed)
// - fixed bug which caused [EOF]s to be displayed when 
// - fixed restore bug which causes use of memory copies of files instead of disk copies
//JAT 10/12/03
// - initial release of e-mail announcement code
//JMS 11/1/03
// - modified announcement code to avoid warnings with new PHP version
// - modified announcement code use of "order" (replaced with "resource")
// - changed display format for announcement receipients
// - cleaned out extra memberlookup calls in user maintenance
// - fixed usermaintenance bug with restriction/unrestriction
//JMS 11/2/03
// - added billformat option to list by resource (includes late deletions)
//JAT 11/15/03
// - moved various utilities out of base utilities folder and added calls to generic menu function
//JMS 11/17/03
// -re-secured access logic
//JMS 2/6/04
// -added hyperlinks and change format of configuration page
//JMS 2/7/04
// -fixed config page update problem after "update configuration"
//JMS 2/19/04
// -removed .htacess logic (moved to upgrade.php)
//JMS 2/20/04
// -revised unlock/lock to use getfilelist
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
