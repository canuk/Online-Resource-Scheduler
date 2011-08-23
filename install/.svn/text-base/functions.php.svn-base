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

//----------------------------
// default settings
ignore_user_abort(true);
require_once('defaults.php');

if (file_exists(config_encoded)) {
    include(config_encoded);
} else
    activateconfig(config_file);

if (twentyfourhour_clock)
    define("timeformat","H:i");
else
    define("timeformat","h:i a");

if (eurodate) {
    define("dateformat","d/m");
    define("javadateformat","%d/%m");
} else {
    define("dateformat","m/d");
    define("javadateformat","%m/%d");
}
@set_time_limit(300);  //wait 5 mins for time-out (may not be permitted, so catch error)

//----------------------------
// SCHEDULELOCK
//create general lock to keep other processes
// from modifying files while we write
function schedulelock($uid = "") {

        if (!file_exists(root_dir)) {
            trigger_error("Cannot locate data folder - please reinstall ORS",
                E_USER_WARNING);
            return(false);
        }

        clearstatcache();

        //max x second wait
        $x = wait_for_lock;
        $startat = mktime();
        clearstatcache();

        if (file_exists(lockfile_name)) {
            if (debug_locks) echo "waiting for lock at: " . date("H:i:s m/d/y",mktime()) . "<br>";
            $waiting = True;
            $lockinfo = readtextfile(lockfile_name);        //see when this lock expires
            $startat  = $lockinfo[1];

            //if (false & $lockinfo[0]==$uid) { //MY lock? force unlock now
            //    scheduleunlock();
            //    $waiting = False;
            //}
                
        } else {
            $waiting = False;
        }

        srand((double) microtime()*1000000);

        //even if it didn't exist a moment ago, pause a short time and try again...
        for ($cycle = 1; $cycle<=2; $cycle++) {  //do multiple times
            $wt = rand(1,10)*1000;
            if (substr(php_uname(), 0, 7) == "Windows")
                for ($delay = 1; $delay<$wt; $delay++);
            else
                usleep($wt);
            
            while (file_exists(lockfile_name) & (mktime() - $startat)<$x) {
                $waiting = True;
                $wt = rand(1,10)*100;
                if (substr(php_uname(), 0, 7) == "Windows")
                    for ($delay = 1; $delay<$wt; $delay++);
                else
                    usleep($wt);
                clearstatcache();
            }
        }
        if ($waiting) {
            if (debug_locks) echo "Stopped waiting at: " . date("H:i:s m/d/y",mktime()) . "<br>";
            $lockinfo = readtextfile(lockfile_name);        //see when this lock expires
            if (mktime() > $lockinfo[1]+wait_for_lock)
                scheduleunlock();                //force an un-lock (dead process?)
            else
                return (false);
        }

        $fp = fopenlock(lockfile_name,"w");
        if (!$fp & debug_locks) {
            //do this in "unsquelched" mode to show user the error
            $fp = fopen(lockfile_name, "w");
        }
        if ($fp) {
            fwrite($fp,$uid . "\n");
            fwrite($fp,mktime() . "\n");
            fclose($fp);

            if (debug_locks) echo "locked: " . date("H:i:s m/d/y",mktime()) . "<br>";
        } else {            
            if (debug_locks) echo "Unable to create lockfile<br>";
            return (false);
        }

        return (true);
}
//----------------------------
// SCHEDULEUNLOCK
//Release lock
function scheduleunlock() {

        clearstatcache();
        if (file_exists(lockfile_name)) $success = unlink(lockfile_name);

        if (debug_locks) {
                if ($success) echo "unlocked: " . date("H:i:s m/d/y",mktime()) . "<br>";
                else echo "Unable to clear lockfile<br>";
        }
}
//----------------------------
// FOPENLOCK
// our own function to handle file opening - allows special locking action
function fopenlock($filename,$mode) {

        $fid = @fopen($filename, $mode);
        @set_file_buffer($fid,0);
        return($fid);
}
//----------------------------
// READTEXTFILE
//Read in a standard text file
function readtextfile($filename){
       $buffer="";
       if (file_exists($filename)) {
              $fd = fopenlock ($filename, "r");
              if ($fd!=FALSE) while (!feof ($fd)) {
                  $buffertmp = fgets($fd, 4096);
                  if (strlen($buffertmp) > 1) $buffer[]=$buffertmp;
              }
              fclose ($fd);
       }
       return $buffer;
}

//----------------------------
// WRITETEXTFILE
//Write a standard text file
function writetextfile($filename, $buffer) {
       $fd = fopenlock ($filename, "w+");
       if ($fd!=FALSE) {
             foreach ($buffer as $line3) fputs($fd, $line3);
             fclose ($fd);
             return 0;
       } else return 1;

}
//----------------------------
// ActivateCONFIG
// Read config file and setup the constant definitions
// Used only if we can't find the encoded version of the constants
function activateconfig($filename) {

       $configbuffer = readtextfile($filename);
       if (is_array($configbuffer) && count($configbuffer)>0) foreach ($configbuffer as $line0) {
            if (!strstr($line0, "****")) {
                $line0 = str_replace("&,", "&@", $line0);
                $linesplit = explode(",", $line0);
                foreach ($linesplit as $ind=>$linetmp) $linesplit[$ind] = trim(str_replace("&@", ',', $linetmp));
                $definition = $linesplit[2];
                if ((count($linesplit) > 2) && ($linesplit[0]!="") && ($linesplit[1]!="")){
                     if (strcasecmp($linesplit[1], "b")==0) {
                         if (strcasecmp($definition, "TRUE")==0) define($linesplit[0], TRUE);
                         if (strcasecmp($definition, "FALSE")==0) define($linesplit[0], FALSE);

                     }elseif (strcasecmp(substr($linesplit[1],0,1), "n")==0) {
                         $defnum = explode("*", $definition);
                         $mltply=1;
                         foreach ($defnum as $defnum2){
                             $mltply = $mltply*$defnum2;
                         }
                         define($linesplit[0], $mltply);
                     } else define($linesplit[0], $definition);
                }
            }
       }
}
//----------------------------
// FGETCSV_TRIM
//Read in one line of a csv file but trim leading/trailing spaces off each array element
// inputs are identical to fgetcsv
function fgetcsv_trim($fp, $length, $sep=',',$encl='"') {

    $items = fgetcsv ($fp, $length, $sep);  //encl is no longer used (default ONLY)
    if (is_array($items) && count($items)>0) 
        foreach ($items as $index=>$item) $items[$index]=trim($item);
    return($items);

}
//----------------------------
// READKEYEDFILE
//Read in any CSV file with keys in first line
// IN: $filename       name of file to read from
//     $index_key      key to use as primary array index
//     $must_exist     name of keys which must be non-empty to include line
//     $default_values array of keys and values which should be used if read value is empty
//     $alldata        starting value for array (e.g. for append, pass existing array)
//     $defaultkeys    overrride of keys - if present and first key does NOT match
//                      keys read from file, defaultkeys is used instead and first line is considered a data block
function readkeyedfile($filename, $index_key = "", $must_exist = array(), $default_values = array(), $alldata = array(), $defaultkeys = array()) {

    //make sure root_dir exists
    if (!file_exists(root_dir)) mkdir (root_dir, 0700);

    clearstatcache();
    if (file_exists($filename)) {
        $fp = fopenlock ($filename,"r");

        $keys = fgetcsv_trim ($fp, 1000, ",");
        if (is_array($keys) && $keys[count($keys)-1]=="" && count($keys)>1) foreach ($keys as $index=>$key) $keys[$index]=trim($key);
        while (!feof($fp) && count($keys)==1 && $keys[0]=="") $keys = fgetcsv_trim ($fp, 1000, ","); //skip blank line(s)
        if (count($defaultkeys) > 0)
            if ($defaultkeys[0] != $keys[0]) {
                $keys = $defaultkeys;
                rewind($fp);
            }
        while (is_array($keys) && $keys[count($keys)-1]=="" && count($keys)>1) unset($keys[count($keys)-1]);                //clear empty last column(s)
        if (is_array($keys)) {
            foreach ($keys as $index=>$key) $keys[$index] = strtolower($key);        //make lower case
            $keys = str_replace(" ","",$keys);                                        //drop any spaces

            while ($data = fgetcsv_trim ($fp, 1000, ",")) {
                if (count($data)==1 && $data[0]=="") continue; //skip blank line(s)
                foreach ($keys as $index => $key) {
                    if (count($data) >= $index+1)
                        $oneline[$key] = trim($data[$index]);                //get value if it's there
                    else
                        $oneline[$key] = "";                        //otherwise, insert empty value into key
                }

                //insert default values for all expected keys
                if (count($default_values)>0) foreach ($default_values as $key=>$value) if ($oneline[$key] == "") $oneline[$key] = $default_values[$key];

                $use_line = count($oneline)>0;   //first, we'll only use this line if it isn't empty

                //then we'll test for required fields
                if (count($must_exist)>0) foreach ($must_exist as $key) if ($oneline[$key] == "") $use_line = false;

                if ($use_line) {
                    if ($index_key != "" && in_array($index_key, array_keys($oneline)))
                        $alldata[$oneline[$index_key]] = $oneline;
                    else
                        $alldata[] = $oneline;
                }
            }
        }
        fclose ($fp);
    }

    return $alldata;

}
//----------------------------
// WRITEKEYEDFILE
//Write out any CSV file with keys in first line
// WARNING! Keys are determined ONLY from the first record (and default_values). If other records
//  have extra keys, they will not be written unless they exist in default_values too!
// IN: $filename       name of file to write to
//     $records        array to write out as CSV
//     $index_key      key to use as "index" (will be first key written for each record)
//     $default_values array of keys and values which should be used if array value is empty
function writekeyedfile($filename, $records, $index_key = "", $default_values = array()) {

    //make sure root_dir exists
    if (!file_exists(root_dir)) mkdir (root_dir, 0700);

    $fp   = fopenlock ($filename,"w");

    if (!$fp) {
      echo "<h2>Unable to open " . $filename . " (file permissions problem?)<br>";
      echo "Please contact the system administrator and relay this warning message.</h2>";
      return(Null);
    }

    if (count($records)>0) {
        //Determine keys
        $foundkeys = array();
        foreach ($records as $record)
            $foundkeys = array_merge($foundkeys,array_keys($record));           //add keys from this record
        if (count($default_values) > 0)
            foreach ($default_values as $key=>$value)
                if (!in_array($key,$foundkeys)) $foundkeys[] = $key;  //add default keys
        foreach ($foundkeys as $index=>$key) if ($key=="" | $key=="0") unset($foundkeys[$index]);

        //put index_key first
        $keys = array();
        if ($index_key != "") $keys[0] = $index_key;
        $foundkeys = array_unique($foundkeys);                                //only unique keys
        $foundkeys = array_diff($foundkeys,array($index_key));
        $keys = array_merge($keys,$foundkeys);                                //add other keys after index_key


        //start by writing the keys
        $itemstring = "";
        foreach ($keys as $key)
            $itemstring = $itemstring . '' . $key . ', ';
        $itemstring = $itemstring . "\n";
        fwrite ($fp, $itemstring);

        //then do the data
        foreach ($records as $item) {
            $itemstring = "";
            foreach ($keys as $key) {
                if (in_array($key,array_keys($item))) {
                    if (strstr($item[$key],','))         //quote if commas are found
                        $itemstring = $itemstring . '"' . $item[$key] . '", ';
                    else
                        $itemstring = $itemstring . '' . $item[$key] . ', ';
                } else
                    $itemstring = $itemstring . '"' . $default_values[$key] . '", ';
            }
            $itemstring = substr($itemstring,0,strlen($itemstring)-2) . "\n";
            fwrite ($fp, $itemstring);
        }
    }
    fclose ($fp);

}
//----------------------------
// GETFILELIST
//  returns a list of files in a specified directory
//   and (optionally) all of it's sub-directories
// Input: 
//   dir : the directory to start at
//   recurse : true = recursive listing of files in sub-dirs too
//   basedir : prepended directory name
//   incldirs : should dirs appear in file list
// Output:
//   array of file names with prepended folder names
function getfilelist($dir,$recurse=true,$incldirs=false,$basedir="") {

    if ($basedir == "") {
        $basedir = $dir;
        $dir = "";
    }

    $files = array();
    $dirs  = array();

    $handle = @opendir($basedir . $dir);
    if ($handle) {
        while (false!=($file = readdir($handle))) {
            if ($file != "." && $file != "..") {
                if (!is_dir($basedir . $dir .$file)) {
                    $files[] = $dir . $file;
                } else {
                    $dirs[]  = $dir . $file . "/";
                }
            }
        }
        closedir($handle);

        if ($incldirs) $files[] = $dir;
        
        //parse subdirs for content files
        if ($recurse && count($dirs)>0) foreach ($dirs as $subdir) $files = array_merge($files,getfilelist($subdir,$recurse,$incldirs,$basedir));

    } else {
        echo "Could not open " . $basedir . $dir . "<br>";
    }

    return($files);

}
//----------------------------
// GETMEMBERLIST
//Read in member list
// Usually includes name, email, phone, and PIN plus other info; but NOT user # (usn)
function getmemberlist($inclauth = false) { //used to be: $users = array()) {

        if ($GLOBALS['membersfile']!=Null) $users = $GLOBALS['membersfile'];  //if a copy is available in globals, use it (faster but requires register globals to be on)
        else {
            $users = readkeyedfile(memberlist_name, "", array("lastname"), array());
            $users = sortbyname($users);
            $GLOBALS['membersfile'] = $users;
        }

        if ($inclauth) $users = array_merge($users,getauthorizedlist());
        return($users);

}
//----------------------------------------------------
// PUTMEMBERLIST
//Write out member list
function putmemberlist ($members) {

        if (demomode) return(null);
        writekeyedfile(memberlist_name, $members, "lastname");
        $GLOBALS['membersfile'] = $members;
        return(null);

}
//----------------------------
// GETAUTHORIZEDLIST
//Read in authorized user list
// (list of authorized users who might not be in member list)
// Usually includes name, and PIN  but NOT user # (usn)
function getauthorizedlist($users = array()) {

        $users = readkeyedfile(authorizedlist_name, "", array("lastname"), array(), $users);
        return sortbyname($users);

}
//----------------------------------------------------
// PUTAUTHORIZEDLIST
//Write out authorized user list
// (list of authorized users who might not be in member list)
// Includes name, and PIN  but NOT user # (usn)
function putauthorizedlist($users = array()) {

        if (demomode) return(null);

        foreach ($users as $index => $item) {
                if ($users[$index]['lastname']=="Administrator" & $users[$index]['firstname']=="")
                        unset($users[$index]);
                else {
                        $users[$index]['lastname']  = ucfirst($item['lastname']);
                        $users[$index]['firstname'] = ucfirst($item['firstname']);
                }
        }
        if (count($users)>0) writekeyedfile(authorizedlist_name, $users, 'lastname');
        elseif (file_exists(authorizedlist_name)) unlink(authorizedlist_name);

}
//----------------------------------------------------
// GETCONTACT
//Look up a user's contact info (e-mail address, phone #s, etc)
//input $which is a string key, default is e-mail 1
function getcontact ($usn, $which = "emails"){
        $users = getusers();

        if ($which == 'emails') {
                //special call which returns both emails (or just one if only one exists)
                $temp1 = getcontact($usn,'email1');
                $temp2 = getcontact($usn,'email2');
                $temp  = "";
                if ($temp1 != "" & $temp2 != "") $temp = $temp1.', '.$temp2;
                if ($temp1 != "" & $temp2 == "") $temp = $temp1;
                if ($temp1 == "" & $temp2 != "") $temp = $temp2;
                return($temp);
        } elseif ($which == 'emails_html') {
                //special call which returns both emails (or just one if only one exists) IN HTML FORM
                $temp1 = getcontact($usn,'email1');
                $temp2 = getcontact($usn,'email2');
                $temp  = "";
                if ($temp1 != "" & $temp2 != "") $temp = '<a href = "mailto:'.$temp1.'">'.$temp1.'</a>, <a href = "mailto:'.$temp2.'">'.$temp2.'</a>';
                if ($temp1 != "" & $temp2 == "") $temp = '<a href = "mailto:'.$temp1.'">'.$temp1.'</a>';
                if ($temp1 == "" & $temp2 != "") $temp = '<a href = "mailto:'.$temp2.'">'.$temp2.'</a>';
                return($temp);
        } else {
                return ($users[$usn][$which]);
        }
}
//----------------------------------------------------
// GETUSERS
//Read in user list
// only contains usn, name, resourceblock, and security code
function getusers (){

        if ($GLOBALS['usersfile']!=Null) return($GLOBALS['usersfile']);  //if a copy is available in globals, use it (faster but requires register globals to be on)
	
	if (file_exists(root_dir . 'userlist.csv')) {
		$users = readkeyedfile(root_dir . "userlist.csv",
		    "usn",
		    array("usn"),
		    array("sec"=>0),
		    array(),
		    array("usn", "resourceblock", "lastname", "firstname", "sec"));
	
		$oldresblock = FALSE;
		foreach ($users as $usn => $oneuser) {
		    $users[$usn]['lastname']  = ucfirst($oneuser['lastname']);
		    $users[$usn]['firstname'] = ucfirst($oneuser['firstname']);
		    //For backwards compatibility, read in old-integer values
		    //  and convert to the new "b-string" format
		    if (strcmp($users[$usn]['resourceblock'][0], "b") != 0){
		       $users[$usn]['resourceblock'] = "b".strrev(strval(($users[$usn]['resourceblock'])));
		       $oldresblock = TRUE;
		    }
		    //fix administrator account SEC
		    if ($users[$usn]['lastname'] == "Administrator" && $users[$usn]['firstname'] == "") {
			    $users[$usn] = array_merge($users[$usn],
				array(
				    "lastname"  =>"Administrator",
				    "firstname" =>"",
				    "sec"       =>1110,
				    "pin"       =>admin_password,
				    "email1"    =>admin_email1,
				    "email2"    =>admin_email2));
		    }
		}
		if($oldresblock) putusers($users); //if old resource block exists rewrite with new format
	
		$GLOBALS['usersfile'] = $users;
	}
	else {
		// Create an administrator account
		// FIXME: Should probably be part of install script
		$users[1] = array(
				'usn'			=>1,
				'lastname'  	=>'Administrator',
				'firstname' 	=>'',
				'sec'			=>1110,
				'pin'       		=>admin_password,
				'email1'	    	=>admin_email1,
				'email2'    		=>admin_email2);
		putusers($users);
	}

        return $users;
}
//----------------------------------------------------
// PUTUSERS
//Write out user list
// only contains usn, name, and security code
function putusers ($users) {

        if (demomode) return(Null);
        writekeyedfile(root_dir . "userlist.csv", $users, "usn");
        $GLOBALS['usersfile'] = $users;

}
//----------------------------------------------------
// VALIDATEUSERS
// make sure all users are in member list and that all member list
// people are in users (handles people being removed)
// If save_deleteduser_signups is true, then any signups a user
//   has will be transferred to an authorized user "Abandoned"
//   UNLESS the user being deleted IS "Abandoned", in which case
//   the signups will ALWAYS be deleted.
function validateusers() {

return;

}
//----------------------------------------------------
// REMOVEUSER
// Remove indicated users
// If save_deleteduser_signups is true, then any signups a user
//   has will be transferred to an authorized user "Abandoned"
//   UNLESS the user being deleted IS "Abandoned", in which case
//   the signups will ALWAYS be deleted.
function removeuser($usns) {

    if ($usns=="" || $usns==Null) return;
    if (!is_array($usns) && $usns!="") $usns = array($usns);

    //usn without matching username?
    if (count($usns) > 0) {

        $did_abandon = null;
        $total = 0;
        foreach ($usns as $usn) {
                $allusers = getusers();
                $user = $allusers[$usn];
                logaction("Deleted User","",$user);
                unset($allusers[$usn]);
                putusers($allusers);

                $isabandoned = !strcasecmp ("Abandoned", str_replace(" ","",$user['firstname'] . $user['lastname']));
                if (!$isabandoned) {
                    $signups = getusersignups ($usn, False);                //get user signups
                    $total  += count($signups);
                }

        }

        //create Abandoned user if necessary
        if ($total > 0 & save_deleteduser_signups) {
            $abandoned_usn = null;
            foreach (getusers() as $index => $user) {
                $usercode = $user['firstname'] . $user['lastname'];
                $usercode = str_replace(" ","",$usercode);
                if (!strcasecmp ("Abandoned", $usercode)) { $abandoned_usn = $index; break; }
            }
            if ($abandoned_usn == null) {                //if abandoned didn't exist before now...
                $auth     = getauthorizedlist();
                $thisuser = array('lastname'=>"Abandoned", "firstname"=>"", "pin"=>"");
                $auth[]   = $thisuser;
                $auth     = sortbyname($auth);
                putauthorizedlist($auth);                        //add them as authorized user
                //and get new usn for them
                $abandoned_usn = assignusn($thisuser['lastname'],$thisuser['lastname'],$thisuser['firstname'],0);
            }
        }

        //remove user and delete signups
        foreach ($usns as $usn) {
            $signups = getusersignups ($usn, False);                //get user signups
            //deal with signups
            if (count($signups) > 0) {
                foreach ($signups as $signupnumber=>$signup) {
                    if (!save_deleteduser_signups)        //delete them
                        promote(deletesignup(array('usn'=>$usn,'signup'=>$signupnumber)),false);        //promote quietly
                    else                 //assign to Abandonded user
                        update(array('usn'=>$usn,
                                     'signup'=>$signupnumber,
                                     'comment'=>$signup['comment'],
                                     'thisuserid'=>1
                                     ), $abandoned_usn, "",true);
                                     //do this as administrator (usn = 1)
                }
                if (save_deleteduser_signups & !$isabandoned) {
                    logaction("Abandoned Signups","",$signups);
                    $did_abandon[] = $usns;
                }
            }
        }

        if (count($did_abandon)>0) {
            $message    = "Please be advised, users were deleted and their sign-ups were assigned to user \"abandoned\".\n";
            $message   .= "\n";
            $message   .= " Visit " . scheduling_URL . " for additional information.\n";
            $addheaders = "From: " . email_from . "\n";

            $to         = "";
            foreach (getusers() as $usn => $oneuser) if (checksec($oneuser['sec'],1000)) $to .= getcontact($usn) . ",";

            $subject = "Abandoned Sign-ups";

            sendemail ($to, $subject, $message, $addheaders);
        }

    }
}
//----------------------------------------------------
// OUR_CRYPT
//our own version of crypt using our own version of a salt
function our_crypt ($password) {

        if ($password=="") return("");

        //Create Salt for Crypt Function
        $saltd1 = ord($password);
        $saltd2 = ord(substr($password,1,1));
        if ($saltd1 < 80) {
             $saltd1 = $saltd1 + intval(($saltd1/5));
        } else {
             $saltd1 = $saltd1 - intval(($saltd1/5));
        }
        if ($saltd2 < 80) {
             $saltd2 = $saltd2 + intval(($saltd2/5));
        } else {
             $saltd2 = $saltd2 - intval(($saltd2/5));
        }
        $salt = chr($saltd1).chr($saltd2);
        return(crypt($password, $salt));

}
//----------------------------------------------------
// CHECKSEC
//check for specific SEC settings
// requires an SEC to check and the expected flags in $needed
function checksec ($sec, $needed) {

        //first convert binary to integer
        $eneeded = 0;
        for ($n = 0; floor($needed/pow(10,$n))>0; $n++) $eneeded += ((floor($needed/pow(10,$n))%2)>0)*pow(2,$n);
        $esec = 0;
        for ($n = 0; floor($sec/pow(10,$n))>0; $n++) $esec += ((floor($sec/pow(10,$n))%2)>0)*pow(2,$n);

        //then compare encoded "needed" to encoded "sec"
        if (($esec & $eneeded) == $eneeded) return (true);
        return (false);
}
//----------------------------------------------------
// CHECKPERMISSIONS
//check for specific permissions
// requires a permissions string to check for the character $checkchr
// for resource $resourcenum
function checkpermissions ($permissions, $checkchr, $resourcenum) {
         if ((strlen($permissions)-1) < $resourcenum) {
              $perm = "0";         //if this resource bit was never set, then by defualt the user has access
         } else //bit was set - get permissions from string
              $perm = substr($permissions,$resourcenum, 1);
         //check permissions
         if ($checkchr!="0") return(((0+$checkchr) & (0+$perm))==$checkchr);  //compare bit-wise
         else return ((1 & (0+$perm))!=1);   //special case (0 implies bit 1 NOT set)
}
//----------------------------------------------------
// GETUIDS
//read in UID list
// ALSO: check if we should do first-login-of-day actions
// UID list is usn, encoded UID and expiration date info
function getuids(){

        $users = readkeyedfile(root_dir . "uid.csv",
                        "uid",
                        array("uid"),
                        array("issuedate"=>0,"lastused"=>0,"prefs"=>""),
                        array(),
                        array("uid", "usn", "issuedate", "lastused","prefs"));

        //don't read in UID key if issuedate is older then uid_history
        foreach ($users as $index => $user)
                if ($user['lastused'] <= zulutime()+timezone-uid_valid) unset($users[$index]);

        return $users;

}
//----------------------------------------------------
// PUTUIDS
//write out UID list
function putuids($allusn){

        writekeyedfile(root_dir . "uid.csv", $allusn, "uid");

}
//----------------------------------------------------
// GETUSELOG
//read in USELOG
// UID list is usn, encoded UID and expiration date info
function getuselog(){

        $uselog = readkeyedfile(root_dir . "uselog.csv",
                        "uid",
                        array("uid"),
                        array("logondate"=>0),
                        array(),
                        array("uid", "usn", "logondate"));

        //don't read in UID key if issuedate is older then uid_history
        foreach ($uselog as $index => $user)
                if ($user['logondate'] <= zulutime()+timezone-uid_history) unset($uselog[$index]);

        return $uselog;

}
//----------------------------------------------------
// PUTUSELOG
//write out USELOG
function putuselog($uselog){

        writekeyedfile(root_dir . "uselog.csv", $uselog, "uid");

}
//----------------------------------------------------
// ASSIGNUID
//create and add item to UID list -- also removes expired
//  really old UID items
function assignuid ($usn){

    recordactivity($usn);

        $uid_length   = 13; //How long should the session ID be (Longer = more secure but longer download time&, 13 is probably good enough!)

        $uid          = md5($usn . zulutime()+timezone);        //make UID code (MD5 encoding of usn and current time)
        $uid          = substr($uid,0,$uid_length);        //truncate to requested UID length
        $allusn       = getuids ();                        //read in all but over-timed UIDs
        $allusn[$uid] = array(
                        "uid"       => $uid,
                        "usn"       => $usn,
                        "issuedate" => zulutime()+timezone,
                        "lastused"  => zulutime()+timezone,
                        "prefs"     => ""
                                                );                //add new item
        putuids ($allusn);                        //write out list
        return $uid;
}
//----------------------------------------------------
// LOGOUTUID
//invalidate UID NOW (but keep usn and time for later review)
//  Normally, the "lastused" date is used to figure out when
//  the user hasn't done anything for "n" minutes. This
//  function changes the "lastused" time to be the difference
//  between the last-revalidated "lastused" time and the "issuedate".
//  This new value is kept so we can tell how long the user
//  was logged in (if we care to look later). However, it is an
//  invalid "lastused" value because, if this was interpreted as a
//  date, it would be something like Jan 1, 1970.
function logoutuid ($UID){
        $allusn = getuids ();
        $user   = $allusn[$UID];
        if (count($user)>0) {
                //$user['lastused'] = $user['lastused']-$user['issuedate'];                //invalidate now
                //$allusn[$UID] = $user;
                unset($allusn[$UID]);
                putuids($allusn);
        }
}
//----------------------------------------------------
// LOOKUPUSN
//Search $users array for user matching the string
// passed in $tolookup as either a firstname+lastname
// or a lastname or a firstname/lastname which SOUNDS
// like what was provided.
function lookupusn($tolookup,$users=null) {

        if ($users==null) $users = getusers();
        $tolookup = str_replace(" ","",$tolookup);
        foreach ($users as $index => $user) {
                $usercode = $user['username'];
                if ($usercode == "") $usercode = $user['lastname'];
                $usercode = str_replace(" ","",$usercode);
                if (!strcasecmp ($tolookup, $usercode)) return($index);
        }
        return (null);
}
//----------------------------------------------------
// LOOKUPMEMINDEX
//Search $members array for the array index of the
// user matching the user number passed in $usn.
// If the user is not found (e.g. it is an authorized user)
//  then "-1" is returned
// This function relates the USN to the member list.
function lookupmemindex($usn) {

        $users   = getusers();
        $members = getmemberlist();

        $usercheck = $users[$usn]['firstname'] . $users[$usn]['lastname'];
        $usercheck = str_replace(" ","",$usercheck);

        foreach ($members as $index => $membersearch) {
                $membercheck = $membersearch['firstname'] . $membersearch['lastname'];
                $membercheck = str_replace(" ","",$membercheck);
                if (!strcasecmp ($membercheck, $usercheck)) return($index);
        }
        return (-1);
}
//----------------------------------------------------
// ASSIGNUSN
//Create new usn and insert into userlist
function assignusn($username,$lastname,$firstname,$sec) {

        $usn = "";
        //try reading the lastusn file
        // this allows us to keep assigning NEW user numbers instead of reusing a usn
        // if the last user in the list is deleted
        if (file_exists(root_dir . "lastusn.csv")) {
            $lastusn = readtextfile(root_dir . "lastusn.csv");        //get last assigned usn (if there)
            $usn = $lastusn[0]+1;
        }

        //if that fails, try the OLD method
        $users = getusers();
        if (empty($usn) || $usn==0) {
            if (count($users)>0) {
                    $usn = max(array_keys($users)) + 1;
            } else {
                $usn = 1;
            }
        }

        //write this usn out to the file as the last usn assigned
        $fp = fopenlock(root_dir . "lastusn.csv","w");
        if ($fp) {
            fwrite($fp,$usn . "\n");
            fclose($fp);
        }        

        $resourceblock = "b";
        foreach (getresources() as $oneitem) {
            $strlength = strlen($resourceblock);
            if($oneitem['resource'] > ($strlength-1)) for($grow = 0; $grow <= ($oneitem['resource']-$strlength); $grow++) $resourceblock=$resourceblock."1";
            if($oneitem['defaultaccess'] == 1) $resourceblock[$oneitem['resource']] = "0";
        }
        $users[$usn] = array(
                'usn' => $usn,
                'username'  => $username,
                'lastname'  => $lastname,
                'firstname' => $firstname,
                'resourceblock' => $resourceblock,
                'sec' => $sec);
        putusers($users);

        logaction("Assigned USN","",$users[$usn]);
        return($usn);
}
//----------------------------------------------------
// USERMENU
//Give a menu of users
// inputs (all optional):
//  usn       : usernumber to have selected
//    (if USN is empty or that usn doesn't match secmask, selected option will be "Select User")
//  itemname  : name for the pull-down menu
//  returnusn : boolean: true = return usn as value (otherwise, return name)
//  secmask   : security mask (only those user with sec containing this mask image (binary) will be shown)
function usermenu($usn = 0, $itemname = "usn", $returnusn = true, $secmask = 0) {

        if ($usn == "") $usn = 0;

        $users = getusers();
        $allowed_signup = ($users[$usn]['sec'] & $secmask);
        $users = sortbyname($users);

        echo '<select name="' . $itemname . '">';
        if ($usn == 0 | $usn == "" | $usn == " " | !$allowed_signup )
                echo "<option value=\"\" selected>Select User...</option>\n";
        foreach ($users as $user) {
                if (!$secmask | ($user['sec'] & $secmask)) {
                        if ($returnusn) echo "<option value=\"" . $user['usn'] . "\"";
                        else echo "<option value=\"${user['firstname']} ${user['lastname']}\"";
                        if ($usn == $user['usn']) echo " selected";
                        if ($user['firstname'] == "")
                                echo ">${user['lastname']} *</option>\n";
                        else
                                echo ">${user['lastname']}, ${user['firstname']}</option>\n";
                }
        }
        echo " </select>";
}
//----------------------------------------------------
// SORTBYNAME
// sort any list with firstname and lastname fields by those fields (last,first)
function sortbyname($users) {

        $users = sortby($users,"firstname");
        $users = sortby($users,"lastname");
        return ($users);

}
//----------------------------------------------------
// SORTBY
// sort any list by any subfield
// WARNING: modifies record keys
function sortby($items,$sortkey) {

        if (count($items) > 0) {
                $extracted = array();
                foreach ($items as $item) $extracted[] = $item[$sortkey];
                array_multisort($extracted,$items);
        }
        return ($items);

}
//----------------------------------------------------
// USERIDLOOKUP
//Search $users array for user matching the lastname and password fields
// passed in $tolookup
function useridlookup($tolookup) {
        $fields = array_keys($tolookup);

        $try_remote = (remote_auth && $tolookup['username']=="" && $tolookup['password']=="" && $tolookup['UID']=="");

        if ((in_array("username", $fields) && in_array("password", $fields)) || $try_remote ) {
                $users = getusers(true);

                //Make password case-INsensitive
                $enterpassword  = strtolower($tolookup['password']);
                $key = $tolookup['username'] . our_crypt(str_pad($enterpassword, 4, "0", STR_PAD_LEFT));
                if ($try_remote) {
                    $key = $tolookup['REMOTE_USER'];
                }

                foreach ($users as $usn => $user) {
                        $user['pin'] = str_pad($user['pin'], 4, "0", STR_PAD_LEFT);

                        if ($user['username']=="") $user['username']=strtolower($user['lastname']);  //use last name if username is blank (old format)
                        $usercode = $user['username'] . our_crypt($user['pin']);        //plain-text support
                        $usercode2 = $user['username'] . $user['pin'];                        //encrypted pw support
                        if ($try_remote) $usercode2 = $user['username'];

                        //if name+PIN matches (and PIN is NOT empty!) this is them
                        if ((!strcasecmp ($key, $usercode) | !strcasecmp ($key, $usercode2)) && (remote_auth || strlen($user['pin'])>0))  {

                                $userinfo = array('usn' => $usn, 'UID' => assignuid($usn), 'prefs'=>"" );

                                //Add login entry uselog.csv
                                $uselog = getuselog();
                                $uselog[] = array('usn' => $usn, 'uid' => $userinfo['UID'], 'logondate' => zulutime()+timezone, 'ip' => $_SERVER['REMOTE_ADDR']);
                                putuselog($uselog);

                                if (file_exists(announcementfile_name)) {
                                    include(announcementfile_name);
                                }
                                return($userinfo);
                        }
                }
                if ($tolookup['lastname'] != "") logaction("Bad Password","",$tolookup);                //log the bad password

        } elseif (in_array("UID", $fields) & $tolookup['UID'] != "") {
                //lastname/password fields not there but we have a UID code, look it up
                $allusn = getuids ();
                $user   = $allusn[$tolookup['UID']];
                if (count($user)>0) {
                        if ($user['lastused'] > (zulutime()+timezone-uid_valid)) {
                                $user['lastused'] = zulutime()+timezone;                //revalidate for more time
                                $allusn[$tolookup['UID']] = $user;
                                putuids($allusn);

                                $userinfo = array('usn'=>$user['usn'],'UID'=>$tolookup['UID'],'prefs'=>$tolookup['prefs']);
                                return($userinfo);

                        } else
                            echo "<h3>Your session has expired...</h3>";
                } else
                    echo "<h3>Your session has expired...</h3>";
        }
        $userinfo = array('usn'=>null,'UID'=>null, 'prefs'=>null);                //default answer (not there)
        return($userinfo);
}
//----------------------------------------------------
// RECORDACTIVITY
//  reset 'lastactivity' field to date of most recent signup activity
// $usn is usernumber to update
function recordactivity ($usn) {

    $users = getusers();
    $lastactivity = $users[$usn]['lastactivity'];
    if (time() > validatedate(str_replace("inactive","",$lastactivity))) {
         $users[$usn]['lastactivity'] = date(dateformat . "/y");
         putusers($users);
    }

}
//----------------------------------------------------
// GETUSERSIGNUPS
//Read in all of a user's signups
//u"usn".csv:  signup, resource, starttime, endtime, comment
function getusersignups ($usn, $datetrim = True){

        $signups = readkeyedfile(root_dir . "u" . $usn . ".csv",
                        "signup",
                        array("signup","resource","starttime","endtime"),
                        array("comment"=>"","status"=>""),
                        array(),
                        array("signup","resource","starttime","endtime","comment","status"));

        //do not read in signups already in the past if datetrim is on
        if ($datetrim) foreach ($signups as $index => $signup)
                if ($signup['endtime'] < zulutime()+timezone+allow_signup_offset) unset($signups[$index]);

        //NEVER read in signups beyond calendar_history old
        foreach ($signups as $index => $signup)
                if ($signup['endtime'] < zulutime()+timezone-calendar_history) unset($signups[$index]);

        return $signups;

}
//----------------------------------------------------
// PUTUSERSIGNUPS
//Write out all of a user's signups
//u"usn".csv:  signup, resource, starttime, endtime, comment
function putusersignups ($usn,$data){

        writekeyedfile(root_dir . "u" . $usn . ".csv", $data, "signup");

}
//----------------------------------------------------
// SETSIGNUPSTATUS
//Records a status string for a particular user's signup
function setsignupstatus($usn,$signup,$status) {
    $signups = getusersignups($usn,false);
    if (count($signups[$signup])>0) { //signup exists?
        $signups[$signup]['status'] = $status;

        //recognize special status changes and note time
        if ($status==1) $signups[$signup]['checkout'] = zulutime()+timezone;
        elseif ($status==0) $signups[$signup]['checkin'] = zulutime()+timezone;
        elseif ($status=="") {
            $signups[$signup]['checkin'] = "";
            $signups[$signup]['checkout'] = "";
        }

        putusersignups($usn,$signups);
    }
}
//----------------------------------------------------
// GETRESOURCES
//read in resources list
function getresources(){

        if ($GLOBALS['resourcesfile']!=Null) return($GLOBALS['resourcesfile']);  //if a copy is available in globals, use it (faster but requires register globals to be on)

        $resources = readkeyedfile(resourcesfile_name,
                        "resource",
                        array("resource"),
                        array(),
                        array(),
                        array("resource","short","long","initials","maxsignup","doublebook","hidepublic"));

        //default is no resources

        //fill in missing order values
        foreach ($resources as $index=>$resource) if ($resource['order']=="") $resources[$index]['order']=$resource['resource'];

        $GLOBALS['resourcesfile'] = $resources;

        return $resources;

}
//----------------------------------------------------
// PUTRESOURCES
//write out resources list
function putresources($resources) {

        if (demomode) return(Null);
        writekeyedfile(resourcesfile_name, $resources, "resource");
        $GLOBALS['resourcesfile'] = $resources;

}
//----------------------------------------------------
// GETDAYSIGNUPS
//Read in one days' signups
//"date".csv:  hour, priority, usn, signup, length
function getdaysignups ($date,$resource){

        $dayinfo = array();
        if ($resource<1 || $date<=0) return($dayinfo);   //stop if invalid date or resource
        
        if (!file_exists(root_dir . $resource)) mkdir (root_dir . $resource, 0755);
        $textdate = date("mdy", $date);
        if (file_exists(root_dir . $resource . "/" . $textdate . ".csv")) {

            $dayinfo = readkeyedfile(root_dir . $resource . "/" . $textdate . ".csv",
                            "",
                            array(),
                            array(),
                            array(),
                            array("hour","priority","usn","signup","length"));

            //round hour and length to signup interval
            foreach ($dayinfo as $index => $signup) {
                $dayinfo[$index]['hour']   = floor($dayinfo[$index]['hour']/signup_interval)*signup_interval;
                $dayinfo[$index]['length'] = ceil($dayinfo[$index]['length']/signup_interval)*signup_interval;
            }

        } elseif (recurring_enabled && file_exists(root_dir . $resource . "/defaults.csv")) {

            $autosignups = readkeyedfile(root_dir . $resource . "/defaults.csv",
                            "",
                            array(),
                            array(),
                            array(),
                            array("datekey","value","hour","length"));

            $matchdate = false;        //if we match a "match this date" entry, ONLY do that type of entry (special day)
            if (count($autosignups)>0)
                foreach ($autosignups as $recurringindex=>$one)
                    if ((!$matchdate | $one['datekey']=="n/j/y") 
                        & date($one['datekey'],$date) == $one['value']
                        & ($one['enddate']=='' || $one['enddate']>=$date) 
                        & ($one['startdate']=='' || $one['startdate']<=$date)) {
                        //if we have a match on this signup
                        // round hour and length to signup interval
                        $one['hour']   = floor($one['hour']/signup_interval)*signup_interval;
                        $one['length'] = ceil($one['length']/signup_interval)*signup_interval;
                        if ($one['datekey'] == 'n/j/y') {
                                $matchdate = true;
                        }
                        if ($one['usn']=="") $one['usn'] = 0;
                        $one['recurring'] = True;

                        //using that info, repeatedly fill in signups up to specified priority
                        $maxpri = $one['priority'];
                        if ($maxpri=="") $maxpri = 1;
                        for ($pri = 1; $pri<=$maxpri; $pri++) {
                            $one['priority'] = $pri;
                            $one['signup']   = -$recurringindex+$pri/10;
                            $one['resource'] = $resource;
                            $dayinfo[]       = $one;
                        }
                    }
        }
        return $dayinfo;
}
//----------------------------------------------------
// PUTDAYSIGNUPS
//Write out one days' signups
//"date".csv:  hour, priority, usn, signup, length
//$data contains fields as above
function putdaysignups ($datecode, $resource, $data){

    $date = date("mdy", $datecode);

    //remove recurring signups
    //  and store them to add in a moment
    $toadd = array();
    foreach ($data as $key=>$oneitem) if ($oneitem['recurring']==1) { // || !strcasecmp($oneitem['usn'],"0")) {
        $toadd[] = $oneitem;
        unset($data[$key]);            
    }

    $targetfile = root_dir . $resource . "/" . $date . ".csv";
    //    if (count($data) == 0)        { //empty? delete any file...
    //        if (file_exists($targetfile)) unlink($targetfile);
    //    } else {
    //        writekeyedfile($targetfile, $data, "hour");
    //    }
    //NOTE: we are now writing the file whether or not it is empty - this keeps recurring signups from
    // being recreated when manipulating a SINGLE singup on a day with recurring signups (although it
    // does mean that you can't just delete all the signups on a day to get back the recurring signups)
    writekeyedfile($targetfile, $data, "hour");
        
    //create REAL signups for recurring signups which were on this day
    detachrecurring($toadd,$datecode);

}
//----------------------------------------------------
// DETACHRECURRING
function detachrecurring($toadd,$datecode) {

    //create REAL signups for recurring signups which were on this day
    if (!empty($toadd)) foreach ($toadd as $item) {
        $item['starttime'] = $datecode + $item['hour']*60*60;
        $item['endtime'] = $item['starttime'] + $item['length']*60*60;

        $signup = addsignup($item);
    }

}
//----------------------------------------------------
// GETSIGNUPPRIORITY
//Locate the priority for a given signup for a given user
// returns zero if we can't find the signup
function getsignuppriority ($usn, $signup){

        $info = getusersignups ($usn,false);

        if (count($info) > 0) $info = $info[$signup];                //get signup from all the signups

        if (count($info) > 0) {
            $dayinfo = getdaysignups($info['starttime'],$info['resource']);

            //go through day's info until you find this signup and it's associtated priority
            if (count($dayinfo)>0)
                foreach ($dayinfo as $onesignup)
                    if ($onesignup['usn'] == $usn & $onesignup['signup'] == $signup) return ($onesignup['priority']);
        }

        return (0);        //signup not found? pass back zero
}
//----------------------------------------------------
// NEWDAY
//Create empty day
function newday() {
        for ($priority = 1; $priority <= num_signups; $priority++) $block[$priority] = 1;
        for ($hour = day_start; $hour <= (day_end - hour_interval); $hour += hour_interval) $day["$hour"] = $block;
        return ($day);
}
//----------------------------------------------------
// GETDAY
//Create one day's template (Add a signup for each indicated hour and priority)
//<IN> $date (timecode), $resource
function getday ($date, $resource){
        $day = newday();
        $signups = getdaysignups($date, $resource);

        //insert any signups into the appropriate slot
        if (count($signups) > 0)
            foreach ($signups as $signup) {
                $myhour            = "" . floor(($signup['hour']-day_start)/hour_interval)*hour_interval+day_start;
                if ($myhour < min(array_keys($day)) ) $myhour = min(array_keys($day));
                $signup['length'] = "" . $signup['length'] + $signup['hour'] - $myhour;
                $day["" . $myhour][$signup['priority']] = $signup;
            }

        // now go through all hours and fill in zeros for taken time slots
        foreach ($day as $myhour=>$hourinfo)
        foreach ($hourinfo as $priority=>$signup)
        if (is_array($signup)) {
            $length = floor($signup['length']/hour_interval)*hour_interval;
            for ($rowspan = 1; ($rowspan*hour_interval) < $length; $rowspan ++) {
                $ind = "" . ($myhour + $rowspan*hour_interval);
                if ($day[$ind][$priority] == 1) {
                    $day[$ind][$priority] = 0;
                } else {  // oh crap! an overlapping signup? cut the upper one short
                        $day[$myhour][$priority]['length'] = $rowspan*hour_interval;
                        break;
                }
            }
            $day[$myhour][$priority]['rowspan'] = $rowspan;
        }

        return $day;
}

//----------------------------------------------------
// COMBINEOPEN
//Run through a day array and combine adjacent open
// priority blocks in an hour's slot
function combineopen($day) {
        foreach ($day as $hour => $block) {
                for ($priority = 1; $priority <= num_signups; $priority++) {
                        if (!is_array($block[$priority]) & $block[$priority]>0) {
                                $width = $block[$priority];
                                for ($subp = $priority+1; $subp <= num_signups; $subp++) {
                                        if (is_array($block[$subp]) | $block[$subp]==0) break;
                                        $width        = $width + $block[$subp];
                                        $block[$subp] = 0;
                                }
                                $block[$priority] = $width;
                        }
                }
                $day[$hour] = $block;
        }
        return ($day);
}
//----------------------------------------------------
// FINDPRIORITY
//Determine priority useable for given signup
function findpriority ($starttime,$endtime,$resource,$maxpriority = num_signups,$forcepriority = False){

        if ($maxpriority <= 0) $maxpriority = num_signups;

        $startinfo = getdate($starttime);
        $endinfo   = getdate($endtime);

        if (!$forcepriority)
            $priority = 1;                                //start assuming priorty 1 (or whatever is in $minpriority)
        else
            $priority = $maxpriority;                //we're ONLY allowed to select $maxpriority

        $overlaps = findoverlaps(array('starttime'=>$starttime, 'endtime'=>$endtime, 'priority'=>$maxpriority, 'resource'=>$resource));
        if (count($overlaps) == 0) return($priority);

        if (automanage_alternates) {
            //locate highest-numbered priority we can get for this slot (don't go under someone else)
            foreach ($overlaps as $item) {
                if ($item['priority'] >= $priority)
                    $priority = $item['priority']+1;
            }
            if ($priority > $maxpriority)
                $priority = num_signups+1;
        } else {
            //locate lowest-numbered OPEN priority we can get for this slot
            $allslots = array();
            foreach ($overlaps as $item) $allslots[$item['priority']] = 1;
            for ($open = $priority; $open<=$maxpriority; $open++)
                if ($allslots[$open]=="") return($open);  //take first open slot
            $priority = num_signups+1;                
        }
        return ($priority);

}
//----------------------------------------------------
// FINDOVERLAPS
// Find other signups which are overlapping (higher or lower) in given time range
// $data should contain: starttime, endtime, priority, resource
// $higher defines looking for higher priorities or lower ones
// RETURNS: array of arrays which include: usn, signup, priority
function findoverlaps ($data, $higher = false, $overlap = array() ) {

        $data['starthour'] = date("G",$data['starttime']);
        $data['endhour']   = date("G",ceil($data['endtime']/60/60)*60*60);        //round UP to nearest hour

        $newoverlap   = array();

        $ndays     = ceil(($data['endtime']-21600)/24/60/60) - floor(($data['starttime']-21600)/24/60/60);
        for ($day = 0; $day < $ndays; $day++) {
            $signups   = getdaysignups($data['starttime']+$day*24*60*60,$data['resource']);
            if (count($signups)>0)         //any signups to compare for the given day?
                foreach ($signups as $index => $signup) {

                    $signupdetails = array();
                    //see if we can get the details from the user's file
                    if ($signup['usn'] > 0) {
                        //read in details on this signup
                        $signupdetails = getusersignups($signup['usn'],false);
                        $signupdetails = $signupdetails[$signup['signup']];
                    }
                    //if not, fake them based on the day's file (might be auto-entry)
                    if (count($signupdetails)==0) {
                        $signupdetails['starttime'] =
                            strtotime(date("m/d/y",$data[starttime]+$day*24*60*60))
                             + $signup['hour']*60*60;
                        $signupdetails['endtime'] = $signupdetails['starttime'] + $signup['length']*60*60;
                        $signupdetails['usn']     = $signup['usn'];
                        $signupdetails['signup']  = $signup['signup'];
                    }

                    //check for overlap
                    if (($signupdetails['starttime'] < ($data['endtime'])) &&
                        ($signupdetails['endtime']) > $data['starttime'] &&
                       (($signupdetails['signup'] != $data['signup']) || ($signup['usn'] != $data['usn']))) {
                            if ($higher & $signup['priority'] >= $data['priority'])
                                $newoverlap[] = array('usn'=>$signup['usn'], 'signup'=>$signup['signup'], 'priority'=>$signup['priority']);
                            elseif (!$higher & $signup['priority'] <= $data['priority'])
                                $newoverlap[] = array('usn'=>$signup['usn'], 'signup'=>$signup['signup'], 'priority'=>$signup['priority']);
                    }
        }   }

        //find TRUELY new overlap items
        if (count($newoverlap)>0) foreach ($newoverlap as $index => $signup) {        //did we have this new overlap item already?
                $addit = True;                                //assume we didn't
                if (count($overlap)>0) foreach ($overlap as $known)
                        if ($known['usn'] == $signup['usn'] & $known['signup'] == $signup['signup']) {
                                $addit = False;                //we did have it, don't add this one
                                break;
                        }
                if ($addit) $overlap[] = $signup;                //add this one
                else unset($newoverlap[$index]);                  //otherwise dump it
        }

        if (count($newoverlap)>0) {
                //if looking at HIGHER overlaps, test each for additional overlaps b/c of the new ones
                if ($higher) foreach ($newoverlap as $signup) $overlap = findoverlaps ($signup,$higher,$overlap);
        }

        return ($overlap);
}
//----------------------------------------------------
// PROMOTE
//Try to promote the given signups
function promote ($signups, $echopromotions = true) {

    if (count($signups) > 0) {
        foreach ($signups as $item) {
            $usersignups            = getusersignups($item['usn'],false);
            $thissignup             = $usersignups[$item['signup']];
            $thissignup['usn']      = $item['usn'];
            $thissignup['priority'] = $item['priority'];
            $allsignups[]           = $thissignup;
        }

        $userspromoted = array();
        $myloop = 0;
        while (count($allsignups) > 0 & $myloop < 20) {
            $myloop++;
            $searchpriority = num_signups+5;
            foreach ($allsignups as $item) if ($item['priority'] < $searchpriority) $searchpriority = $item['priority'];
            foreach ($allsignups as $i=>$item) {
                if ($item['priority'] == $searchpriority) {                         //this item matches this priority?
                    $newpriority = findpriority($item['starttime'],$item['endtime'],$item['resource'],$searchpriority-1);
                    if ($newpriority < $searchpriority) {                                //got a better priority?
                        $newoverlaps = deletesignup($item);                        //delete old signup
                        $item['priority'] = $newpriority;
                        addsignup($item);                                        //add as new signup
                        unset($allsignups[$i]);                                          //done with this one, erase it from allsignups
                        message("promotion",$item);                                //Report promotion to user

                        $userspromoted[] = $item['usn'];                        //make note of this usernumber for report

                        foreach ($newoverlaps as $ii => $newitem) {                //do we have these overlaps already?
                            $matches = false;
                            foreach ($allsignups as $olditem)                         //compare to existing signups
                                if ($olditem['usn'] == $newitem['usn'] & $olditem['signup'] == $newitem['signup']) {
                                    $matches = true;
                                    break;
                                }
                            if ($matches) unset($newoverlaps[$ii]);                          //had this, skip it
                            else {
                                $usersignups       = getusersignups($newitem['usn'],false);
                                $thissignup        = $usersignups[$newitem['signup']];
                                $thissignup['usn'] = $newitem['usn'];
                                $thissignup['priority'] = $newitem['priority'];
                                $allsignups[]      = $thissignup;
                            }
                        }
                        break;                                //break out of allsignups foreach, start again
            }   }   }
        }
        if ($echopromotions & echo_promotion & automanage_alternates & count($userspromoted)>0) {
            echo '<table border=1 bgcolor="#eeffee"><tr><td><b><font color="#990000">This change promoted the folowing user(s). Please contact:</b><br></font>';
            echo "<ul>\n";
            foreach ($userspromoted as $usn) listusercontact($usn);                //list contact info for user just promoted
            echo "</ul></td></tr></table>\n";
        }
    }   
}
//----------------------------------------------------
// DEMOTE
// attempt to demote all the signups overlapping
// the range of the given parameters
// $data should contain: starttime, endtime, priority, resource
// RETURNS: true if signups could be demoted, false if not
// WARNING: Demote will actually allow signups to be demoted beyond num_signups!
function demote($data, $echodemotions = True) {

        $overlaps = findoverlaps($data,true);                //find higher overlaps
        // overlaps is array of arrays which include: usn, signup, priority

        $usersdemoted = array();
        if (count($overlaps)>0) {
            //work backwards updating each signup
            for ($testpriority = num_signups; $testpriority >0; $testpriority--)
                foreach ($overlaps as $item) {
                    if ($item['priority'] == $testpriority) {
                        $usersignups = getusersignups($item['usn'],false);
                        if (count($usersignups[$item['signup']])>0) foreach ($usersignups[$item['signup']] as $key=>$value) $item[$key]=$value;
                        $item['resource']   = $data['resource'];
                        $item['priority']   = $item['priority']+1;
                        $item['thisuserid'] = $data['thisuserid'];
                        $usersdemoted[]     = $item['usn'];                        //make note of this usernumber for report
                        message("demotion",$item);                                //Report demotion to user
                        update($item,"",$testpriority+1,True);                //update QUIETLY (no notice)
                }
            }
        }
        if ($echodemotions & echo_demotion & automanage_alternates & count($usersdemoted)>0) {
            echo '<table border=1 bgcolor="#eeffee"><tr><td><b><font color="#990000">This change demoted the folowing user(s). Please contact:</b><br></font>';
            echo "<ul>\n";
            foreach ($usersdemoted as $usn) listusercontact($usn);        //list contact info for user just demoted
            echo "</ul></td></tr></table>\n";
        }
        return (true);
}
//----------------------------------------------------
// ADDSIGNUP
//Actually add signup to day and user data files
// data contains: resource, starttime, endtime, usn, comment, priority
function addsignup ($data) {

        if (!in_array("starttime",array_keys($data)) | !in_array("endtime",array_keys($data))
                | !in_array("resource",array_keys($data)) | !in_array("usn",array_keys($data)))
                return (0);                //can't do signup

        $usersignups = getusersignups($data['usn'],false);
        if (count($usersignups) > 0) $signupnumber = max(array_keys($usersignups))+1;
        else $signupnumber = 1;


        //add to user file (see also "reconcilecalendar" which also writes this same info on restore)
        $usersignups[$signupnumber] = array(
                "signup"    => $signupnumber,
                "resource"  => $data['resource'],
                "starttime" => $data['starttime'],
                "endtime"   => $data['endtime'],
                "status"    => $data['status'],
                "checkout"  => $data['checkout'],
                "checkin"   => $data['checkin'],
                "comment"   => $data['comment']);
        putusersignups($data['usn'],$usersignups);

        //add to day files
        $startinfo = getdate($data['starttime']);
        $endinfo   = getdate($data['endtime']);

        //calculate starting & ending dates in day units
        $startday = strtotime(date("m/d/Y",$data['starttime']))/60/60/24;
        $endday   = strtotime(date("m/d/Y",$data['endtime']))/60/60/24;

        //remove from day files noting any overlapping signups at higher priority
        $ndays     = ($endday - $startday);

        for ($day = 0; $day <= $ndays; $day++) {
                if ($day == 0) $hour = $startinfo['hours']+$startinfo['minutes']/60;
                else           $hour = 0;//0;//day_start;!!!
                if ($day == $ndays) $length = $endinfo['hours']+$endinfo['minutes']/60 - $hour;
                else                      $length = 24-$hour;//24-$hour;//day_end - $hour;!!!

                if ($length>0) {
                        $signups   = getdaysignups(($startday+$day)*24*60*60,$data['resource']);
                        $signups[] = array(
                                "hour"     => $hour,
                                "priority" => $data['priority'],
                                "usn"      => $data['usn'],
                                "signup"   => $signupnumber,
                                "length"   => $length);
                        putdaysignups(($startday+$day)*24*60*60,$data['resource'],$signups);
                }
        }

        recordactivity($data['usn']);    //Reset user inactivity timer
        return ($signupnumber);
}
//----------------------------------------------------
// DELETESIGNUP
//Actually remove signup from day and user data files
// returns list of signups which were affected by the
// deletion
// REQUIRES in data: usn, signup
function deletesignup ($data, $updatedelete=false, $skiprestricted=false, $updatedstarttime="", $updatedendtime="", $updatedusn="") {
        $usersignups       = getusersignups($data['usn'],false);
        $mainsignup        = $usersignups[$data['signup']];
        $users             = getusers();
        $resources         = getresources();

        //check if this account is restricted or expired
         $restrictedorexpired = ($users[$data['usn']]['restricted'] || (strcmp($users[$data['usn']]['expire'], "Expired")==0));
         $doublebook = $resources[$mainsignup['resource']]['doublebook'];

        //Only check other signups for deletion if this account is restricted or expired
        if (($restrictedorexpired)&&($doublebook)&&(!$skiprestricted)) {
             $usersignups2 = $usersignups;
        }else{
             unset($usersignups2);
             $usersignups2[] = $mainsignup;
        }

        //delete the signup(s)
        $alloverlaps = array();
        if ($updatedstarttime != "")
             $checksignup['starttime']=$updatedstarttime;
        else
             $checksignup['starttime']=$mainsignup['starttime'];

        if ($updatedendtime != "")
             $checksignup['endtime']=$updatedendtime;
        else
             $checksignup['endtime']=$mainsignup['endtime'];
        if ($updatedusn == "") $updatedusn = $data['usn'];

        if (count($usersignups2) > 0) foreach ($usersignups2 as $usersignup){
             $signupnumber = $usersignup['signup'];
             $thissignup   = $usersignup;
             $thissignup['usn'] = $data['usn'];
             $overlap = array();

             //Is this alternate signup affected by the deletion of the main signup?
             $illegalsignup = 0;
             if ($thissignup['signup'] == $mainsignup['signup'])
                  $illegalsignup = 1;
             elseif ((($thissignup['starttime'] >= $mainsignup['starttime']) && ($thissignup['endtime'] <= $mainsignup['endtime'])) &&
                     (($thissignup['starttime'] < $checksignup['starttime']) || ($thissignup['endtime'] > $checksignup['endtime']) ||
                      ($thissignup['usn'] != $updatedusn) || ($updatedelete==false)))
                  $illegalsignup = 1;

             //delete the signup
             if (($illegalsignup)&&(count($thissignup)>0)) {
                     //remove from user file
                     unset($usersignups[$signupnumber]);

                     //calculate starting & ending dates in day units
                     $startday = strtotime(date("m/d/Y",$thissignup['starttime']))/60/60/24;
                     $endday   = strtotime(date("m/d/Y",$thissignup['endtime']))/60/60/24;

                     //remove from day files noting any overlapping signups at higher priority
                     $ndays     = ($endday - $startday);
                     for ($day = 0; $day <= $ndays; $day++) {
                             $signups   = getdaysignups(($startday+$day)*24*60*60,$thissignup['resource']);
                             if (count($signups)>0) {
                                     foreach ($signups as $index => $signup)                //is this our signup? then remove it
                                             if ($signup['usn'] == $thissignup['usn'] & $signup['signup'] == $thissignup['signup']) {
                                                     $thissignup['priority'] = $signup['priority'];
                                                     $thissignup['starthour'] = $signup['hour'];
                                                     $thissignup['endhour']   = $signup['hour'] + $signup['length'] - 1;
                                                     unset($signups[$index]);
                                                     break;
                                             }
                                     foreach ($signups as $index => $signup) $overlap = findoverlaps($thissignup,true,$overlap);
                                     putdaysignups(($startday+$day)*24*60*60,$thissignup['resource'],$signups);
                             }
                     }
                     }
             //combine overlaps for all signups deleted
             $alloverlaps = array_merge($alloverlaps,$overlap);
        }
        //Re-write the user file that now includes the updates
        putusersignups($data['usn'],$usersignups);
        return($alloverlaps);
}
//----------------------------------------------------
// UPDATE
// update an entry (if it doesn't make it impossible)
// IN: $data must contain: usn, signup (for old signup),
//         and also: starttime, endtime, resource (for NEW signup)
//  (if any of the last three are missing, the current values will be used)
//   $newusn is a new user number to assign (reassign signup)
//   $forcepriority is the new priority to force (instead of "best available")
//   $quiet is a flag to keep from sending e-mails to users
// returns zero if update successful, otherwise, returns
// signupnumber of re-assigned signup (no changes except signup number)
function update ($data, $newusn = "", $forcepriority = "", $quiet = false) {

        $usersignups               = getusersignups($data['usn'],false);
        $oldsignup                 = $usersignups[$data['signup']];
        $oldsignup['usn']          = $data['usn'];
        $oldsignup['signup']       = $data['signup'];
        $oldsignup['thisuserid']   = $data['thisuserid'];  //copy for appropriate e-mail messages

        //copy missing data fields from oldsignup
        if ($data['starttime'] == "") $data['starttime'] = $oldsignup['starttime'];
        if ($data['endtime'] == "")   $data['endtime']   = $oldsignup['endtime'];
        if ($data['resource'] == "")  $data['resource']  = $oldsignup['resource'];
        if ($data['checkout'] == "")  $data['checkout']  = $oldsignup['checkout'];
        if ($data['checkin'] == "")   $data['checkin']   = $oldsignup['checkin'];

        $signups = getdaysignups($oldsignup['starttime'],$oldsignup['resource']);

        if (count($signups) != 0)        // locate original priority for the signup to be changed
                foreach ($signups as $index => $signup)
                        if ($signup['usn'] == $oldsignup['usn'] & $signup['signup'] == $oldsignup['signup']) {
                                $oldsignup['priority'] = $signup['priority'];
                                break;
                        }

        //Don't delete other signups if this is a valid update for a restricted user
        if (($newusn != $oldsignup['usn']) || ($data['starttime'] > $oldsignup['starttime']) || ($data['endtime'] < $oldsignup['endtime']))
              $skiprestricted = false;
        else
              $skiprestricted = true;
        $affected = deletesignup($data, true, $skiprestricted, $data['starttime'], $data['endtime'], $newusn);                //Delete original

        if ($forcepriority == "") {                        //Not trying to force some priority? see whats available
                if ($oldsignup['resource'] == $data['resource'])
                        $priority  = findpriority ($data['starttime'],$data['endtime'],$data['resource'],$oldsignup['priority']);
                else {
                        $priority  = findpriority ($data['starttime'],$data['endtime'],$data['resource']);
                        if ($priority > $oldsignup['priority']) $priority = num_signups+1;
                }
                $forcepriority = 0;
        } else        {                                        //given a specific priority, see if it is available
                $priority  = findpriority ($data['starttime'],$data['endtime'],$data['resource'],$forcepriority, True);
        }

        $data['priority']               = $priority;
        if ($newusn != "") $data['usn'] = $newusn;
        $warning = array();
        
        if ($priority <= $oldsignup['priority'] || $priority == $forcepriority) {        //don't allow WORSE priority (unless forced)
                $newsignup = addsignup ($data);             //add new signup                                                

                //if this update caused a rule violation and we're hard-enforceing those, give warning and reject the update
                $users = getusers();
                if (!checksec($users[$data['thisuserid']]['sec'],10) & hard_enforce_rules) {
                        $ruletest = validatesignups($data['usn'],false,$data['signup']);
                        if (count($ruletest) > 0) {
                                //start by removing the newly-made signup (we'll re-add the old one in a moment)
                                deletesignup(array('usn'=>$data['usn'],'signup'=>$newsignup),true,true);
                                //get the warning messages which are relevent
                                $warning[] = "Sign-up causes a rule violation (or others already exist).";
                                $warning   = array_merge($warning,$ruletest);
                        }
                }
        
        } else {  //can't get same or better priority  

                if ($forcepriority > 0) { //try DEMOTING overlapping items if forcepriorty is set, otherwise don't update
                        $data['priority'] = $forcepriority;
                        demote($data);
                        $newsignup = addsignup ($data);
                        if (!$quiet) {
                                if ($data['thisuserid'] != $data['usn']) {
                                        if ($data['usn'] != $oldsignup['usn']) message("asuser",$data);
                                        else         message("update",$data);
                                }
                                if ($data['thisuserid'] != $oldsignup['usn'])
                                        if ($data['usn'] != $oldsignup['usn']) message("superdelete",$oldsignup);
                        }
                        promote ($affected);                                //promote effected items
                        return (array("success"=>true, "signup"=>$newsignup, "warning"=>$warning));
                } else {
                    $warning[] = "Sign-up overlaps other sign-ups.";
                }

        }

        if (count($warning)>0) {    //can't update signup... fail and return old signup with its new signup number (other details the same)
        
            return (array("success"=>false, "signup"=>addsignup ($oldsignup), "warning"=>$warning));

        } else {  //no warning messages, finish up normally

            if (!$quiet) {
                    if ($data['thisuserid'] != $data['usn']) {
                            if ($data['usn'] != $oldsignup['usn']) message("asuser",$data);
                            else         message("update",$data);
                    }
                    if ($data['thisuserid'] != $oldsignup['usn'])
                            if ($data['usn'] != $oldsignup['usn']) message("superdelete",$oldsignup);
            }
            promote ($affected);                                //promote effected items
            return (array("success"=>true, "signup"=>$newsignup, "warning"=>$warning));
        }
        
}
//----------------------------------------------------
// READACTIONLOG
//read in action log
// Each entry has two elements: 'date' and 'data'
//  date is the time and date the entry was made
//  data is all of the various relevant HTTP_POST_VARS plus
//    one entry ['action'] which contains the textual event
function readactionlog() {
        $items = array();
        if (file_exists(actionlog_name)) {
                $fp = fopenlock (actionlog_name, "r");
                while ($oneline = fgetcsv_trim ($fp, 1000, ","))  {
                        $subitems = array();
                        for ($i=0; $i<=count($oneline); $i=$i+2)
                                if ($oneline[$i] != "") $subitems[$oneline[$i]] = $oneline[$i+1];
                        $items[] = array('date' => $subitems['date'], 'data' => $subitems);
                }
                fclose ($fp);
        }
        return ($items);
}

//----------------------------------------------------
// APPENDTOACTIONLOG
// Actually append an action log entry to the logfile
function appendToActionLog($item) {

    $success = true;   //assume the best

	$fp = fopenlock (actionlog_name, 'a');    
    if ($fp) {
    	fwrite ($fp, "date, " . $item['date']);
    	unset($item['data']['date']);                                //write it back to the log (except date which we did above)
    	foreach ($item['data'] as $key => $subitem) {
    		if (!empty($key)) {
    			fwrite ($fp, ", " . $key . ", " . str_replace(","," ",$subitem));
    		}
    	}
    	fwrite ($fp, "\n");
    	fclose ($fp);

	} else {
	    $success = false;
    }
    
    return($success);
}

//----------------------------------------------------
// LOGACTION
// Build an item to append to the action log and send it to
// appendToActionLog() to be written out
// if called with NO action, we test for dailyactions having been performed
function logaction($action = "",$thisuserid = "",$data = "") {
	
	clearstatcache();		// Make sure we get the real file modification date in a moment
    if (empty($action)) { //ONLY do this if no actions input (ONLY when forced)

    	// First find out if this is the first logged action of the day
            if (!file_exists(actionlog_name) || (file_exists(actionlog_name) && (filemtime(actionlog_name) < strtotime(date("m/d/y",zulutime()))))) {
    		//add report of daily actions (keeps subsequent calls to logaction during
    		// dailyactions from activating daily actions themselves)$items = readactionlog();

    		$item = array('date' => date(dateformat . "/y " . timeformat,zulutime()+timezone), 'data' => array('action' => 'Daily Actions'));

    		if (appendToActionLog($item)) {

            	clearstatcache();		// Make sure we get the real file modification date on the next line
    		    require_once('maintfunctions.php');
        		dailyactions();

    		} else {  //append to action log failed - do NOT do daily actions. Fail now... do nothing else

                scheduleunlock();
                echo "<h3>FATAL ERROR</h3><b>Sorry, the scheduler cannot write to its 'action log' file. Daily maintenance could not be performed and the scheduler must shut down - please contact system administrator.</b>";
                trigger_error ("Cannot write to action log. Daily maintenance could not be performed.", E_USER_ERROR);

    		}
    	}
	
    } else {
    
        //add our new item
        $data['action']     = $action;
        $data['thisuserid'] = $thisuserid;

        if (!actionlog_details) {
            $users = getusers();
            $resourcelist = getresources();
            if ($data['action'] == "Bad Password") {
                $action = $data['action'] . " for " . $data['lastname'];
            } elseif (in_array(strtolower($data['action']),
                    array("deleted authorized user","added authorized user","updated authorized user","restore from backup","updated configuration","assigned usn","deleted user"))) {
                $action = $users[$data['thisuserid']]['username'] . " " . $data['action'] . " <b>" . $data['username'] . "</b>";
                if (in_array("sec", array_keys ($data))) $action .= " SEC = " . $data['sec'];
                if (in_array("usn", array_keys ($data))) $action .= " USN = " . $data['usn'];
            } else {
                if ($data['thisuserid'] != $data['usn'] && !empty($data['usn'])) {
                    $targetuser = $users[$data['usn']]['username'];
                    $action = $users[$data['thisuserid']]['username'] . " " . $data['action'] . " <b>" . $targetuser . "</b>";
                } elseif ($data['thisuserid'] != "") {
                    $action = $users[$data['thisuserid']]['username'] . " " . $data['action'] . " self";
                }
                $action .=  ' ' . $resourcelist[$data['resource']]['short'];
                if (in_array("sec", array_keys ($data))) $action .=  " SEC = " . $data['sec'];
                if (in_array("starttime", array_keys ($data)) & in_array("endtime", array_keys ($data)))
                        $action .=  ", " . date(dateformat . " " . timeformat,$data['starttime']) . "-" . date(dateformat . " " . timeformat,$data['endtime']);
                if (in_array("priority", array_keys ($data)) && $data['priority']>0) $action .=  ", priority " . $data['priority'];
                if (in_array("signup", array_keys ($data))) $action .= " (signup " . $data['signup'] . ")";
            }
            $data = array('action'=>$action);
        }

        $item = array('date' => date(dateformat . "/y " . timeformat,zulutime()+timezone), 'data' => $data);
    
        appendToActionLog($item);
    }
}
//----------------------------------------------------
// DECODESIGNUP
//creates a string representing a single signup
//IN: $item  : array of signup info including usn, starttime, endtime, signup, comment, priority
//    $label : label for signup (usually either a name or a resource)
//    $html  : boolean flag to indicate if this should be in HTML or plain-text
function decodesignup($item,$label="",$html=true) {

        $strng = "";
        $strng .= date(dateformat . "/y (D)",$item['starttime']);

        if ($label == "") {
                $strng .= " from ";
        } else {
                if ($html) $strng .= "&nbsp;&nbsp;-&nbsp;&nbsp;"; else $strng .= "  -  ";

                $strng .= $label;

                if ($html) $strng .= "&nbsp;&nbsp;-&nbsp;&nbsp;"; else $strng .= "  -  ";
        }

        $strng .= date(timeformat,$item['starttime']);
        if (date("D " . dateformat,$item['starttime']) == date("D " . dateformat,$item['endtime']))
                $strng .= " to " . date(timeformat,$item['endtime']);
        else
                $strng .= " to " . date("D " . dateformat . " " . timeformat,$item['endtime']);
        $length = ($item['endtime'] - $item['starttime'])/60/60;
        $strng .= " (";
        if ($length >= 24 & $length < 48) {
                $strng .= floor($length / 24) . " day";
                $length = ($length % 24);
                if ($length > 0) $strng .= ", ";
        } elseif ($length >=48) {
                $strng .= floor($length / 24) . " days";
                $length = ($length % 24);
        if ($length > 0) $strng .= ", ";
        }
        if ($length == 1)
                $strng .= "1 hour)";
        elseif ($length > 0)
                $strng .= sprintf("%1.1f",$length) . " hours)";
        else        $strng .= ")";

        if ($item['priority']=="") 
            $priority = getsignuppriority ($item['usn'], $item['signup']);
        else
            $priority = $item['priority'];
        if ($priority > 1 && automanage_alternates) {
                if ($label == "") $strng .= " as";
                $strng .= " alternate #" . ($priority-1);
        }
        if ($item['comment'] != "") {
                if ($html) $strng .= "</a><br>&nbsp;&nbsp;&nbsp;<b>";
                else $strng .= "   ";
                $strng .= comment_text;
                if ($html) $strng .= "</b>";
                $strng .= ": " . stripcslashes($item['comment']);
        }
        if ($item['checkout'] != "") {
            if ($html) $strng .= "&nbsp;&nbsp;&nbsp;<b>";
            else $strng .= "    [";
            $strng .= "Actual In/Out";
            if ($html) $strng .= "</b>";
            $strng .= ": " . date(dateformat . " " . timeformat,$item['checkout']);
            if ($item['checkin']=="")
                $strng .= " - no recorded end";
            elseif (date("D " . dateformat,$item['checkout']) == date(dateformat,$item['checkin']))
                $strng .= " to " . date(timeformat,$item['checkin']);
            else
                $strng .= " to " . date(dateformat . " " . timeformat,$item['checkin']);
            if (!$html) $strng .= "]";
        }

        return ($strng);
}
//----------------------------------------------------
// VALIDATESIGNUPS
//makes sure the signups for the given user are valid, gives warning box if not
//IN: $usn : user number for whom signups should be validated
//    $out : true to echo violations, false to only return the violations in an
//            array (false can be used to test for violation without displaying them)
//    $specificsignup : test if a given signup causes a broken rule
function validatesignups($usn,$out = true,$specificsignup = Null) {

        $signups      = getusersignups($usn,true);
        $resourcelist = getresources();
        $users        = getusers();

        //do not validate users defined as exempt
        $do_not_validate = explode(",", exempt_users);   //users which can not be validated
        unset($valtmp);
        foreach ($do_not_validate as $valline) $valtmp[]=trim($valline);
        $do_not_validate = $valtmp;
        if (in_array($users[$usn]['firstname'] . $users[$usn]['lastname'], $do_not_validate))
                return (array());        //don't check signups for these users

        if (count($signups) == 0) return (array());
        $violations = array();

        //get priorities for all of this user's signups
        foreach ($signups as $signup => $item)
                $signups[$signup]['priority'] = getsignuppriority($usn,$signup);


        //combine adjacent signups
        foreach ($signups as $signup1 => $item1) {
            $changed = true;                //get started
            while ($changed) {
                $changed = false;        //assume we won't have to do this loop again
                foreach ($signups as $signup2 => $item2)
                if ($item1['endtime']==$item2['starttime']
                  & $item1['priority']==$item2['priority']
                  & $item1['resource']==$item2['resource']) {
                    if (!allow_backtoback_signups) {
                        $violations[] = 'Back-to-Back signups not allowed (' . $resourcelist[$item1['resource']]['short'] . ' at ' . date('D ' . dateformat . ' ' . timeformat,$item1['starttime']) . ' and ' . date('D ' . dateformat . ' ' . timeformat,$item2['starttime']) . ').';
                    } else {
                        $item1['endtime'] = $item2['endtime'];                      //use new endtime
                        $item1['comment'] .= $item2['comment'];                     //append comment
                        $signups[$signup1] = $item1;                                //copy into permanent record
                        if ($specificsignup==$signup2) $specificsignup=$signup1;    //re-point specificsignup to the new # (if it was = signup2)
                        unset($signups[$signup2]);                                  //clear second signup
                        $changed = true;                                            //and start inner loop again
                        break;
                    }
                }
            }
        }


        //are they ALLOWED to signup?
        if (!checksec($users[$usn]['sec'], 1) & count($signups) > 0)
                $violations[] = '<b>WARNING: Sign-Ups Not Authorized But Exist!</b>';

        //test for maximum signup length
        foreach ($signups as $signup => $item) {
                if (($specificsignup=="" || $signup==$specificsignup) &&
                        (($item['endtime']-$item['starttime']) > longest_signup))
                        $violations[] = 'Signup for ' . $resourcelist[$item['resource']]['short'] . ' on ' . date('D ' . dateformat . ' ' . timeformat,$item['starttime']) . ' is longer than the longest permitted signup time.';
        }

        // No overlap on different resources OR same resource
        $overlapped = False;
        if (!allow_doublebook && (hard_enforce_doublebook | $specificsignup=="")) foreach ($signups as $signup1 => $item1) {
                if ($specificsignup=="" | $signup1==$specificsignup) {
                        foreach ($signups as $signup2 => $item2)
                               if ($item1['starttime'] <  $item2['endtime']
                                 & $item1['endtime']   >  $item2['starttime']
                                 & $signup1 != $signup2
                                 & $resourcelist[$item1['resource']]['doublebook'] != TRUE
                                 & $resourcelist[$item2['resource']]['doublebook'] != TRUE ) {
                                       $overlapped = True;
                                       break;
                                 }
                        if ($overlapped) break;                //dont compare any more.. got at least one
                }
        }
        if ($overlapped) $violations[] = errormsg_doublebooked;

        //NOTE: we don't test this if we're testing just ONE signup and this isn't a fatal violation
        //any signups must include a comment or overnight signups must include a comment
        if ($specificsignup==""  || missing_comment_fatal) 
            foreach ($signups as $signup => $item) {
                if ($specificsignup=="" || $specificsignup==$signup) {
                    //start and end days are different?
                    if ($item['comment']=="" && require_comment)
                            $violations[] = 'Signup for '
                                     . $resourcelist[$item['resource']]['short']
                                     . ' on '
                                     . date('D ' . dateformat . ' ' . timeformat,$item['starttime'])
                                     . ' '.errormsg_comment_required;
                    elseif (date('z',$item['starttime']) != date('z',$item['endtime']) & $item['comment']=="" & require_overnight_comment)
                            $violations[] = 'Signup for '
                                     . $resourcelist[$item['resource']]['short']
                                     . ' on '
                                     . date('D ' . dateformat . ' ' . timeformat,$item['starttime'])
                                     . ' '.errormsg_comment_missing;
                }
        }

        //Should we ignore signups which end < 24 hours off for everything below this point?
        if (!near_signup_violations) {
                foreach ($signups as $signup => $item)
                        if ($item['endtime'] < (zulutime() + timezone + allow_signup_offset + 24*60*60))
                                unset($signups[$signup]);
        }

        //1.Members may reserve an Aircraft they are authorized to operate for a maximum of
        //   X sign-ups. The following restrictions apply:

        //Test for total number of signups
        $signupcount[1] = 0;
        $signupcount[2] = 0;
        foreach ($signups as $signup => $item) {
                if (($item['priority']==1)&& (($resourcelist[$item['resource']]['doublebook']==FALSE)||count_dbook)) $signupcount[1]++;
                elseif (($resourcelist[$item['resource']]['doublebook']==FALSE)||count_dbook) $signupcount[2]++;
        }
        if ($signupcount[1] > signupcount) $violations[] = errormsg_toomany_primary;
        if ($signupcount[2]+$signupcount[1] > signupcount2) $violations[] = errormsg_toomany_total;

        //a.A single continuous reservation may not include more than X
        //   full (eight (8) hours or more) weekend days. Therefore the maximum sign-up
        //   may not exceed X days.
        //b.No more than X separate sign-ups may include full weekend days.
        //b.Sign-ups may not include more than a total of four X full weekend days.

        $weekend_tally = 0;
        $total_weekend_days = 0;
        foreach ($signups as $signup => $item) {
                if ($item['priority'] == 1) {
                        //calc # days rounded UP from day_end-8 and DOWN from day_start+8
                        $weekday  = date('w',$item['starttime']);
                        if (($item['endtime'] - $item['starttime'])/60/60 >= 8 ) {
                                $startday = floor($item['starttime']/60/60/24+19/24);
                                  if (date('G',$item['starttime'])+date('i',$item['starttime'])/60 > day_end - 8) {
                                        $startday++;    //after 15:00, count starts on next day
                                        $weekday++;
                                  }
                                  if ($weekday == 7) $weekday = 0;
                                $endday   = ceil($item['endtime']/60/60/24+19/24);
                                  if ((date('G',$item['endtime'])+date('i',$item['endtime'])/60) < (day_start + 8)) $endday--;      //before day_start+8 hours? count ends previous day
                                $ndays    = $endday - $startday;

                                $nwd = floor($ndays/7)*2;                           //two weekend days for each full week
                                $dow = $weekday-1; if ($dow == -1) $dow = 6;        //convert to monday=0

                                if ($ndays%7>0 & (($ndays%7)+$dow-1)>4) {                //and analyze left-over days too
                                        $nwd++;                                        //> 0 days left over and falls over weekend?
                                        if ((($ndays%7)+$dow-1)>5) $nwd++;        //> 2 days left over and < Sunday? add another
                                }

                                //does this one include > 3 weekend days?
                                if (($specificsignup=="" | $signup==$specificsignup) & $nwd > max_weekend_days_single) $violations[] = 'Signup for ' . $resourcelist[$item['resource']]['short'] . ' on ' . date('D ' . dateformat . ' ' . timeformat,$item['starttime']) . ' '.errormsg_maxweekend_single;

                                //check to see if it includes ANY weekend days and add to our signup total
                                if ($nwd > 0) $weekend_tally++;        //add one to # of signups with weekend days

                                //also tally total # of weekend days
                                $total_weekend_days += $nwd;
                        }
                }
        }
        if ( $weekend_tally > max_weekend_reservations )
                $violations[] = errormsg_maxweekend;
        if ( $total_weekend_days > max_weekend_days_all )
                $violations[] = errormsg_maxweekend_all;

        //Test for signups longer than allowed (as indicated in resource list)
        foreach ($signups as $signup => $item) {
                if (($specificsignup=="" | $signup==$specificsignup) &
                        (($item['endtime']-$item['starttime']) > ($resourcelist[$item['resource']]['maxsignup'])*60*60) && $resourcelist[$item['resource']]['maxsignup'] >0)
                        $violations[] = 'Signup for ' . $resourcelist[$item['resource']]['short'] . ' on ' . date('D ' . dateformat . ' ' . timeformat,$item['starttime']) . ' is over '.$resourcelist[$item['resource']]['maxsignup'] .' hours '.errormsg_maxsignup;
        }

        if (count($violations)>0 & $out) {
                echo '<table border=1 bgcolor="#ffeeee"><tr><td><b><font color="#990000">'.errormsg_general.'</b><br></font>';
                echo '<ul>';
                foreach ($violations as $item) echo '<li>' . $item . '</li>';
                echo '</ul>';
                echo "</td></tr></table>";
        }

        return ($violations);

}
//----------------------------------------------------
// LISTUSERCONTACT
//Collects a user's contact info in a useful text format
// This returned to the caller and optionally, displayed immediately
// inputs are: 
//   usn - user number to list
//   short - TRUE gives short (contact only) version
//           FALSE gives user name and contact info in <li> format
//   display - TRUE print contact info now
//             FALSE return it ONLY (allows caller to decide what to do if empty)
function listusercontact($usn,$short=FALSE,$display=TRUE) {

        $info = "";
        $users   = getusers();

        if ($users[$usn]['ratings']=='hide' && !nouserhide) return("");
        
        if ($short!=TRUE){
             $info .= "<li><b>" . $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] . "</b>";
             $info .= "<br>\n";
        }
        $temp = getcontact($usn,'emails_html');
        if ($temp != "") {
                $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>e-mail:</b> " . $temp;
                $info .= "<br>\n";
        }
        $temphome = getcontact($usn,'homephone');
        $tempwork = getcontact($usn,'workphone');
        if ($short!=TRUE){
          if ($temphome != "") {
                  $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>home phone:</b> " . $temphome;
                  $info .= "<br>\n";
          }
          if ($tempwork != "") {
                  $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>work phone:</b> " . $tempwork;
                  $info .= "<br>\n";
          }
        } else {
          if (($temphome != "") && ($tempwork != "")) $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>home phone:</b> " . $temphome."<font color='FFFFFF'>.........</font><b>work phone:</b>". $tempwork ."<br>\n";
          if (($temphome != "") && ($tempwork == "")) $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>home phone:</b> " . $temphome."<br>\n";
          if (($temphome == "") && ($tempwork != "")) $info .= "&nbsp;&nbsp;&nbsp;&nbsp;<b>work phone:</b> " . $tempwork."<br>\n";
        }
        if ($short!=TRUE) $info .= "</li>";

        if ($display) echo $info;
        return($info);
}
//----------------------------------------------------
// MESSAGE
//prepares various message to e-mail to uers
function message($mode,$data) {

$users   = getusers();
$resourcelist = getresources();

$message = "";
switch ($mode) {
case "primaries" :
        $overlap = findoverlaps($data);
        if (count($overlap)>0 && echo_prodding && automanage_alternates) {
                echo '<table border=1 bgcolor="#eeffee"><tr><td><b><font color="#990000">Please contact the following Primary(ies) to this signup to let them know there is an alternate to their signup:</b><br></font>';
                echo "<ul>\n";

                $addheaders = "From: " . email_from . "\n";

                foreach ($overlap as $item)  {  //usn, signup, priority

                        if (($item['signup'] . $item['usn']) != ($data['signup'] . $data['usn'])) {
                                listusercontact($item['usn']);

                                if (email_primaries & $item['usn']!=$data['usn']) {
                                        $usersignups       = getusersignups($item['usn'],false);
                                        $thissignup        = $usersignups[$item['signup']];
                                        $message =  "\nDear " . $users[$item['usn']]['firstname'] . " " . $users[$item['usn']]['lastname'] . ",";
                                        $message .=  "\n" . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'];
                                        $email  = getcontact($data['usn'],"emails");
                                        if ($email != "") $message .=  " (" . $email . ")";
                                        $message .= " has signed up as an alternate to your signup for " . $resourcelist[$data['resource']]['long'];
                                        $message .= " (your signup starts";
                                        $message .= " at " . date(timeformat,$thissignup['starttime']);
                                        $message .= " on " . date("D " . dateformat,$thissignup['starttime']);
                                        $message .= ").";
                                        $message .= "\nPlease contact any alternates if you cancel this signup. \nThank you!\n";
                                        $message .= "\n";
                                        $message .= " Visit " . scheduling_URL . " for additional information.\n";

                                        $email = getcontact($item['usn'],"emails");

                                        sendemail ($email, "On-Line Sign-up Message", $message, $addheaders);

                                }
                        }
                }
                echo "</ul>\n";
                echo "</td></tr></table>";
        }
        $message = "";
        break;

case "promotion" :
        if (!automanage_alternates) return(Null);
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nGood News! Your signup for " . $resourcelist[$data['resource']]['long'];
        $message .= " at " . date(timeformat,$data['starttime']);
        $message .= " on " . date("D " . dateformat,$data['starttime']);
        $message .= " has been promoted to ";
        if ($data['priority'] == 1) $message .= "a primary signup";
        else $message .= "alternate number " . ($data['priority']-1);
        $message .= ".\n";
        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

case "demotion" :
        if (!automanage_alternates) return(Null);
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nI'm sorry to inform you that your signup for " . $resourcelist[$data['resource']]['long'];
        $message .= " at " . date(timeformat,$data['starttime']);
        $message .= " on " . date("D " . dateformat,$data['starttime']);
        $message .= " has been demoted to ";
        if ($data['priority'] == 1) $message .= "a primary signup";
        else $message .= "alternate number " . ($data['priority']-1);
        $message .= " by ";
        $message .= $users[$data['thisuserid']]['firstname'] . " " . $users[$data['thisuserid']]['lastname'];
        $email  = getcontact($data['thisuserid'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";
        $message .= ".\n";
        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

case "update" :  //message that your signup was updated by another user (superuser)
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nOne of your signups has been changed by ";
        $message .= $users[$data['thisuserid']]['firstname'] . " " . $users[$data['thisuserid']]['lastname'];
        $email  = getcontact($data['thisuserid'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";
        $message .= ". The new signup is";
        $message .= " for " . $resourcelist[$data['resource']]['long'];

        $message .= " on " . decodesignup($data,"",false);
        $message .= ".\n";

        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

case "noshow" :  //message that your signup was no-showed by another user
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nYour signup ";
        $message .= " for " . $resourcelist[$data['resource']]['long'];
        $message .= " at " . date(timeformat,$data['starttime']);
        $message .= " on " . date("D " . dateformat,$data['starttime']);

        $message .= " has been 'No-Showed' (i.e. Deleted) by " . $users[$data['thisuserid']]['firstname'] . " " . $users[$data['thisuserid']]['lastname'];
        $email  = getcontact($data['thisuserid'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";
        $message .= ".\n";
        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

case "superdelete" :  //message that your signup was deleted by another user (superuser)
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nYour signup ";
        $message .= " for " . $resourcelist[$data['resource']]['long'];
        $message .= " at " . date(timeformat,$data['starttime']);
        $message .= " on " . date("D " . dateformat,$data['starttime']);
        $message .= " has been deleted by " . $users[$data['thisuserid']]['firstname'] . " " . $users[$data['thisuserid']]['lastname'];
        $email  = getcontact($data['thisuserid'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";
        $message .= ".\n";
        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

case "asuser" :  //message that you were signed up by another user
        $message =  "\nDear " . $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'] . ",";
        $message .= "\nYou have been signed up by " . $users[$data['thisuserid']]['firstname'] . " " . $users[$data['thisuserid']]['lastname'];
        $email  = getcontact($data['thisuserid'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";
        $message .= " for " . $resourcelist[$data['resource']]['long'];

        $message .= " on " . decodesignup($data,"",false);
        $message .= ".\n";

        $message .= "\n";
        $message .= " Visit " . scheduling_URL . " for additional information.\n";
        break;

}

$addheaders = "From: " . email_from . "\n";
$email = getcontact($data['usn'],"emails");

sendemail ($email, "On-Line Sign-up Message", $message, $addheaders);


}
//----------------------------------------------------
// EMAILMONITORS
//Send an e-mail message to people monitoring a given resource.
//INPUTS: mode ("signup","update","delete", etc), $data (incl: resource, starttime, usn, comment)
function emailmonitors($mode,$data) {
    //check to see if this schedule change falls within the resource monitor time limit
    if ((resmonitor_timelimit == 0) || ($data['starttime'] < (zulutime() + timezone + resmonitor_timelimit)))
    {
    $users        = getusers();
    $resourcelist = getresources();

    //translate mode into descriptive string
    switch ($mode) {
    case "signup" :
        $mode = "Creation of a signup";
        break;
    case "update" :
        $mode = "Change to a signup";
        break;
    case "delete" :
        $mode = "Deletion of a signup";
        break;
    default :
        $mode = ucfirst($mode) . " of a signup";
    }

    //find list of applicable users
    $to = array();
    foreach ($users as $usn => $oneuser) if (checkpermissions($oneuser['resourceblock'],"2",$data['resource'])) $to[] = $usn;

    if (count($to)>0) foreach ($to as $usn)  {

        $addheaders = "From: " . email_from . "\n";

        $message =  "\nDear " . $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] . ",";
        $message .=  "\nWe just received the following change to the schedule for " . $resourcelist[$data['resource']]['long'] . ".";
        $message .=  "\n\n" . $mode . " for ";

        $message .=  $users[$data['usn']]['firstname'] . " " . $users[$data['usn']]['lastname'];
        $email  = getcontact($data['usn'],"emails");
        if ($email != "") $message .=  " (" . $email . ")";

        $message .=  "\n  " . decodesignup($data,"",false);

        $message .= "\n\nVisit " . scheduling_URL . " for additional information.\n";
        $message .= "\nYou received this message because you are indicated as a resource monitor. ";
        $message .= "If you wish to stop receiving these automatic messages, please contact the system administrator.";
        $message .= "\nThank you!\n";

        $email = getcontact($usn,"emails");

        sendemail ($email, "On-Line Sign-up Message", $message, $addheaders);

    }
    }
}
//----------------------------------------------------
//FORGOTTENPASSWORD
// provide the user with an e-mail containing a link to log-in and change
//  their password
function forgottenpassword($data) {

    $users        = getusers();

    //find list of applicable users
    $to = array();
    foreach ($users as $usn => $oneuser) 
        if (strcasecmp($oneuser['username'],$data['username'])==0 | 
           (empty($oneuser['username']) && strcasecmp($oneuser['lastname'],$data['username'])==0) ) 
                $to[] = $usn;

    if (count($to)>0) {
        foreach ($to as $usn)  {

            $addheaders = "From: " . email_from . "\n";

            $message =  "\nDear " . $users[$usn]['firstname'] . " " . $users[$usn]['lastname'] . ",";
            $message .=  "\nThe on-line scheduling system at " . scheduling_URL . " was recenly told you might have forgotten your password and wanted to have a link e-mailed to you which you can use to log in. ";
            $message .=  "\n\nTo log in and change your password, simply click on the following link (if it isn't \"active\" such that clicking takes you to the web page, simply copy this address into your web browser):";

            //log the user in (get a UID for them)
            $userinfo = array('usn' => $usn, 'UID' => assignuid($usn), 'prefs'=>"" );
            //Add login entry uselog.csv
            $uselog = getuselog();
            $uselog[] = array('usn' => $usn, 'uid' => $userinfo['UID'], 'logondate' => zulutime()+timezone, 'ip' => $_SERVER['REMOTE_ADDR']);
            putuselog($uselog);

            $message .= "\n\n  " . scheduling_URL . user_management_prog . "?UID=" . $userinfo['UID'] . "&submit=editmember";

            $message .=  "\n\nIf you did not make this request yourself, don't worry. You can ignore this message (it was probably a mistyped name). This link will automatically expire and has been sent to nobody but you, so your account is still secure.";

            $email = getcontact($usn,"emails");
            sendemail ($email, "On-Line Scheduling Forgotten Password Request", $message, $addheaders);

            logaction("Forgotten Password for","",$userinfo);

        }
        echo "<b>A message with password information has been sent to the e-mail address stored in your account.</b>\n";
    } else {
        echo "<b>Sorry, there appears to be no account with that username.</b>\n";
        $data['usn'] = 'unknown';
        logaction("Unknown User Forgotten Password","",$data);
    }
}

//----------------------------------------------------
// SENDEMAIL
//Send an e-mail message. Alternately will do debug of mail
//INPUTS: 
//   to (addresses)
//   subject
//   message
//   addheaders (additional headers like from, bcc, etc)
//   attach_filepath : string filename to attach (or array of string filenames to attach)
// NOTE: this routine makes use of some code submitted as an anonymous contribution on the PHP.NET website
function sendemail($to, $subject, $message, $addheaders="", $attach_filepath = Null) {

    if (demomode) return(null);
    if (!debug_mail) {
        if (strlen($to)>1 & strlen($message)>1) {
            if ($attach_filepath == Null || (is_array($attach_filepath) && count($attach_filepath)==0)) {

                //No attachment, send normally
                return @mail($to, $subject, $message, $addheaders);

            } else {

                //With attachment, prepare and send message
                if (!is_array($attach_filepath)) $attach_filepath = array($attach_filepath);  // we need an array...
                $mail_attached = "";
                $boundary = md5(uniqid(time(),1))."_xmail";
                if (count($attach_filepath)>0) {
                   foreach ($attach_filepath as $onefile) {
                       if ($fp = fopen($onefile,"rb")) {
                           $file_name = basename($onefile);
                           $content   = fread($fp,filesize($onefile));
                           $mail_attached .= "--".$boundary."\r\n"
                               . "Content-Type: unknkown; name=\"$file_name\"\r\n"
                               . "Content-Transfer-Encoding: base64\r\n"
                               . "Content-Disposition: inline; filename=\"$file_name\"\r\n\r\n"
                               . chunk_split(base64_encode($content))."\r\n";
                           fclose($fp);
                       }
                   }
                   $mail_attached .= "--".$boundary." \r\n";
                   $boundry_info   = "MIME-Version: 1.0\r\nContent-Type: multipart/mixed; boundary=\"$boundary\"";
                   if (substr($addheaders,-1)!="\n") $addheaders .= "\r\n";
                   $addheader_one = $addheaders . $boundry_info;
                   $mail_content   = "--" . $boundary . "\r\n"
                               . "Content-Type: text/plain; charset=iso-8859-1; format=flowed\r\n"
                               . "Content-Transfer-Encoding: 8bit\r\n\r\n"
                               . $message . "\r\n\r\n" . $mail_attached;
                   return @mail($to, $subject, $mail_content, $addheader_one);
               } else {
                   return @mail($to, $subject, $message, $addheaders);
               }

           }

        }
    } else {
        echo "<pre>-----Mail Debug ON-----\n";
        echo "To: " . $to . "\n";
        echo "Subject: " . $subject . "\n";
        echo $addheaders . "\n";
        echo $message . "\n";
        if ($attach_filepath != Null) {
            if (!is_array($attach_filepath)) $attach_filepath = array($attach_filepath);  // we need an array...
            if (count($attach_filepath)>0) foreach ($attach_filepath as $onefile) echo "Attached file: " . $onefile;
        }                            
        echo "-----End of Mail Debug-----</pre>";
        return(true);
    }
}
//----------------------------------------------------
// CLEANUP
function cleanup($_POST) {
        unset($_POST['starttime']);
        unset($_POST['endtime']);
        unset($_POST['checkout']);
        unset($_POST['checkin']);
        unset($_POST['comment']);
        unset($_POST['usn']);
        unset($_POST['signup']);
        unset($_POST['submit']);
        unset($_POST['signupname']);
        return($_POST);
}
//----------------------------------------------------
// SANITIZE
//Remove offending characters (quotes, tags, special chars) from a string
function sanitize($mystr) {

        if (!is_array($mystr)) {
                $mystr = stripcslashes($mystr);                        //convert from coded chars
                $mystr = strip_tags($mystr);                        //strip html tags from comment
                $mystr = str_replace('\\','',$mystr);                  //drop extra slashes
                $mystr = str_replace('"','',$mystr);                   //drop double quotes
                $mystr = addcslashes($mystr,"&?!+;\\\0..\31");     //convert commas in line to coded comma
        } else
                foreach ($mystr as $key=>$value) $mystr[$key] = sanitize($value);        //recursive call if mystr is array

        return($mystr);

}
//----------------------------------------------------
// MAKE_GET_STR
//Convert string to be submitted with a GET request
function make_get_str($mystr) {
        $mystr = sanitize($mystr);                        //convert from coded chars
        $mystr = str_replace(' ','+',$mystr);           //spaces = +
        $mystr = str_replace('#','',$mystr);                   //remove #
        return($mystr);
}
//----------------------------------------------------
// ZULUTIME
//return time stamp in mktime format for zulu time
// (based on user-entered time-zone)
// NOTE: we used to use gmmktime and correct for time zone differences
//  but there were far too many inconsintencies between PHP installations
//  for that to be reliable, now everything is set by the user and we just
//  use mktime here and correct for timezone where needed.
function zulutime() {
        return (mktime());
}
//----------------------------------------------------
// BESTTIME
// Convert time (in seconds) to most convenient textual description
function besttime($ts) {

    if ($ts<90) { //seconds are best
        $units = "seconds";
        $factor = 1;

    } elseif ($ts<90*60) { //minutes are best
        $units = "minutes";
        $factor = 60;

    } elseif ($ts<24*60*60) { //hours are best
        $units = "hours";
        $factor = 60*60;

    } elseif ($ts<90*24*60*60) { //do it in days
        $units = "days";
        $factor = 60*60*24;

    } else { //do it in months
        $units = "months";
        $factor = 60*60*24*30;

    }
    $ts = $ts/$factor;
    if ($ts==floor($ts))
        $format = '%u';  
    else
        $format = '%0.1f';  //show decimal ONLY if necessary

    $enc = sprintf($format . ' %s',$ts,$units);

    return($enc);
}
//----------------------------------------------------
// VALIDATEDATE
// Validate Date entry.  Entry can be provided as a string
//    using english text if desired.  This function fixes the
//    problem with using dashes in the date as well.
//    The function returns "-1" if the date is invalid.
//    If the date is valid, the function returns the date
//    formatted as a timestamp.
//  NOTE: this function also converts euro-date format into
//    time-stamps (since the strtotime function won't do this)
function validatedate($dateinput){

        //If dateinput is empty, then date is bad
        if (strcmp($dateinput, "") == 0) return (-1);

        //Replace any dashes with slashes in the date entry since dashes cause problems
        $dateslash = str_replace("-","/",$dateinput);

        //set aside any time information at the end
        $dateslash = trim($dateslash);
        $datetime  = explode(" ",$dateslash);
        if (count($datetime)>2) { //probably "jan 5, 2005" format
            return(strtotime($dateinput));
        }
        $dateslash = $datetime[0];

        //Check if month and day values are valid (if entered usind dashes or slashes)
        $dateexplode = explode("/",$dateslash);
        if (count($dateexplode) == 3) {
            if ((!eurodate && !checkdate($dateexplode[0], $dateexplode[1], $dateexplode[2]))
            || (eurodate && !checkdate($dateexplode[1], $dateexplode[0], $dateexplode[2]))) return (-1);
        }elseif (count($dateexplode) == 2) {
            if ( (!eurodate && ($dateexplode[0] > 12) || ($dateexplode[0] < 1) )
                || (eurodate && ($dateexplode[1] > 12) || ($dateexplode[1] < 1) )) return (-1);
        }

        //Check if the date string is of the proper syntax
        if (!eurodate) {
            $datestring = strtotime($dateslash . " " . $datetime[1]);
        } else {
            //invert eurodates
            if (count($dateexplode) == 3)
                $datestring = strtotime("" . $dateexplode[1] . "/" . $dateexplode[0] . "/" . $dateexplode[2] . " " . $datetime[1]);
            elseif (count($dateexplode) == 2)
                $datestring = strtotime("" . $dateexplode[1] . "/" . $dateexplode[0] . " " . $datetime[1]);
            else
                $datestring = strtotime($dateslash . " " . $datetime[1]);
        }

        //If date is valid, return the date as a timestamp
        return ($datestring);
}
//----------------------------------------------------
// RESOURCELINK
//  create resource icons and HTML links as indicated by resource info
//  inputs are a resource record to determine what links should be given
//  and "givename" - a boolean flag when true = give resource name and <br> after each set of links
function resourcelink($UID,$resource,$givename=false) {

    if ($resource['comments']!="") {  //got a comment? give help icon and links
    
        $commentlinkinfo = "resourcecomments" . required_php_extension . '?resource=' . $resource['resource'];
        echo "<a target=\"_blank\" href=\"$commentlinkinfo\" ";
        echo "onclick=\"javascript:infowin=window.open('" . $commentlinkinfo . "','comments','title=no,toolbar=no,scrollbars=yes,status=no,menubar=no,resizable=yes,width=600,height=230');";
        echo "infowin.focus(); return false;\">";
        echo '<img src="helpicon.gif" alt="(?)" border=0>';
        if ($givename) echo "&nbsp;" . $resource['long'] . "<br>";
        echo '</a>';
        echo "\n";

    }

    if ($resource['warnings']!="") {  //got a comment? give help icon and links
    
        $commentlinkinfo = "resourcecomments" . required_php_extension . '?resource=' . $resource['resource'];
        echo "<a target=\"_blank\" href=\"$commentlinkinfo\" ";
        echo "onclick=\"javascript:infowin=window.open('" . $commentlinkinfo . "','comments','title=no,toolbar=no,scrollbars=yes,status=no,menubar=no,resizable=yes,width=600,height=230');";
        echo "infowin.focus(); return false;\">";
        echo '<img src="warnicon.gif" alt="(?)" border=0>';
        if ($givename) echo "&nbsp;" . $resource['long'] . "<br>";
        echo '</a>';
        echo "\n";

    }

}
//----------------------------------------------------
// PRINTFOOTER
//     Print the footer for the end of each page.
// Inputs "tableend" and "hr" control extra HTML tags to be output
function printfooter($tableend=true,$hr=true){
    if ($tableend) echo  '</td></tr></table>';
    if ($hr)       echo '<hr>';
    echo '<div align="right"><em><font face="Arial, Helvetica, sans-serif" size=-1><a href="license.html" target="_blank">Powered-by ORS</a></font></em></div>';
    echo '</body>';
    echo '</html>';
}
//----------------------------------------------------
// HYPHENATED
//make a rough guess at where a word should be hypenated if it is > $maxlength
function hyphenated($name) {

    $maxlength = hyphen_length;
    $vowels = array('a','e','i','o','u');
    if (strlen($name) > $maxlength) {

        //look for a space character first
        $lookat = strlen($name)/2;
        if ($lookat < $maxlength) $lookat = $maxlength;
        $pos = 0;
        for ($j=0; ( ($j<=$lookat) & (j+$lookat)<(strlen($name)-1) ); $j++) {
            if ($name{$lookat-$j}==' ') {
                $pos = $lookat-$j+1;
                break;
            }
        }
        if ($pos>1 & $pos<strlen($name)-1) {
            $name = substr($name,0,$pos) . "<br>" . substr($name,$pos);
        } else {
            //no space, check for two non-vowels to split between 
            $lookat = strlen($name)/2;
            for ($j=0; ( ($j<=$lookat) & (j+$lookat)<(strlen($name)-1) ); $j++) {
                if (!in_array(strtolower($name{$lookat-$j}),$vowels) & !in_array(strtolower($name{$lookat-$j+1}),$vowels)) {
                    $pos = $lookat-$j+1;
                    break;
                }
                if (!in_array(strtolower($name{$lookat+$j}),$vowels) & !in_array(strtolower($name{$lookat+$j+1}),$vowels)) {
                    $pos = $lookat+$j+1;
                    break;
                }
            }
            if ($pos>0 & $pos<strlen($name)-1) $name = substr($name,0,$pos) . "-<br>" . substr($name,$pos);
        }
    }

    return($name);
}
//---------------------------------------------------


//----------------------------------------------------
//JMS 2/14/02 Modified Update to output array of info
// -added e-mails to "done by" e-mail messages (e.g. "you were updated by whomever (email)" )
// -added sortbystarttime
//JMS 2/19/02
// -added listing of users promoted or demoted
// -added option to keep deleted-user's signups (makes them "Abandoned")
// -added validateusers to usermenu function
// -added automatic support for second e-mail ('emails' option in getcontact)
//JMS 2/21/02
// -added sanitize and make_get_str to firmly close security holes
//JMS 2/22/02
// -fixed comma problem associated with date field
//JMS 2/28/02
// -added e-mail of all signups as backup
//JMS 3/4/02
// -fixed alternate signup validate bug (<=4 primaries, <=6 TOTAL)
// -removed supression of mail warnings for daily actions
//jms 3/9/02
// -added e-mail of abandoned signups
// -modified e-mail message format
// -added (but did not fully implement) prefs in UIDs
//jms 3/16/02
// -fixed overnight signup bug when in 1/2 hour view mode
//jms 3/31/02
// -added recursive sanitize call for arrays passed to sanitize
// -converted to csvwrite and csvread routines
// -added full support of viewing only selected resources
//jms 4/16/02
// -time adjustment to getusersignups
// -test for empty matrix in demote (bug when reassigning AND promoting?)
//jms 4/19/02
// -modified test for "Daily Actions Needed"
//jms/cw 7/22/02
// BUGS FIXED
// -fixed multi-day overlap promote bug (only promoted first overlaped item)
// -removed world-modifiable status for resource folders
// FEATURES ADDED
// -added test for empty authorized user list
// -included additional headers in all debug_mail output,
// -squelch all mail error messages
// -add password encryption for on-line memberlist control (only)
// -add default state for resource blocking (what new users can sign up for)
//jms 7/28/02
// -created single "sendemail" function to be used for all e-mail
// -made lockfile logic more robust, added variable wait_for_lock default
// -fixed zero-length file read error in readkeyedfile
// -fixed no-users bug in assignusn
//CBW 8/4/02
// -modified "getcontact" so that the returned email addresses would
//           contain a link to actually send an email
// -modified "listusercontact" to allow option for shortened listing of info
// -added two extra fields to resources.csv file: maxsignup & doublebook
// -modified "validatesignups" rules that check for signup overlaps and signups
//     that are too long to use the new doublebook and maxsignup settings
// -modified "max signups rule" to allow option to NOT count reservations
//     for resources that can be doublebooked (ie. don't tally instructor
//     reservations, other wise a single instructor+aircraft reservation
//     would count towards two signups rather than one)
// -moved rule definitions (and some error messages) into defaults file
// -added GUI interface to change program settings (used to be stored in
//      in the defaults file).  Settings are now stored in defaults & in
//      the config.php file (located in the data directory).
// -added "ReadConfigFile", "WriteConfigFile", and "ActivateConfigFile"
//      functions to support the SuperUser program settings GUI
//JMS 8/27/02
// -added CBW's fix to validateusers (trim)
//JMS 10/22/02
// -added reconcilecalendar
// -renamed read/writeconfigfile to read/writetextfile
// -added test for config_encoded
// -oh god... lots of other changes... major revisions
// -revised "getcontact" so that "emails" gives raw e-mails (for sending messages) but "emails_html" does HTML tagged e-mails
//JMS 10/30/02
// -split out some functions to speed up compiliation (dailyaction related stuff)
//CBW 11/10/02
// -added ValidateDate function
//JMS 11/14/02
// -moved annoucements read to after assignment of UID (it can now be used in announcements file in PHP code)
// -added complete am/pm clock support
// -revised validatesignups to allow new-signup testing (for hard-enforcement)
// -fixed bug in email_primaries call in message
//CBW 11/15/02 -validatedate fix
//CBW 11/17/02
//  -added "Deny public view" field to resource database
//CBW 11/23/02
//  -added "reconcilefuturesignups" function to provide ability to restrict inactive or expired accounts
//  -modified addsignup and deletesignup to include functionality for expired and restricted accounts
//JMS 12/02/02
// -moved reconcileexpiredaccount to maintfunctions
// -removed validateusers from lookupmemindex -too damn slow and taken care of by other calls.
//JMS 12/03/02
// -fixed bug in validdatedate with two-element (no year) dates
// -fixed late-in-day bugs related to ndays calcs in deletesignup and addsignup (made more rigorous calc)
//CBW 1/12/03
// -fixed bug with "resourceblock" field in userlist.csv file which used to
//      be limited to 15 resources.  The integer field was changed to a
//      "b-format" string which should be limitless (within reason)
//JMS 2/23/02
// -modified schedulelock logic - now delays as expected and returns false if no lock available
// -added set_time_limit at start
// -added faster lookupusn option (if you pass in users list)
//JMS 3/02/03
// -added fixed SEC for administrator account (1110)
// -added test for file existance in putauthorizedusers
//JMS 3/21/03
// -fixed bug which caused recursive dailyaction calls when timezone was large
//JMS 4/5/03
// -use globally-cached versions of users,resources, and members instead of
//    re-reading the files multiple times.
// -removed extraneous commented-out lines
//CBW 4/7/03
// -added userlog.csv, modified UID handling
//JMS 4/8/03
// -modified UID timeout logic (getuids and useridlookup)
//JMS 4/13/03
// -checkpermissions - revised logic to handle bit-wise comparisons (expanded info in bit)
//JMS 4/15/03
// -removed validate_admin_signups test
//JMS 4/25/03
// -added IP address logging to userlog
// -added forgotten password e-mail functionality
//CBW 5/7/03
// -added time limit to resource monitor functionality (user defined in config.php)
//JMS 5/19/03
// -changed "append" action of getmemberlist to "flag" which also reads in authorizedlist when true
//CBW 6/18/03
// -updated "update" and "deletesignup" functions to fix bug with restricted user signups that
//  were wrongfully deleted when the associated double-booked signup was updated in any way
//JMS 7/27/03
// -added {} around conditional includes (just to be sure)
// -added test for [longest_signup] in validate signups
//JMS 7/30/03
// -if testing a single signup in validatesignups, renumber if that signup gets combined (because it was back-to-back) with another
// -added configuration option to allow rule violation for back-to-back signups
// -modified update to test for hard-enforce rule violations and reject signup if it breaks those rules
//  (This change will also enable us to do tests for restricted user problems - we simply add a similar test
//   in the same place for any change which might cause a restricted user to break the double-book restriction)
//JAT 8/3/03
// -possible fix for "Insert between 2 other priorities" bug (693431):
//  commented out priority reduction code in demote().
//JMS 8/5/03
// -validated JAT fix for insert bug
// -fixed bug which kept overlapping signups from being promoted when updates with forcepriority were done
//JMS 8/18/03
// -revised getday to handle any combination of hour_interval and signup length
// -fixed listusercontact to show correct phone number when only one is present
// -removed case sensitivity for name in getcontact
//JMS 10/1/03
// - used include_once for all instances of include of maintfunctions
//JMS 10/3/03
// -moved expire, restricted, and lastactivity into users
// -created new function "RECORDACTIVITY" to localize updating of user activity date
//JAT 10/12/03
// -Improved formatting of e-mail debug feature
//JMS 11/2/03
// -slight mod to decodesignup (allow inclusion of priority in passed data)
//JMS 12/11/03
// -added allow_doublebook flag (turns off double-book warnings)
//JMS 12/14/03
// -listusercontact now returns user contact info and OPTIONALLY displays it (default)
// -added echo_prodding option to message
// -revised many calls to getusersignups to get past signups too (maintains them)
//JMS 12/29/03
// -added signup status support to update and addsignup
// -added automanage_alternates feature (turn off alternates reporting & e-mails)
// -fix format of length of signup
// -changed findpriority to handle automatic calendar entries
//JMS 1/5/04
// -added eurodate support
// -modified recurring signup logic
// -added lowest-open-slot logic if automanage_priority is off
//JMS 1/8/04
// -fixed status-update bug
//JMS 1/18/04
// -added checkin/checkout times to setsignupstatus and decodesignups
// -changed normal values for status
//JMS 1/26/04
// -added attachment functionality to sendemail
// -test in readkeyedfile for empty lines (ignored)
// -made recurring signup functionality dis-ableable
// -minor change to printfooter (now used everywhere)
// -added getfilelist function (dir of folder)
//JMS 2/6/04
// -revised UID length to be hard-coded (removed config option)
// -added hypenated back as general-use function
//JMS 2/7/04
// -added quiet e-mail sending (squelch error messages)
// -added ability to include directories in file list from getfilelist
//jms 2/13/04
// -added warning functionality
//jms 2/19/04
// -added better "not installed" logic and tests for empty config info
//jms 3/11/04
// -added "enddate" and "startdate" tests for recurring signups
// -fixed potential future bug in login when we go to usernames instead of last names
//JAT 3/25/04
// -now append action log entries instead of reading/writing entire file
//JMS 3/29/04
// -changed logaction to only perform dailyactions test on no-action-to-log calls
//JMS 4/23/04
// -revised recurring signups to NOT change if other signups already exist
//JMS 4/28/04
// -trim extra spaces off leading and trailing data fields as they are read in by readkeyedfile
//JMS 4/29/04
// -fixed PHP 4.3.6 CSV bug
// -allow users without usernames to do forgotten password using last name instead
//JMS 7/29/04
// -fixed abandoned bug (assignusn called without username input)
//CBW 8/7/04
// -fixed logaction (removed "+timezone" in check whether action is first logged action of day)
//  (Daily actions were sent multiple times if script hosted in different timezone than users)
//JMS 8/25/04
// -added logic to convert recurring signups to real if another signup is created on the same day
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
