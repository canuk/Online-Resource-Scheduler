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

if ($administrator) {
    switch($_POST['submit']){
            case "restore" :
            case "Restore" :
            case "Confirm Restore" :
            case "Review Contents" :

                    echo "<b>Restore from backup:</b><br>";

                    if (demomode) {echo "Demo mode - function disabled"; break;}

                    //verify that backup folder exists
                    $backup_folder = root_dir . "../backup/";
                    if (!file_exists($backup_folder)) mkdir ($backup_folder, 0700);        // install backup folder if necessary
                    chmod ($backup_folder,0700);

                    //get list of old backup files
                    $backup_files  = array();
                    $filelist = getfilelist($backup_folder,false);
                    foreach ($filelist as $file) if (substr($file,-4)=='.ors') $backup_files[] = $file;
                    foreach ($filelist as $file) if (substr($file,-4)=='.dat') $backup_files[] = $file;

                    $configpw = our_crypt(strtolower($_POST['ConfigPassword']));
                    if (strcasecmp($configpw,configpassword)!=0 & strcasecmp($configpw,our_crypt(configpassword))!=0) $_POST['ConfigPassword'] = "";

                    if (strcasecmp(configpassword,"") != 0 & $_POST['ConfigPassword']=="") {

                        ?>
                        You must enter the Configuration Password to restore backup data.
                        <?

                        echo "<form name=\"configpass\" method=\"" . submit_method . "\" action=\"" . backup_util_prog . "?submit=restore\">";
                        echo "<br>";
                        echo "<b>Please Enter the Configuration settings password:</b>";
                        echo "<input type=\"password\" name=\"ConfigPassword\" value=\"\">";
                        echo "<input type=\"hidden\" name=\"UID\" value=\"" . $UID . "\">";
                        echo "<input type=\"hidden\" name=\"mode\" value=\"restore\">";
                        echo "<input type=\"submit\" name=\"submit\" value=\"Restore\">";
                        break;

                    }

                    //Show list of available files (or request upload of file if none), confirm restore
                    if ($_POST['submit']!="Confirm Restore" && $_POST['submit']!="Review Contents") {
                        chmod ($backup_folder,0777);

                        ?>
                        To restore the system from a previous backup, choose the desired backup file from the list below. <br>
                        To restore from a backup previously e-mailed but not listed below, you must first upload the backup <br>
                        file using an FTP client. The file should be uploaded into the <b>backup</b> folder in the root <br>
                        scheduler folder ( <? echo scheduling_URL ?>backup/ ) as a file with the extension <b>.dat</b>.<br>
                        After this is done, <a href = <? echo '"' . backup_util_prog . '?UID=' . $UID . '&submit=restore&ConfigPassword=' . $_POST['ConfigPassword'] . '"'; ?> >
                                <font color="#990000">Click here to refresh the file list</font></a><p>
                        <?

                        //sort names by date
                        $filedates = array();
                        foreach ($backup_files as $onefile) $filedates[] = filectime($backup_folder . $onefile);
                        array_multisort($filedates,$backup_files);
                        $backup_files = array_reverse($backup_files);

                        ?>
                        <table border=1 cellspacing=0 cellpadding=4><tr><td>
                        <form name="confirmrestore" action="<? echo backup_util_prog; ?>">
                            <input type="hidden" name="UID" value="<? echo $UID; ?>">
                            <input type="hidden" name="ConfigPassword" value="<? echo $_POST['ConfigPassword']; ?>">
                            <b>Available Backup Files:</b><br>
                            <select name="backupfile" rows=4>
                                <? 
                                if (count($backup_files)>0) foreach ($backup_files as $onefile) {
                                    echo "<option value=\"" . $onefile . "\"";
                                    if (strcasecmp($_POST['backupfile'],$onefile)==0) echo " selected";
                                    echo ">" . $onefile . " (";
                                    echo date(dateformat . "/Y " . timeformat,filectime($backup_folder . $onefile)) . ")";
                                    echo "</option>\n";
                                }
                                ?>
                                <option value="uninstall">** Uninstall ORS Database **</option>
                            </select><p>
                            <input type="submit" name="submit" value="Review Contents">
                            <input type="submit" name="submit" value="Confirm Restore">
                            </td></tr></table>
                            <h3>WARNING! After you click "Confirm Restore" all existing<br> data will be overwritten with the backup data!</h3>
                            This will return the system to the status as of the last backup and is unrecoverable.<br>
                            You may wish to backup the <b>current</b> status prior to restoration.<p>
                            You can use the "review contents" button to view the raw file contents before restoring. <br>
                            If the contents appear garbled (other than plain text), do NOT use this file for restore.
                            <p>
                        
                        </form>
                        <?
                        break;
                    }
                    chmod ($backup_folder,0700);

                    $reviewonly = ($_POST['submit']=="Review Contents");
                    $uninstall  = ($_POST['backupfile']=='uninstall');

                    if ($uninstall) {
                        $uncomp = array('');
                        if ($reviewonly) echo "<h3>All ORS databases will be erased if you choose Uninstall</h3>";
                    } else {
                        if ($reviewonly) echo "<h3>Contents of " . $_POST['backupfile'] . "</h3>";
                        if ($reviewonly) {
                            echo '<a href = "' . backup_util_prog . '?UID=' . $UID . '&submit=restore&backupfile=' . $_POST['backupfile'] . '&ConfigPassword=' . $_POST['ConfigPassword'] . '">';
                            echo 'Click here to return to file list</a>';
                        }

                        //real file - read it if we can
                        if (in_array('zlib',get_loaded_extensions())) {
                            //load and uncompress backup file
                            $uncomp = gzfile($backup_folder . $_POST['backupfile']);
                        } else {  //Don't have access to zlib
                            //load backup file
                            $uncomp = file($backup_folder . $_POST['backupfile']);
                        }                    
                        if (!is_array($uncomp)) $uncomp = array($uncomp);
                        if (!$uncomp || count($uncomp)==0 || (count($uncomp)==1 && $uncomp[0]=="")) {
                            ?>
                            <h3><font color="#990000">Unable to read from backup file "<? echo $_POST['backupfile']; ?>" or file is empty.</font></h3>
                            Please check the access permissions on the file. They need to be world-readable and writable (666 = rw-rw-rw- ).<p>
                            <?
                            $_POST['submit']="";
                            break;
                        }
                    }


                    //if this is old "dat" format, decode it
                    if (substr($_POST['backupfile'],-4)=='.dat') {
                        @set_time_limit(0);

                        for ($c=1; $c<27; $c++) $vals[] = "" . chr($c+96);
                        for ($c=32; $c<64; $c++) $vals[] = "" . chr($c);
                        for ($c=1; $c<27; $c++) $vals[] = "" . chr($c+64);
                        $keys = $vals;
                        for ($j=1; $j<12; $j++) array_unshift($vals,array_pop($vals));
                        $enc = array();
                        $k = count($keys);
                        for ($j=0; $j<$k; $j++) $enc[$vals[$j]] = $keys[$k-$j-1];

                        $filename = "";
                        $comp     = $uncomp;
                        $uncomp   = array();
                        foreach ($comp as $line) {
                            if ($filename=="") {
                                if (substr($line,0,3) == "==>") {
                                    $filename = root_dir . trim(substr($line,3,-1));
                                    $uncomp[] = "[==>FILENAME:" . trim(substr($line,3,-1)) . "]\n";
                                }
                            } elseif (substr($line,0,5) == "[EOF]") {                        //end of file marker?
                                    $uncomp[] = "[==>EOF]\n";
                                    $filename = "";                            //and clear our flag
                                    $fid = false;
                            } else {
                                if (substr($filename,-11)!='restore.php') {
                                    if (strlen($line)>2) {
                                        $uncomp[] = strtr($line,$enc) . "";
                                        for ($j=1; $j<8+shiftseed; $j++) array_unshift($vals,array_pop($vals));
                                        $enc = array();
                                        $k = count($keys);
                                        for ($j=0; $j<$k; $j++) $enc[$vals[$j]] = $keys[$k-$j-1];
                                    }
                                } else {
                                    $uncomp[] = $line . "";
                                }
                            }
                        }
                    }

                    //erase old files if not just reviewing
                    if (!$reviewonly) {
                        //remove files (except ones starting with "."
                        foreach (getfilelist(root_dir,true,false) as $filename) 
                            if ($uninstall | substr(basename($filename),0,1)!=".") @unlink(root_dir . $filename);
                        //remove dirs except root dir and non-empty dirs (e.g. ones with . files in them)
                        foreach (getfilelist(root_dir,true,true) as $filename) 
                            if (is_dir(root_dir . $filename) && $filename!="" && count(getfilelist(root_dir . $filename,true,true))<2) @rmdir(root_dir . $filename);

                        //remove root dir if uininstall
                        if ($uninstall) {
                            echo "Unlocking scheduler files<br>";
                            foreach (getfilelist(root_dir . '../',true,false) as $filename) 
                                if (!is_dir(root_dir . '../' . $filename) && $filename!="") @chmod(root_dir . '../' . $filename,0777);
                            echo "Unlocking backup folder<br>";
                            chmod ($backup_folder,0777);
                            echo "Removing root database directory<br>";
                            if (is_dir(root_dir)) @rmdir(root_dir);
                            echo "<h3>All ORS databases have been cleared and the Backup folder has been made world-writeable for you to manually clear the backups</h3>";
                            break;
                        }
                    }

                    $uncomp = implode('',$uncomp);
    
                    //extract files from file
                    $anyfailure = false;
                    while (strlen($uncomp)>0) {
                        $stok    = strpos($uncomp,"[==>FILENAME:");
                        $etok    = strpos($uncomp,"[==>EOF]");

                        //parse section into filename and contents
                        $onefile = substr($uncomp,13,$etok-$stok-13);  //extract single file from top
                        $ftok    = strpos($onefile,"]");               //locate end of filename markup
                        $filename     = substr($onefile,0,$ftok);
                        $filecontents = substr($onefile,$ftok+2);

                        if (!$reviewonly) {
                            //check for pre-pended folder name and create if necessary
                            $toparse = $filename;
                            $dirname = "";
                            while (False!=($slashtok = strpos($toparse,"/"))) {
                                $dirname = $dirname . substr($toparse,0,$slashtok+1);
                                $toparse = substr($toparse,$slashtok+1);
                                if (!file_exists(root_dir . $dirname)) {
                                    mkdir(root_dir . substr($dirname,0,-1), 0755);
                                }
                            }

                            //create file as specified
                            $fid = @fopen(root_dir . $filename,'w');
                            if ($fid) {
                                fwrite($fid,$filecontents);
                                fclose($fid);
                                echo ".";
                            } else {
                                $anyfailure = true;
                                echo "<br>Cannnot Create File " . $filename . "&nbsp;&nbsp;&nbsp; - skipped\n";
                            }
                        } else { 
                            //just looking
                            echo "<br><font size=+1><b>File: </b>'" . $filename . "'</font><br>";
                            echo "<pre>";
                            echo htmlspecialchars($filecontents);
                            echo "</pre>\n";
                        }
                        
                        //drop file off of uncompressed contents
                        $uncomp  = substr($uncomp,$etok+9);

                    }

                    if ($reviewonly) {
                        echo '<a href = "' . backup_util_prog . '?UID=' . $UID . '&submit=restore&backupfile=' . $_POST['backupfile'] . '&ConfigPassword=' . $_POST['ConfigPassword'] . '">';
                        echo 'Click here to return to file list</a>';
                    }
                    
                    if (!$reviewonly) {
                    

                        //force reload of files from disk
                        $GLOBALS['membersfile']   = Null;
                        $GLOBALS['usersfile']     = Null;
                        $GLOBALS['resourcesfile'] = Null;

                        //old format restore? run restore.php file
                        if (substr($_POST['backupfile'],-4)=='.dat')
                            if (file_exists(root_dir . "restore.php")) include(root_dir . "restore.php");

                        logaction("Restore from Backup",$thisuserid);

                        if ($anyfailure) echo "<h2><font color=#990000><b>WARNING:</b> Some data was not restored and aspects of the scheduler may not be correct.<br> You may also have to use <em>default passwords</em> for the administrator account.</font></h2>";
                        else echo "<h3><font color=\"#009900\">Restore complete</font></h3>";
                        echo "You must now <a href=\"upgrade" . required_php_extension . "\">log out and log back in</a> as an administrator to complete the restoration.<p>";

                    }

                    break;

            //- - - - - - -
            case "backup" :
                    ?>
                    <b>Backup to:</b><br>
                    <? if (demomode) {echo "Demo mode - function disabled"; break;} ?>
                    <ul>
                    <a href=<? echo '"' . backup_util_prog . '?UID=' . $UID . '&submit=backupemail"'; ?>>
                    <li>Write to site backup folder and E-mail to designated users</a>
                    <a href=<? echo '"textbackup.php?UID=' . $UID . '"'; ?>>
                    <li>Write to site backup folder and Download file</a>
                    </ul>
                    <?
                    break;

            case "backupemail" :

                    if (demomode) {echo "Demo mode - function disabled"; break;}
                    require_once('maintfunctions.php');
                    $userlist = mailallsignups();
                    echo "<b>Backup has been created ";
                    if (strlen($userlist)>2) {
                        echo " and e-mailed to the following e-mail addresses:</b><br>";
                        echo "&nbsp;&nbsp;&nbsp;" . $userlist . "<br>";
                    } else {
                        echo ".</b><p>";
                    }
                    $_POST['submit']="";
                    break;
    }
}

scheduleunlock();
printfooter();

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
//JMS 1/26/04
// -revised restore for new backup format
// -added choice from listing of all backup files on site
//JMS 2/7/04
// -added logic to handle if no zlib is available to uncompress
// -added restore from old format
// -also remove folders when restoring from backup
//JMS 2/19/04
// -revised procedure for restore
// -added uninstall option
//JMS 4/23/04
// -changed menus slightly
//JMS 4/30/04
// -go through upgrade after restore
//JMS 7/8/04
// -mkdir needs mode in addition to name
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!

