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

//maintenance special submit requests

if ($administrator) {        //only allowed if Admin!

switch ($_POST['submit']) {

    //- - - - - - -

    case "listresources"  :
    case "deleteresource" :
    case "blockresource"  :
    case "allowresource"  :
    case "copyresource"   :
    case "confirmdeleteresource" :
    case "confirmblockresource" :
    case "confirmallowresource" :
    case "editresource"   :
    case "Updateeditresource" :
    case "Canceleditresource" :
    case "Canceladdresource" :
    case "addresource"    :
    case "Add" :
    case "copyresourceblock" :
    case "confirmcopyresourceblock"  :

        echo '<b>Resources:</b>';
        if (demomode) {echo "<br><font size=-1>Demo mode - saving disabled</font>";}
        echo "<p>\n";

        $resourcelist = getresources();

        if ($_POST['submit']=='addresource') {

            //first locate next available resource #
            for ($newitem = 1; !empty($resourcelist[$newitem]); $newitem++);
            $order = count($resourcelist)+1;

            $resourcelist[$newitem] = array(
                    "resource" => $newitem,
                    "long"     => "",
                    "short"    => "",
                    "initials" => "",
                    "defaultaccess" => !blocknewresource,
                    "order"    => $order);
            $_POST['resource'] = $newitem;
            //this now gets passed to the edit page below where the user can modify it...
        }

        if ($_POST['submit']=='Updateeditresource'
          && $_POST['long']==""
          && $_POST['short']==""
          && $_POST['initials']=="")
                $_POST['submit'] = "deleteresource";

        if ($_POST['submit']=='Updateeditresource' || $_POST['submit']=='Add') {
            //make changes to resource            
            $res = $_POST['resource'];

            $resourcelist[$res]['long']          = $_POST['long'];
            $resourcelist[$res]['short']         = $_POST['short'];
            $resourcelist[$res]['initials']      = $_POST['initials'];
            $resourcelist[$res]['defaultaccess'] = $_POST['defaultaccess']==1;
            $resourcelist[$res]['doublebook']    = $_POST['doublebook'];
            $resourcelist[$res]['maxsignup']     = $_POST['maxsignup'];
            $resourcelist[$res]['hidepublic']    = $_POST['hidepublic'];
            $resourcelist[$res]['comments']      = $_POST['comments'];
            $resourcelist[$res]['warnings']      = $_POST['warnings'];
            $resourcelist[$res]['popupwarning']  = $_POST['popupwarning'];

            $neword = $_POST['order'];
            $oldord = $_POST['xorder'];
            if ($neword == '') $neword = $oldord;
            if ($neword < 1) $neword = 1;
            if ($neword > count($resourcelist)) $neword = count($resourcelist);

            if ($neword != $oldord) { //change in order!
                if ($neword < $oldord) { //move up
                    foreach ($resourcelist as $index=>$resource)
                        if ($resource['order'] >= $neword & $resource['order'] < $oldord)
                            $resourcelist[$index]['order'] = $resourcelist[$index]['order']+1;
                } else {  //move down
                    foreach ($resourcelist as $index=>$resource)
                        if ($resource['order'] > $oldord & $resource['order'] <= $neword)
                            $resourcelist[$index]['order'] = $resourcelist[$index]['order']-1;
                }
            }
            $resourcelist[$res]['order'] = $neword;
            $resourcelist[$res]['resource'] = $res;

            putresources($resourcelist);

            if ($_POST['submit']=='Add') {
                //set user access
                foreach ($users as $usn=>$user) {
                    //$addres = count($resourcelist) - strlen($user['resourceblock']) + 1;
                    $addres = max(array_keys($resourcelist)) - strlen($user['resourceblock']) + 1;
                    if ($addres > 0)  //verify that our string is long enough (fill in "permitted" for missing bits)
                    {
                       for ($i=0; $i < $addres; $i++) $users[$usn]['resourceblock'] = $users[$usn]['resourceblock']."0";
                    }
                    if (!$resourcelist[$res]['defaultaccess']) $perm = "1"; else $perm = "0";
                    $users[$usn]['resourceblock'][$res] = $perm;
                }
                putusers($users);
            }
        }

        if ($_POST['submit']=='confirmdeleteresource') {

            $resource = $_POST['resource'];
            foreach ($users as $usn => $user) {              //get future signups for all users
                $signup = getusersignups ($usn,true);
                if (count($signup)>0)                        //and delete those for this resource
                    foreach ($signup as $item) if ($item['resource'] == $resource) {
                        $item['usn']        = $usn;
                        $item['thisuserid'] = $thisuserid;
                        if ($usn != $thisuserid) message("superdelete",$item);
                        deletesignup($item);
                    }
            }

            logaction('deleted resource',$thisuserid,$_POST);
            foreach ($resourcelist as $index=>$data)
                if ($data['order'] >= $resourcelist[$resource]['order'])
                    $resourcelist[$index]['order'] = $resourcelist[$index]['order']-1;
            unset($resourcelist[$resource]);
            putresources($resourcelist);
        }

        if ($_POST['submit']=='confirmcopyresourceblock') {
             //we want to change resourceB to hold same resource security blocking as resourceA
             $resA = $_POST['resource1'];
             $resB = $_POST['resource2'];
             foreach ($users as $usn=>$user){
                 $addres = max(array_keys($resourcelist)) - strlen($users[$usn]['resourceblock']) + 1;
                 if ($addres > 0)
                 {
                     for ($i=0; $i < $addres; $i++) $users[$usn]['resourceblock'] = $users[$usn]['resourceblock']."0";
                 }
                 $users[$usn]['resourceblock'][$resB] = $users[$usn]['resourceblock'][$resA];
             }
             putusers($users);
             echo "<table border=1 cellpadding=2 cellspacing=0><tr><td bgcolor=\"#aaffaa\">";
             echo "<B>Resource Permissions have been copied from resource ".$resA." to resource ".$resB."</B>";
             echo "</table>";
             logaction("Copied Resource Permissions from resource ".$resA." to resource ".$resB,$thisuserid,$data);

        }

        if ($_POST['submit']=='confirmblockresource') {
            foreach ($users as $usn=>$user)
            {
                $addres = max(array_keys($resourcelist)) - strlen($user['resourceblock']) + 1;
                //$addres = count($resourcelist) - strlen($user['resourceblock']) + 1;
                if ($addres > 0)
                {
                    for ($i=0; $i < $addres; $i++) $users[$usn]['resourceblock'] = $users[$usn]['resourceblock']."0";
                }
                if (checkpermissions($users[$usn]['resourceblock'],"0",$_POST['resource']))
                        $users[$usn]['resourceblock'][$_POST['resource']] = $users[$usn]['resourceblock'][$_POST['resource']] + "1";
            }
            putusers($users);
        }

        if ($_POST['submit']=='confirmallowresource') {
                foreach ($users as $usn=>$user)
                {
                        $addres = max(array_keys($resourcelist)) - strlen($user['resourceblock']) + 1;
                        //$addres = count($resourcelist) - strlen($user['resourceblock']) + 1;
                        if ($addres > 0)
                        {
                             for ($i=0; $i < $addres; $i++) $users[$usn]['resourceblock'] = $users[$usn]['resourceblock']."0";
                        }
                        if (checkpermissions($users[$usn]['resourceblock'],"1",$_POST['resource']))
                                $users[$usn]['resourceblock'][$_POST['resource']] = $users[$usn]['resourceblock'][$_POST['resource']] - "1";
                }
                putusers($users);
        }

        //- - - - - - - - - - - - - - - - - - - - - - - - 
        // display as appropriate
        
        if ($_POST['submit']=='editresource' || $_POST['submit']=='addresource' ) {
            $resource = $resourcelist[$_POST['resource']];
        
            echo '<form name="SignupForm" method="' . submit_method . '" action="' . resource_management_prog . '">';
            echo '<input type="hidden" name="UID"      value="' . $UID . '">';
            echo '<input type="hidden" name="resource" value="' . $resource['resource'] . '">';
            echo '<input type="hidden" name="xorder"   value="' . $resource['order'] . '">';

            if ($_POST['submit']=='addresource') {
                echo "<b>Adding Resource Number: " . $resource['order'] . "</b><br>\n";
                echo '<input type="hidden" name="mode"     value="addresource">';
            } else {
                echo "<b>Editing Resource Number: " . $resource['order'] . "</b><br>\n";
                echo '<input type="hidden" name="mode"     value="editresource">';
            }
            echo '<table border=1 cellspacing=0>';
            echo '<tr align="center" valign="bottom" bgcolor="#eeeeff">';
            echo '<td nowrap><b>&nbsp;Order (in list)&nbsp;</b></td>';
            echo '<td nowrap><b>&nbsp;Long Description&nbsp;</b></td>';
            echo '<td nowrap><b>&nbsp;Short Desc.&nbsp;</b></td>';
            echo '<td nowrap><b>&nbsp;Initials&nbsp;<font size=-1>(very short desc)</font></b></td>';
            echo '</tr>';

            $otherforminfo = resource_management_prog . '?UID=' . $UID . '&submit=';

            echo '<tr align="center">' . "\n";
            echo '<td nowrap><input type="text" size=3 maxlength=3 name="order" value="' . $resource['order'] . '"></td>' . "\n";
            echo '<td nowrap><input type="text" name="long" value="' . $resource['long'] . '"></td>' . "\n";
            echo '<td nowrap><input type="text" name="short" value="' . $resource['short'] . '"></td>' . "\n";
            echo '<td nowrap><input type="text" name="initials" value="' . $resource['initials'] . '"></td>' . "\n";
            echo '</tr>';

            echo '<tr align="center" bgcolor="#eeeeff">';
            if ($_POST['submit']=='addresource') {
                echo '<td nowrap><b>&nbsp;Current/New User Access?&nbsp;</b></td>';
            } else {
                echo '<td nowrap><b>&nbsp;New User Access?&nbsp;</b></td>';
            }
            echo '<td nowrap><b>&nbsp;Max Signup Time (Hrs)&nbsp;</b></td>';
            echo '<td nowrap><b>&nbsp;Double Book?&nbsp;</b></td>';
            echo '<td nowrap><b>&nbsp;Deny Public View?&nbsp;</b></td>';
            echo '</tr>';
            echo '<tr align="center">';
            echo '<td><input type="checkbox" name="defaultaccess" value="1" '; if ($resource['defaultaccess']) echo "checked"; echo '>Allow</td>' . "\n";
            echo '<td><input type="text" size =3 maxlength =3 name="maxsignup" value="' . $resource['maxsignup'] . '"></td>' . "\n";
            echo '<td><input type="checkbox" name="doublebook" value="1" '; if ($resource['doublebook']) echo "checked"; echo '>Allow</td>' . "\n";
            echo '<td><input type="checkbox" name="hidepublic" value="1" '; if ($resource['hidepublic']) echo "checked"; echo '>Deny</td>' . "\n";
            echo '</tr>' . "\n";

            echo '<tr><td colspan=4 bgcolor="#eeeeff">';
            echo '<b><img src="helpicon.gif">&nbsp;Comments:</b>';
            echo '</td></tr>' . "\n";
            echo '<tr><td colspan=4>';
            echo '<textarea name="comments" cols=55 rows=4>' . stripcslashes($resource['comments']) . '</textarea>';
            echo '</td></tr>' . "\n";

            echo '<tr><td colspan=4 bgcolor="#eeeeff">';
            echo '<b><img src="warnicon.gif">&nbsp;Warnings:</b>';
            echo '</td></tr>' . "\n";
            echo '<tr><td colspan=4>';
            echo '<textarea name="warnings" cols=55 rows=4>' . stripcslashes($resource['warnings']) . '</textarea>';
            echo '<br><input type="checkbox" name="popupwarning" value="1" '; 
                if ($resource['popupwarning']) echo "checked"; 
            echo '>Show as "popup" alert when user signs up for this resource' . "\n";
            echo '</td></tr>' . "\n";

            echo '<tr><td colspan=4 align="center">';
            if ($_POST['submit']=='addresource')
                echo '<input type="submit" name="submit" value="Add">';
            else
                echo '<input type="submit" name="submit" value="Update">';
            echo '&nbsp;<input type="submit" name="submit" value="Cancel">';
            echo '</td></tr>' . "\n";
            echo '</form>';
            echo '</table>';

            break;
        }

        if ($_POST['submit']=='copyresource') {
                if (!$_POST['resource2'])
                        echo '<font color="#990000"><b>Select resource TO WHICH access permissions should be copied</b></font><p>';
                else
                        echo '<font color="#990000"><b>Copy permissions FROM resource ' . $_POST['resource1'] . ' TO resource ' . $_POST['resource2'] . '?</b></font><p>';
        }

        if ($_POST['submit']=='deleteresource')
                echo '<font color="#990000"><b>Warning: Selected resource will be deleted and all signups on that resource will be erased!</b></font><p>';

        if ($_POST['submit']=='blockresource')
                echo '<font color="#550000"><b>Notice: Selected resource will be blocked (disallowed) for all users</b></font><p>';

        if ($_POST['submit']=='allowresource')
                echo '<font color="#550000"><b>Notice: Selected resource will be unblocked (allowed) for all users</b></font><p>';

        echo '<p><a href="' . resource_management_prog . '?UID=' . $UID . '&submit=addresource"><font size="-1">[Add New Resource]</font></a><br>';

        echo '<table border=1 cellspacing=0>';
        echo '<tr align="center" valign="bottom">';
        echo '<td><b>&nbsp;Order&nbsp;</b></td>';
        echo '<td><b>&nbsp;Long Description&nbsp;</b></td>';
        echo '<td><b>&nbsp;Short Desc.&nbsp;</b></td>';
        echo '<td><b>&nbsp;Initials&nbsp;</b></td>';
        echo '<td><b>&nbsp;New User<br>Access?&nbsp;</b></td>';
        echo '<td><b>&nbsp;Max Signup<br>Time (Hrs)&nbsp;</b></td>';
        echo '<td><b>&nbsp;Double<br>Book?&nbsp;</b></td>';
        echo '<td><b>&nbsp;Deny<br>Public View?&nbsp;</b></td>';
        echo '<td><b>&nbsp;Actions&nbsp;</b></td>';
        echo '<td><b>&nbsp;Access&nbsp;</b></td>';
        echo '</tr>';

        $otherforminfo = resource_management_prog . '?UID=' . $UID . '&submit=';

        foreach (sortby($resourcelist,'order') as $resource) {
                if (in_array($_POST['submit'],array('deleteresource','blockresource','allowresource')) & $resource['resource']==$_POST['resource']) {
                        echo '<tr align="center">';
                        echo '<td>&nbsp;<b>' . $resource['order'] . '</b>&nbsp;</td>';
                        echo '<td>';
                        if ($resource['comments']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="helpicon.gif" alt="(?)" align="right" border=0></a>';
                        if ($resource['warnings']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="warnicon.gif" alt="(!)" align="right" border=0></a>';
                        echo '&nbsp;<b>' . $resource['long'] . '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>' . $resource['short'] . '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>' . $resource['initials'] . '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>';
                        if ($resource['defaultaccess']) echo "Yes"; else echo "No";
                        echo '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>' . $resource['maxsignup'] . '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>';
                        if ($resource['doublebook']) echo "Yes"; else echo "No";
                        echo '</b>&nbsp;</td>';
                        echo '<td>&nbsp;<b>';
                        if ($resource['hidepublic']) echo "Yes"; else echo "No";
                        echo '</b>&nbsp;</td>';
                        echo '<td colspan=2>';

                        if ($_POST['submit']=="deleteresource")
                                echo '<b><a href=' . $otherforminfo . 'confirmdeleteresource&resource=' . $resource['resource'] . '>[Confirm&nbsp;Delete]</a></b>';

                        if ($_POST['submit']=="blockresource")
                                echo '<b><a href=' . $otherforminfo . 'confirmblockresource&resource=' . $resource['resource'] . '>[Confirm&nbsp;Block&nbsp;All]</a></b>';

                        if ($_POST['submit']=="allowresource")
                                echo '<b><a href=' . $otherforminfo . 'confirmallowresource&resource=' . $resource['resource'] . '>[Confirm&nbsp;Allow&nbsp;All]</a></b>';

                        echo '&nbsp;<font size="-1"><a href=' . $otherforminfo . 'listresources>[Cancel]</a></font>';
                        echo '</td>';
                        echo '</tr>';
                } elseif ($_POST['submit']=='copyresource') {
                        echo '<tr align="center">';
                        echo '<td>&nbsp;' . $resource['order'] . '&nbsp;</td>';
                        echo '<td>';
                        if ($resource['comments']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="helpicon.gif" alt="(?)" align="right" border=0></a>';
                        if ($resource['warnings']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="warnicon.gif" alt="(!)" align="right" border=0></a>';
                        echo '&nbsp;' . $resource['long'] . '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['short'] . '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['initials'] . '&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['defaultaccess']) echo "Yes"; else echo "No"; echo '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['maxsignup'] . '&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['doublebook']) echo "Yes"; else echo "No"; echo '</b>&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['hidepublic']) echo "Yes"; else echo "No"; echo '</b>&nbsp;</td>';
                        echo '<td>';
                        echo '<font size="-1"><a href=' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '>[Edit]</a></font>';
                        echo '&nbsp;<font size="-1"><a href=' . $otherforminfo . 'deleteresource&resource=' . $resource['resource'] . '>[Delete]</a></font>';
                        echo '</td><td>';
                        if ($resource['resource']==$_POST['resource1']){
                            echo '&nbsp;<font size="-1"><B>(Copy Permissions FROM here)</B></a></font>';
                            echo '&nbsp;<font size="-1"><a href=' . $otherforminfo . 'listresources>[Cancel]</a></font>';
                        } else {
                                if ($_POST['resource2']){
                                    if ($resource['resource']==$_POST['resource2']){
                                        echo ' <font size="-1"><B>(TO&nbsp;here)</B></a></font>';
                                        echo ' <b><a href=' . $otherforminfo . 'confirmcopyresourceblock&resource1='.$_POST['resource1'].'&resource2=' . $_POST['resource2'] . '>[Confirm]</a></b>';
                                    }
                                } else {
                                    echo ' <font size="-1"><B><a href=' . $otherforminfo . 'copyresource&resource1='.$_POST['resource1'].'&resource2=' . $resource['resource'] .'>[TO&nbsp;here]</B></a></font>';
                                }
                        }
                        echo '</td>';
                        echo '</tr>' . "\n";

                 } elseif (!in_array($_POST['submit'],array('editresource','deleteresource','blockresource','allowresource'))) {
                        echo '<tr align="center">';
                        echo '<td>&nbsp;' . $resource['order'] . '&nbsp;</td>';
                        echo '<td>';
                        if ($resource['comments']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="helpicon.gif" alt="(?)" align="right" border=0></a>';
                        if ($resource['warnings']!="") 
                            echo '<a href="' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '"><img src="warnicon.gif" alt="(!)" align="right" border=0></a>';
                        echo '&nbsp;' . $resource['long'];
                        echo '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['short'] . '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['initials'] . '&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['defaultaccess']) echo "Yes"; else echo "No"; echo '&nbsp;</td>';
                        echo '<td>&nbsp;' . $resource['maxsignup'] . '&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['doublebook']) echo "Yes"; else echo "No"; echo '</b>&nbsp;</td>';
                        echo '<td>&nbsp;'; if ($resource['hidepublic']) echo "Yes"; else echo "No"; echo '</b>&nbsp;</td>';
                        echo '<td>';
                        echo '<font size="-1"><a href=' . $otherforminfo . 'editresource&resource=' . $resource['resource'] . '>[Edit]</a></font>';
                        echo ' <font size="-1"><a href=' . $otherforminfo . 'deleteresource&resource=' . $resource['resource'] . '>[Delete]</a></font>';
                        echo '</td><td>';
                        echo '<font size="-1"><a href=' . $otherforminfo . 'blockresource&resource=' . $resource['resource'] . '>[Block&nbsp;All]</a></font>';
                        echo ' <font size="-1"><a href=' . $otherforminfo . 'allowresource&resource=' . $resource['resource'] . '>[Allow&nbsp;All]</a></font>';
                        echo ' <font size="-1"><a href=' . $otherforminfo . 'copyresource&resource1=' . $resource['resource'] . '>[Copy&nbsp;Permissions]</a></font>';
                        echo '</td>';
                        echo '</tr>';
                }
        }
        echo '</table>';
        if (!in_array($_POST['submit'],array('editresource'))) {
            echo '<img src="helpicon.gif" alt="(?)" border=0> = Extended comments exist for resource.<br>';
            echo '<img src="warnicon.gif" alt="(?)" border=0> = Warnings exist for resource.';
        }
        break;
    }
}

if ($calendarauth) {        //only allowed if calendarauthorized!

switch ($_POST['submit']) {

    //- - - - - - -

    case "listwarnings"  :
    case "editwarnings"  :
    case "Canceleditwarnings" :
    case "Updateeditwarnings" :
    
        echo '<b>Resources:</b>';
        if (demomode) {echo "<br><font size=-1>Demo mode - saving disabled</font>";}
        echo "<p>\n";

        $resourcelist = getresources();

        if ($_POST['submit']=='Canceleditwarnings')
                if ($resourcelist[$_POST['resource']]['long']==""
                  & $resourcelist[$_POST['resource']]['short']==""
                  & $resourcelist[$_POST['resource']]['initials']=="")
                        $_POST['submit'] = "confirmdeleteresource";

        if ($_POST['submit']=='Updateeditwarnings') {
                //make changes to resource
                $res = $_POST['resource'];

                $resourcelist[$res]['warnings']      = $_POST['warnings'];
                $resourcelist[$res]['popupwarning']      = $_POST['popupwarning'];

                putresources($resourcelist);
        }


        //- - - - - - - - - - - - - - - - - - - - - - - - 
        // display as appropriate
        
        if ($_POST['submit']=='editwarnings') {
            $resource = $resourcelist[$_POST['resource']];
        
            echo '<form name="SignupForm" method="' . submit_method . '" action="' . resource_management_prog . '">';
            echo '<input type="hidden" name="UID" value="' . $UID . '">';
            echo '<input type="hidden" name="mode" value="editwarnings">';
            echo '<input type="hidden" name="resource" value="' . $resource['resource'] . '">';

            echo '<table border=1 cellspacing=0>';
            echo '<tr align="center" valign="bottom" bgcolor="#eeeeff">';
            echo '<td nowrap><b>&nbsp;Resource&nbsp;</b></td>';
            echo '</tr>';

            $otherforminfo = resource_management_prog . '?UID=' . $UID . '&submit=';

            echo '<tr align="center">' . "\n";
            echo '<td nowrap>' . $resource['long'] . '</td>' . "\n";
            echo '</tr>';

            echo '<tr><td bgcolor="#eeeeff">';
            echo '<b><img src="warnicon.gif">&nbsp;Warnings:</b>';
            echo '</td></tr>' . "\n";
            echo '<tr><td colspan=4>';
            echo '<textarea name="warnings" cols=55 rows=4>' . stripcslashes($resource['warnings']) . '</textarea>';
            echo '<br><input type="checkbox" name="popupwarning" value="1" '; 
                if ($resource['popupwarning']) echo "checked"; 
            echo '>Show as "popup" alert when user signs up for this resource' . "\n";
            echo '</td></tr>' . "\n";

            echo '<tr><td align="center">';
            echo '<input type="submit" name="submit" value="Update">';
            echo '&nbsp;<input type="submit" name="submit" value="Cancel">';
            echo '</td></tr>' . "\n";
            echo '</form>';
            echo '</table>';

            break;
        }

        echo '<table border=1 cellspacing=0>';
        echo '<tr align="center" valign="bottom">';
        echo '<td><b>&nbsp;Long&nbsp;Description&nbsp;</b></td>';
        echo '<td><b>&nbsp;Warnings<br>&nbsp;Set?&nbsp;</b></td>';
        echo '<td><b>&nbsp;Actions&nbsp;</b></td>';
        echo '</tr>';

        $otherforminfo = resource_management_prog . '?UID=' . $UID . '&submit=';

        foreach (sortby($resourcelist,'order') as $resource) {
            echo '<tr align="center">';
            echo '<td>';
            echo '&nbsp;' . $resource['long'];
            echo '&nbsp;</td>';
            echo '<td align="center">&nbsp;';
            if ($resource['warnings']!="") 
                echo '<img src="warnicon.gif" alt="(!)" border=0>';
            echo '&nbsp;</td>';
            echo '<td>';
            echo '<font size="-1"><a href=' . $otherforminfo . 'editwarnings&resource=' . $resource['resource'] . '>[Edit]</a></font>';
            echo '</td>';
            echo '</tr>';
        }
        echo '</table>';
        break;
    }
}

scheduleunlock();
printfooter();
//----------------------------------------------
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!

