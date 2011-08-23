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

//----------------------------------------------------
// DAILYACTIONS
//Do actions we need to do only daily
function dailyactions (){

	purgeActionLog();	// Purge old action log entries first due to next step being memory-intensive
    //Check if the last backup mailing was within mail_backup_interval
    $revitems    = array_reverse(readactionlog());
    $lastmailing = "1/1/80";        //default date if we find none
    foreach ($revitems as $oneitem)
        if ($oneitem['data']['action'] == "Backup Mailed") {
            $lastmailing = $oneitem['date'];
            break;
        }
    if ((strtotime(date("m/d/y",max(0,validatedate($lastmailing))))+backup_interval) <= strtotime(date("m/d/y",zulutime()+timezone))) {
        mailallsignups();                //e-mail signups as backup
    }

    purgecalendar();                                //purge calendar of really old days

    reconcileexpiredaccounts(); //identify expired accounts and restrict their signups

}

//----------------------------------------------------
// PURGEACTIONLOG
// Purge stale entries from the action log
function purgeActionLog() {
	if (file_exists(actionlog_name)) {
		$fp = fopenlock(actionlog_name, 'r+');
		
		// First, find the point at which we reach the entries we want to keep
		$currentStart = 0;
		while ($oneLine = fgetcsv_trim($fp, 1000, ','))  {
		    $thisdate = validatedate($oneLine[1]);
		    if ($thisdate<0) $thisdate = 0;
			if ((strtotime(date("m/d/y",$thisdate))+loglength) < strtotime(date("m/d/y",zulutime()+timezone))) {
				$currentStart = ftell($fp);
			}
		}
		
		// Overwrite the new entries with the old ones (256KB at a time to conserve memory)
		$readPointer = $currentStart;
		$writePointer = 0;
		do {
			fseek ($fp, $readPointer);
			$newData = fread($fp, 262144);
			$readPointer = ftell($fp);
			if (strlen($newData) == 0) {
				break;
			}
			fseek ($fp, $writePointer);
			fwrite($fp, $newData);
			$writePointer = ftell($fp);
		} while (true);
		
		// Truncate the file to the proper length and close the file
		rewind($fp);
		ftruncate($fp, filesize(actionlog_name)-$currentStart);
                fclose ($fp);
	}
}

//----------------------------------------------------
// PURGECALENDAR
//Purge all resouce dirs from really old day files
function purgecalendar (){

        $resourcelist = getresources();
        if (count($resourcelist)>0) foreach ($resourcelist as $resource=>$desc)
                for ($day=0; $day<30; $day++) {                //clear calendar from "calendar_history" days ago back 30 days
                        $datestr = date("mdy", zulutime()+timezone-($day*60*60*24)-calendar_history);
                        $onefile = root_dir . $resource . "/" . $datestr . ".csv";
                        if (file_exists($onefile)) unlink($onefile);
                }
        logaction ("Calendar Purged");
}
//----------------------------------------------------
// RECONCILECALENDAR
// verify that all future calendar entries have matching user entries
function reconcilecalendar (){

    $resourcelist = getresources();
    $users        = getusers();

    $missing = 0;
    $short   = 0;
    $changed = array();

    $today   = strtotime(date("m/d/y",zulutime()+timezone));

    for ($day=0; $day<365*5; $day++) {
        $mydate = $today + $day*60*60*24;
        foreach ($resourcelist as $res=>$resource) {
            $signups = getdaysignups ($mydate,$res);
            if (count($signups)>0) foreach ($signups as $signup) {
                $useritems  = getusersignups($signup['usn']);
                $thissignup = $useritems[$signup['signup']];

                $starttime = strtotime(date('m/d/y',$mydate) . ' ' . floor($signup['hour']) . ':' . ((($signup['hour']+$signup['length'])%1)*3) . "0");
                $endtime   = strtotime(date('m/d/y',$mydate) . ' ' . floor($signup['hour']) . ':' . ((($signup['hour']+$signup['length'])%1)*3) . "0")+$signup['length']*60*60;

                if ($endtime > zulutime()+timezone) {        //if the endtime hasn't passed already
                    //is this signup completely missing in the user file? create it.
                    if ($thissignup == "") {

                        $newinfo = array(
                                'signup'    => $signup['signup'],
                                'starttime' => $starttime,
                                'resource'  => $res,
                                'endtime'   => $endtime,
                                'comment'   => "" );

                        $useritems[$signup['signup']] = $newinfo;
                        putusersignups ($signup['usn'],$useritems);
                        logaction("Recovered Signup","",$newinfo);
                        $changed[] = $users[$signup['usn']]['firstname'] . " " . $users[$signup['usn']]['lastname'];
                        $missing++;

                    }
                    //is this a continuation of the same signup? (maybe?!? Resource # must match, calendar-indicated endtime must be > current endtime,
                    //  and calendar-indicated starttime must be > current starttime)
                    if ($thissignup != "" & $thissignup['endtime'] < $endtime
                        & $thissignup['starttime'] < $starttime & $thissignup['resource']==$res) {

                        $useritems[$signup['signup']]['endtime'] = $endtime;        //update the endtime only
                        putusersignups ($signup['usn'],$useritems);
                        logaction("Combined Signups","",$useritems[$signup['signup']]);
                        $changed[] = $users[$signup['usn']]['firstname'] . " " . $users[$signup['usn']]['lastname'];
                        $short++;

    }   }   }   }   }

    return(array("missing"=>$missing,"short"=>$short,"changed"=>$changed));

}

//----------------------------------------------------
// DECODEALLSIGNUPS
//creates a plain-text string representing all signups
// input $mode indicates type of text:
//     "normal" (default) human-readable
//     "php"  machine-interpretable for restoration of lost data
function decodeallsignups($mode="normal") {

        $resourcelist = getresources();                        //get list of valid resources
        $users        = getusers();
        $strng        = "";
        if ($mode!="normal") {
            $strng  = "<?php\n";
            $strng .= "validateusers();\n";
            $strng .= '$items=array();' . "\n";
        }

        foreach ($resourcelist as $resource=>$resourceinfo) {
            $allsignups   = array();
            foreach ($users as $usn => $user) {              //get future signups for all users
                $signup = getusersignups ($usn,true);
                if (count($signup)>0)                        //and grab those for this resource
                    foreach ($signup as $item) {
                        $item['usn'] = $usn;
                        if ($item['resource'] == $resource) $allsignups[] = $item;
                    }
            }

            if ($mode=="normal") $strng .= "Future signups for " . $resourceinfo['long'] . ":\n\n";

            if (count($allsignups) > 0) {
                $allsignups = sortby($allsignups,'starttime');
                foreach ($allsignups as $item) {

                    if ($mode=="normal")
                        //normal text encoding
                        $strng .= decodesignup($item,$users[$item['usn']]['firstname'] . " " . $users[$item['usn']]['lastname'],false);
                    else {
                        //PHP code encoding
                        $strng .= '$items[]=array(';
                        foreach($item as $key=>$value) {
                            $strng .= '"' . $key . '"';
                            if (gettype($value)=="string") $strng .= '=>"' . $value . '",';
                            else $strng .= '=>' . $value . ',';
                        }
                        $strng .= '"priority"=>' . getsignuppriority ($item['usn'], $item['signup']) . ');';
                    }
                    $strng .= "\n";
                }
            } else
                if ($mode=="normal") $strng .= "No future signups located.\n";

            $strng .= "\n";
        }

        //add "make it happen" code to php encoding
        if ($mode!="normal") {
            $strng .= 'if (count($items)>0) foreach ($items as $item) addsignup($item);' . "\n";
            $strng .= "?>";
        }

        return($strng);
}

//----------------------------------------------------
// CREATEBACKUP
//create backup of current scheduler and save to file (or return if input ASFILE is false)
function createbackup ($asfile = true) {

    $backup_folder = root_dir . "../backup/";
    //Remove old backup files
    $filelist = getfilelist($backup_folder,false);
    foreach ($filelist as $file) {
        if (substr($file,-4)=='.ors' && filectime($backup_folder . $file)<(zulutime()-backup_history)) {
            unlink($backup_folder . $file);
        }
    }

    //create new backup file for today
    $backup_file = $backup_folder . date("Ymd",mktime());
    if (file_exists($backup_file . "_backup.ors")) {
        $rep = 'a';
        while (file_exists($backup_file . $rep . "_backup.ors")) {
            $rep++;
        }
        $backup_file .= $rep;
    }
    $backup_file .= "_backup.ors";

    //Open backup file
    if (in_array('zlib',get_loaded_extensions())) {
        if ($asfile) {
            //open compressed .ors file
            $fid = gzopen($backup_file,'wb');
            if (!$fid) {
                return(false);
            }
        }
    } else {  //if we don't have access to zlib, do regular write
        if ($asfile) {
            //open regular .ors file
            $fid = fopen($backup_file,'wb');
            if (!$fid) {
                return(false);
            }
        }
    }


    //Read in all files and and compress them
    $data = "";
    foreach (getfilelist(root_dir) as $filename) {
        $data .= "[==>FILENAME:" . $filename . "]\n";
        $data .= implode("",file(root_dir . $filename));
        $data .= "[==>EOF]\n";

        if ($asfile) {
            if (in_array('zlib',get_loaded_extensions())) {
                //Write out compressed data to .ors file
                gzwrite($fid,$data);
            } else {  //if we don't have access to zlib, do regular write
                //Write out data to .ors file
                fwrite($fid,$data);
            }
            $data = "";  //start data fresh again
        } else {
            //keep accumulating data until done
        }
    }

    if (in_array('zlib',get_loaded_extensions())) {
        if ($asfile) {
            gzclose($fid);
            return($backup_file);
        } else {
            $compdata = gzcompress($data);
            return($compdata);
        }
    } else {  //if we don't have access to zlib, do regular write
        if ($asfile) {
            fclose($fid);
            return($backup_file);
        } else {
            return($data);
        }
    }

}
//----------------------------------------------------
// MAILALLSIGNUPS
//Create backup and e-mail as needed
function mailallsignups (){

    //create backup file and e-mail to indicated users (if desired)
    $backup_file = createbackup();

    $addheaders = "From: " . email_from . "\n";
    $to         = "";
    foreach (getusers() as $usn => $oneuser) if (checksec($oneuser['sec'],1000)) $to .= getcontact($usn,"emails") . ",";

	//- - - - - - - - - - -
    if (mail_backup) {

        $subject = "On-Line Scheduling Backup (" . date(dateformat . "/Y",zulutime()+timezone) . ")";
        $message = "\nThe attached file is an encoded Backup of the scheduling data and can be ";
        $message .= "used to restore the system at " . scheduling_URL . " as of " . date(dateformat . "/Y " . timeformat,zulutime()+timezone) . ".\n";
        $message .= "To use this file, follow the instructions provided by the administrator maintenance menu \"Restore\" function.\n";
    
        $success1 = sendemail ($to, $subject, $message, $addheaders, $backup_file);
    }
    
	//- - - - - - - - - - -
    if (mail_action_log) mailactionlog();                //e-mail action log to special users

	//- - - - - - - - - - -
	if (mail_signups) {

        $subject = "On-Line Scheduling Signups (" . date(dateformat . "/Y",zulutime()+timezone) . ")";
        $message = "\nThe following is a record of the current signups ";
        $message .= "on the scheduler for " . scheduling_URL . " as of " . date(dateformat . "/Y " . timeformat,zulutime()+timezone) . ".\n";

        $message .= "\n" . decodeallsignups() . "\n";

        $success2 = sendemail ($to, $subject, $message, $addheaders);
	}

	//- - - - - - - - - - -

    logaction("Backup Mailed");

    //if (!$success1) $to = 'Backup e-mail failed';
    //elseif (!$successs2) $to = 'Signup listing e-mail failed';

    return($to);

}
//----------------------------------------------------
// MAILACTIONLOG
//E-mail action log to special users
function mailactionlog ($recent = true){

    $resourcelist = getresources();                        //get list of valid resources
    $users        = getusers();

    $message = "The following actions have been logged:\n\n";

    $items = readactionlog();

    if (count($items) > 0) {
        $items = array_reverse($items);
        if ($recent) {                //send ONLY recent items?
            $allitems = $items;
            $items = array();
            foreach ($allitems as $key => $oneitem) {
                if ($oneitem['data']['action'] != "Log Mailed") $items[$key] = $oneitem;
                else break;
            }
        }
    }

    if (count($items) == 0) {
        $message .= "Log is empty\n";
    } else {
        $message .= "Date, Action [Details]\n";
        foreach ($items as $linenum => $entry) {
            $data = $entry['data'];
            $message .= $entry['date'] . ", ";
            if ($data['action'] == "Bad Password") {
                $message .= $data['action'] . " for " . $data['lastname'];
            } elseif (in_array(strtolower($data['action']),
                   array("deleted authorized user","added authorized user","updated authorized user","restore from backup","updated configuration","assigned usn","deleted user"))) {
                $message .= $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " <b>" . $data['firstname']{0} . ". " . $data['lastname'] . "</b>";
                if (in_array("sec", array_keys ($data))) $message .= " SEC = " . $data['sec'];
                if (in_array("usn", array_keys ($data))) $message .= " USN = " . $data['usn'];
            } else {
                if ($data['thisuserid'] != $data['usn'] & $data['usn'] != "") {
            		$targetuser = $users[$data['usn']]['firstname']{0} . ". " . $users[$data['usn']]['lastname'];
            		if ($users[$data['usn']]['lastname']=="") $targetuser = "(usn " . $data['usn'] . ")";
                        $message .= $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " " . $targetuser;
                } elseif ($data['thisuserid'] != "") {
                        $message .= $users[$data['thisuserid']]['firstname']{0} . ". " . $users[$data['thisuserid']]['lastname'] . " " . $data['action'] . " self";
                } else
                        $message .= $data['action'];
                $message .= " [" . $resourcelist[$data['resource']]['short'];
                if (in_array("sec", array_keys ($data))) $message .= "SEC = " . $data['sec'];
                if (in_array("starttime", array_keys ($data)) & in_array("endtime", array_keys ($data)))
                    $message .= ", " . date(dateformat . " " . timeformat,$data['starttime']) . "-" . date(dateformat . " " . timeformat,$data['endtime']);
                if (in_array("priority", array_keys ($data))) $message .= ", priority " . $data['priority'];
                if (in_array("signup", array_keys ($data))) $message .= " (signup " . $data['signup'] . ")";
                $message .= "]";
            }
            $message .= "\n";
        }
    }
    $message .= "\n";

    $addheaders = "From: " . email_from . "\n";
    $to      = "";
    foreach ($users as $usn => $oneuser) if (checksec($oneuser['sec'],1000)) $to .= getcontact($usn,"emails") . ",";

    $subject = "On-Line Scheduling Action log (" . date(dateformat . "/Y",zulutime()+timezone) . ")";

    sendemail ($to, $subject, $message, $addheaders);

    logaction("Log Mailed");

    return($to);

}

//----------------------------------------------------
// RECONCILEEXPIREDACCOUNTS
// Check user accounts to indentify expired ones.  If expired, delete unauthorized future signups
// If no USN is provided, all accounts are checked for expiration, and reconciled if expired
// If a USN is supplied, force reconciliation for that user whether the account is expired or not
function reconcileexpiredaccounts ($usn = -1){
     $resourcelist = getresources();
     $users        = getusers();

     if ($usn != -1) {
        $selectedusns = array($usn);
        $forcereconciliation = TRUE;
     }else{
        $selectedusns = array_keys($users);  //get all usns
        $forcereconciliation = FALSE;
     }

     //get time stamp for the beginning of today
     $today = validatedate(date(dateformat . "/Y",zulutime()));

     $expiredorinactive = 0;
     foreach ($selectedusns as $memberUSN) {
          $foundexpiredaccount = 0;

          //Check if the account has expired
          if ((strcmp($users[$memberUSN]['expire'],"") != 0) && (strcmp($users[$memberUSN]['expire'],"Expired") != 0) && (!$forcereconciliation)){
                if (validatedate($users[$memberUSN]['expire']) <  $today) {
                    //Set account as expired so we don't check it again until it is reactivated
                    $member = $users[$memberUSN];
                    $users[$memberUSN]['expire'] = "Expired";
                    logaction('User account has expired: ',$thisuserid,array('usn'=>$memberUSN,'member'=>$member['firstname']." ".$member['lastname']));
                    $expiredorinactive = 1;
                    $foundexpiredaccount = 1;
                }
          }

          //Check if the account has become inactive
          if (stristr($users[$memberUSN]['lastactivity'], "inactive") == FALSE){ //first make sure we haven't already marked the account as inactive
               if (($today - validatedate($users[$memberUSN]['lastactivity']) > inactive_account) && (strcmp($users[$memberUSN]['lastactivity'],"") != 0)) {

                     $member = $users[$memberUSN];

                     //send e-mail to administrators
                     $message    = "Please be advised, the following user account has become inactive:.\n";
                     $message   .= $member['firstname']." ".$member['lastname']."\n";
                     $message   .= "Home Phone: ".$member['homephone']."\n";
                     $message   .= "Work Phone: ".$member['workphone']."\n";
                     $message   .= "Email Address 1: ".$member['email1']."\n";
                     $message   .= "Email Address 2: ".$member['email2']."\n";
                     $message   .= "Account Expiration: ".$member['expire']."\n";
                     $message   .= "\n";
                     $message   .= "You may want to contact this person to see if they plan to\n";
                     $message   .= "continue to use this scheduling system.\n";
                     $message   .= "\nIf desired, you may restrict their account in the\n";
                     $message   .= "User Management Screen.  By restricting their account you\n";
                     $message   .= "will limit their ability to keep their existing future signups\n";
                     $message   .= "as well as their ability to create new signups.\n";
                     $message   .= "\n";
                     $message   .= " Visit " . scheduling_URL . " for additional information.\n";
                     $addheaders = "From: " . email_from . "\n";
                     $to         = "";
                     foreach ($users as $usn => $oneuser) if (checksec($oneuser['sec'],100)) $to .= getcontact($usn) . ",";
                     $subject = "ORS User Account has become INACTIVE";
                     sendemail ($to, $subject, $message, $addheaders);

                     //add "inactive" to the inactivity field so we know that we don't have to check it again
                     $users[$memberUSN]['lastactivity'] = $users[$memberUSN]['lastactivity']."inactive";
                     logaction('User account has become inactive: ',$thisuserid,array('usn'=>$memberUSN,'member'=>$member['firstname']." ".$member['lastname']));
                     $expiredorinactive = 1;

                     //uncomment the line below to automatically expire an inactive account
                     //$foundexpiredaccount = 1;
               }
          }

          //If the account has become expired, or reconciliation is forced, then reconcile future signups
          if (($foundexpiredaccount) || ($forcereconciliation)) {
                 $usersignups = getusersignups($memberUSN, TRUE);
                 foreach ($usersignups as $signup) {

                      if (hard_account_expiration) {
                          promote(deletesignup(array('usn'=>$memberUSN,'signup'=>$signup['signup'])));
                          logaction('deleted signup (expired/restricted): ',$thisuserid,array('usn'=>$memberUSN,'signup'=>$signup['signup']));
                      }else {
                           //Check if this signup's resource is double-bookable
                           if (!$resourcelist[$signup['resource']]['doublebook']) {
                                 //Resource is not double-bookable, therefore check if there
                                 //   is another signup for same time period with a double-bookable resource
                                 $signupOK = FALSE;
                                 foreach ($usersignups as $signupcheck){
                                      //Find other signups with double-bookable resources
                                      if ($resourcelist[$signupcheck['resource']]['doublebook']) {
                                          if (($signup['starttime'] >= $signupcheck['starttime'])
                                          && ($signup['endtime'] <= $signupcheck['endtime'])) {
                                               //Matching double-bookable signup found.  Leave signup alone.
                                               $signupOK = TRUE;
                                               break;
                                          }
                                      }
                                 }
                                 //If no matching double-bookable signup found, then delete signup
                                 if ($signupOK == FALSE) {
                                     promote(deletesignup(array('usn'=>$memberUSN,'signup'=>$signup['signup'])));
                                     logaction('deleted signup (account expired): ',$thisuserid,array('usn'=>$memberUSN,'signup'=>$signup['signup']));
                                 }

                           }
                      }
                 }

          }
     }
     if ($expiredorinactive) putusers($users);
     return(0);

}

//---------------------------------------------------
// TIMESHIFTCORRECTION
// Correct calendar for problems due to a shift in system time
//  also handles item overlaps
function timeshiftcorrection() {

    $users = getusers();

    $missing = 0;
    $offsets = array();
    $inconflict = array();
    foreach ($users as $usn=>$user) {

        //get user's signups
        $signups = getusersignups($usn,false);

        if (!empty($signups)) {

            //cycle through all of them
            foreach ($signups as $signup=>$details) {

                foreach (array(0,1,-1) as $dayoffset) { //search day of, day after, day before for this signup

                    $mydate = $details['starttime']+($dayoffset*60*60*24);
                
                    //get the corresponding day
                    $dayssignups = getdaysignups ($mydate,$details['resource']);

                    //search for this signup
                    $found = false;
                    if (!empty($dayssignups)) foreach ($dayssignups as $daykey=>$onesignup) {
                        if ($onesignup['usn']==$usn && $onesignup['signup']==$signup) {
                            //calculate offset (including any day shift we had to make)
                            $offset = date('H',$details['starttime']) - $onesignup['hour'] - ($dayoffset*24);
                            $found = true;
                            break;
                        }
                    }

                    //if found, do correction
                    if ($found) {

                        $signups[$signup]['starttime'] -= $offset*60*60;
                        $signups[$signup]['endtime']   -= $offset*60*60;

                        //add offset to list
                        $offsets[] = "" . $offset;

                        //and check for potential conflicts
                        if (count($dayssignups)>1) {
                            $details = $signups[$signup]; //get modified version
                            $a = $onesignup; //the one we just found
                            foreach ($dayssignups as $b) {
                                if (($a['signup']!=$b['signup'] || $a['usn']!=$b['usn'])
                                    && $a['priority']==$b['priority']
                                    && $a['hour']<($b['hour']+$b['length'])
                                    && ($a['hour']+$a['length'])>$b['hour']) {
                                    //potential conflict!
                                    $pri = findpriority ($details['starttime'],$details['endtime'],$details['resource']);

                                    $dayssignups[$daykey]['priority'] = $pri;
                                    putdaysignups ($mydate,$details['resource'],$dayssignups);
                                
                                    $temp = $dayssignups[$daykey];
                                    $temp = array_merge($temp,$signups[$signup]);
                                    $temp['resource'] = $details['resource'];
                                    $temp['date']     = date(dateformat . "/Y",$mydate);
                                    $inconflict[]     = $temp;

                                }
                            }
                        }


                        //break out of "day loop"
                        break;

                    }
                }  //dayoffset loop

                if (!$found) {
                    //Not found in ANY day?
                    unset($signups[$signup]);
                    $missing ++;
                    //echo 'USN: ' . $usn . ' Signup: ' . $signup . '  Not found in calendar - deleted <br>';
                }
                            
            } //signups loop
            putusersignups($usn,$signups);
        }
    }

    ?>
    <b>Adjusting Mismatch to Calendar</b>
    <table cellpadding=3>
    <tr>
    <td align='center'><b>Offset<br>(hours)</b></td>
    <td align='center'><b>Number of<br>Signups</b></td>
    </tr>
    <p>
    <?
    foreach (array_count_values($offsets) as $offset=>$count) {
        echo "<tr>";
        echo "<td>$offset</td>";
        echo "<td>$count</td>";
        echo "</tr>";
    }
    ?>
    </table>
    <b>Signups without calendar entries:</b> <? echo $missing; ?> (removed)<p>
    <?

    //------------------------------------------------------------------
    // now, check for overlaps (may demote the wrong one, but what can we do!)
    ?>
    <b>Demoting Conflicted Signups (notifying users): </b>
    <?
    $total = 0;
    foreach ($inconflict as $item) {
        if ($item['endtime']<zulutime()) continue; //skip this item (in the past)
        $item['thisuserid'] = 1;  //administrator
        echo $users[$item['usn']]['username'] . ", ";
        message("demotion",$item);                                //Report demotion to user
        $total++;
    }
    if ($total==0) echo "None Found";

    //------------------------------------------------------------------
    // now, check for missed promotions
    ?>
    <p>

    <b>Promotions: </b>
    <?
    foreach ($users as $usn=>$user) {

        //get user's signups
        $signups = getusersignups($usn,false);

        if (!empty($signups)) {

            //cycle through all of them
            foreach ($signups as $signup=>$details) {

                if ($details['endtime']<zulutime()) continue; //skip this item (in the past)
            
                //get the corresponding day
                $dayssignup = getdaysignups ($details['starttime'],$details['resource']);

                //search for this signup
                $found = false;
                foreach ($dayssignup as $daykey=>$onesignup) {
                    if ($onesignup['usn']==$usn && $onesignup['signup']==$signup && $onesignup['priority']>1) {
                        $onesignup['resource']=$details['resource'];
                        promote(array($onesignup));
                        break;
                    }
                }
            }
        }
    }
}


//==============================================================================
//JMS 11/14/02
// -added complete am/pm clock support
//CBW 11/23/02
// -added function call to check for expired and inactive accounts 
//JMS 12/02/02
// -moved reconcileexpiredaccount here from functions
//JMS 2/23/03
// -modified reconciledexpiredaccounts to speed it up (repeated calls to lookupusn)
// -modified mailbackup to allow email of human-readable listing of signups 
// -email action log on schedule of backup (instead of every day)
// -modified format of action log
//JMS 3/02/03
// -added "announcementfile_name" to items to be backed up and restored
//JMS 4/8/03
// -added uselog.csv to list of files to backup
//JMS 7/9/03
// -use timezone offset in all references to zulutime
//JMS 8/17/03
// -add test for empty file in encode backup to avoid warning
//JMS 10/3/03
// -moved expire, restricted, and lastactivity into users
//JMS 10/6/03
// -fixed array_keys bug in reconcileexpiredaccounts
//JMS 12/15/03
// -fixed inactivate bug
//JMS 1/26/04
// -revised backup method
//JMS 2/7/04
// -added test for zlib in backup creation
// -force a minimum backup history of 24 hours
//JMS 2/19/04
// -removed limit of 24 hours backup history (bug anyway)
//JAT 3/25/04
// -added action log purging function
//JMS 4/23/04
// -changed to peace-meal building of backup file (less memory required)
//JMS 2/21/04
// -use validatedate to evaluate inactivity dates and expiration dates
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
?>
