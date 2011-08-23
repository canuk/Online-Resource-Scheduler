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

//--- next step will be load of configuration settings, force it through header
require_once('functions.php');
 
$startlink = "utilities" . required_php_extension . "?username=Administrator&password=letmein&pwsupplied=1&forceconfig=AskedByInstall&submit=Submit";
if ($_GET['htaccess']==1)	$startlink .= "&htaccess=1";  //have utilities add htaccess file

?>
<head>
	<title>
	Scheduler Setup Step 1
	</title>
</head>
<body>
<table border="1" cellspacing="0" cellpadding="0" backgroundcolor="#eeffff">
  <tr>
    <td align="center"> 
      <h2><img src="scheduling.gif" width="376" height="61"
 alt="ORS Online Resource Scheduler"></h2>
 </td></tr>
<tr> <td>

<h2>Installation Step 2: Create lock files and Update old installation</h2>

<?

//========================================
//Create .htaccess files (assumes apache server!)

echo "<li>Creating .htaccess files to lock-out data folders<br>";

$fp = @fopen(root_dir . ".htaccess","w");
if ($fp) {
    fwrite($fp,"<Limit GET>\n");
	fwrite($fp,"Order Deny,Allow\n");
	fwrite($fp,"Deny from All\n");
	fwrite($fp,"</Limit>\n");
    fclose($fp);
} else {
    echo "<h3>Attempt to create root_dir .htaccess file <font color=\"#990000\">FAILED</font></h3> Please see FAQ about creating file manually. Be aware that user files are visible to the internet!";
}

$fp = @fopen(root_dir . "../backup/.htaccess","w");
if ($fp) {
    fwrite($fp,"<Limit GET>\n");
	fwrite($fp,"Order Deny,Allow\n");
	fwrite($fp,"Deny from All\n");
	fwrite($fp,"</Limit>\n");
    fclose($fp);
} else {
    echo "<h3>Attempt to create backup .htaccess file <font color=\"#990000\">FAILED</font></h3> Please see FAQ about creating file manually. Be aware that backup files are visible to the internet!";
}

//========================================
//Update to version 3.5

$users        = getusers();
$members      = getmemberlist(false); //decide based on memberlist ONLY being empty

if (count($members)>0) {

    $members      = getmemberlist(true); //now get ALL members (incl. authorized users)

    echo "<li>Updating old database format to version 3.5<br>";
    foreach ($users as $usn => $oneuser) {

        //look up usn from username
        $usercheck = $users[$usn]['firstname'] . $users[$usn]['lastname'];
        $usercheck = str_replace(" ","",$usercheck);
        $memindex  = -1;
        foreach ($members as $index => $membersearch) {
            $membercheck = $membersearch['firstname'] . $membersearch['lastname'];
            $membercheck = str_replace(" ","",$membercheck);
            if (!strcasecmp ($membercheck, $usercheck)) {
                $memindex = $index;
                break;
            }
        }
        
        //move the fields
        if ($memindex>-1) {
            foreach ($members[$memindex] as $key => $value) $users[$usn][$key] = $value;
            //create username if not there
            if ($users[$usn]['username']=="") $users[$usn]['username']=strtolower($users[$usn]['lastname']);
        }
            
    }
    $members = array();
    
    putusers($users);

    unlink(memberlist_name);
    unlink(authorizedlist_name);
}    

?>

<h3>Install Step 2 Complete - <a href="<? echo $startlink; ?>">Click to continue installation...</a></h3>

</td></tr></table>
</body>
<?//--------------------------------------------------------
//JMS 1/26/04
//   -removed auto-update
//JMS 2/7/04
//   -fixed errors in 3.1/3.5 upgrade tests
//   -fixed install link
//JMS 2/19/04
//   -added .htaccess creation
//JMS 12/16/04
//   -fixed another error in 3.5 upgrade test
//   -eliminate memberlist and authorizedlist files after upgrade
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!
