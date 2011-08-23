<? 
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

require_once('functions.php');

//----------------------------
// GETFILELIST
//  returns a list of files in a specified directory
//   and (optionally) all of it's sub-directories
// Input: 
//   dir : the directory to start at
//   recurse : true = recursive listing of files in sub-dirs too
//   basedir : prepended directory name
// Output:
//   array of file names with prepended folder names
function Ngetfilelist($dir,$recurse=true,$basedir="") {

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
        
        //parse subdirs for content files
        if ($recurse && count($dirs)>0) foreach ($dirs as $subdir) $files = array_merge($files,Ngetfilelist($subdir,$recurse,$basedir));

    } else {
        echo "Could not open " . $basedir . $dir . "<br>";
    }

    return($files);

}
//-----------------------------------------------------
function Ncreatebackup () {

    $backup_folder = root_dir . "../backup/";

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

    //Read in all files and and compress them
    $data = "";
    foreach (Ngetfilelist(root_dir) as $filename) {
        $data .= "[==>FILENAME:" . $filename . "]\n";
        $data .= implode("",file(root_dir . $filename));
        $data .= "[==>EOF]\n";
    }

    //Write out compressed data to .ors file
    $fid = gzopen($backup_file,'wb');
    if ($fid) {
        gzwrite($fid,$data);
        gzclose($fid);
        return($backup_file);
    } else {
        return(false);
    }

}

//-------------------------------------------------------

if (!Ncreatebackup()) {
    echo "<h3>Unable to create backup</h3><p>\n\n";
} else {
    echo "<h3>Backup file has been created in the scheduler's "Backup" folder</h3><p>\n\n";
}

//--------------------------
//JMS - this function allows old versions of ORS to be backed up in new format
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
