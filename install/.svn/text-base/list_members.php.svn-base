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

require_once('utilities_menu.php');
if (!$thisuserid) return;

if (!$noheader) {
	echo "<b>Member List:</b>";
}
echo "<br><font size=-1>Click last name to view user signups and pencil to edit user information.</font><p>";
if ($_GET['name'] == "") $fontsize="-1"; else $fontsize="";
?>

<table border=0 cellpadding="0" cellspacing="0">
<tr align="left" valign="bottom">
	<td nowrap>&nbsp;</td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">Last Name</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">First Name</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
	<td       ><b><font size="<?echo $fontsize; ?>"><? echo info_fieldname; ?></font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">Home Phone</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;&nbsp;</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">Work Phone</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">Email 1</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">&nbsp;&nbsp;</font></b></td>
	<td nowrap><b><font size="<?echo $fontsize; ?>">Email 2</font></b></td>
</tr>

<?php
$users = sortbyname(getusers());

$rulelinecount=0; //make every n'th line gray
foreach ($users as $person) {
        
		if ((
		    $_GET['name'] == ""
		    && ($person['ratings']!='hide' || $person['usn']==$thisuserid || $calendarauth || nouserhide)
		    )
            || strtolower($_GET['name']) == strtolower($person['lastname'])
            || strtolower($_GET['name']) == strtolower($person['username'])) {

            if (!allow_memberlist_view && !$administrator && $person['usn']!=$thisuserid) continue; //skip to next person

            //Show this user...
            
            $username = $person['username'];
            if (empty($username)) $username = $person['lastname'];

            $rulelinecount++;

            if ($rulelinecount==3) {
                $rulelinecount = 0;
                $linecolor = '#eeeeee';
            } else {
                $linecolor = '#ffffff';
            }

            //hidden users are shown in special color to administrators (only!)
            if ($person['ratings']=='hide' && !nouserhide) {
                $linecolor = '#ffdddd';
            }

		?>
<tr bgcolor="<? echo $linecolor; ?>" align="left">
	<td valign="top" nowrap>
	    <? if ($thisuserid==$person['usn'] || $administrator) {
	        ?><a href = <? echo '"' . user_management_prog . '?UID=' . $UID . '&usn=' . $person['usn'] . '&submit=editmember&listmembers=1"'; ?> >
	        <img src="white.gif" height=4 width=1 border=0><br><img src="pencil.gif" align="top" border=0></a><img scr="white.gif" height=1 width=8 border=0>
	        <?
        } else { ?>
            <img scr="white.gif" height=1 width=8 border=0>
        <? } ?>
    </td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>">
		<a href="<? echo list_signups_prog . '?UID=' . $UID . '&submit=listuser&username=' . make_get_str($username); ?>">
		<? echo $person['lastname']; ?>
		</a></font></td>
	<td></td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>">
		<? echo $person['firstname']; ?>
		</a></font></td>
	<td></td>
	<td valign="top"       ><font size="-2"> <? echo $person['ratings']; ?> </font></td>
	<td></td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>"> <? echo $person['homephone']; ?> </font></td>
	<td></td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>"> <? echo $person['workphone']; ?> </font></td>
	<td></td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>">
				<? echo "<a href=\"mailto:{$person['email1']}\">{$person['email1']}</a>"; ?>
		</font></td>
	<td></td>
	<td valign="top" nowrap><font size="<?echo $fontsize; ?>">
				<? echo "<a href=\"mailto:{$person['email2']}\">{$person['email2']}</a>"; ?>
		</font></td>
	</tr>
<? }
}
echo "</table>";

scheduleunlock();
printfooter();

//--------------------------------------
//JMS 11/17/03
// -re-secured access logic
//JMS 7/29/04
//  -added "hide" for users (ratings = "hide" to hide name)
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!