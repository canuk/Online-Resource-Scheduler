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


if (!recurring_enabled || (!$administrator && !recurring_calauth) || (!$calendarauth && recurring_calauth)) {
    echo "<h3>You are not authorized to perform this action</h3>";
    scheduleunlock();
    printfooter();
    return;
}

//Start by defining some useful functions
//-------------------------------------------
function getdefaults($resource) {    
    $autosignups = array();
    if (!file_exists(root_dir . $resource)) mkdir (root_dir . $resource, 0755);
    if (file_exists(root_dir . $resource . "/defaults.csv")) {
        $autosignups = readkeyedfile(root_dir . $resource . "/defaults.csv",
            "",
            array(),
            array(),
            array(),
            array("datekey","value","hour","length"));
    }
    return($autosignups);
}
//-------------------------------------------
function putdefaults($resource,$data) {
    if (demomode) return(null);
    writekeyedfile(root_dir . $resource . "/defaults.csv", $data);
}
//-------------------------------------------


//get current resource #
$resource = $_POST['resource'];

//initalize "warning" array (for bad submits)
$warning = array();

//Handle ACTIONS first
switch ($_POST['submit']) {

case "Add":
case "Update":

    //process add form to create new record for given resource

    $_POST['hour']   = 0+strtok($_POST['hour'],":") + 0+strtok(":")/60 + 12*!!stristr($_POST['hour'],'pm');
    $_POST['length']   = 0+strtok($_POST['length'],":") + 0+strtok(":")/60;

    //don't allow length to be > length of a day
    if ($_POST['length']>24-$_POST['hour']) $_POST['length']   = 24-$_POST['hour'];

    switch ($_POST['freq']) {
        case "once" :
            $item['datekey'] = "n/j/y";
            $item['value']   = $_POST['oneday'];
            if ($item['value']=="") $warning[] = 'Please enter a valid date for an "Occurring Once" event';
            break;
        case "daily" :
            $item['datekey'] = "1";
            $item['value']   = "1";
            break;
        case "monthly" :
            $item['datekey'] = "d";
            $item['value']   = $_POST['monthly'];
            if ($item['value']=="" || $item['value']>31) $warning[] = 'Please enter a valid day of the month for an "Occurring Monthly" event';
            break;
        case "every" :
            $item['datekey'] = "w";
            $item['value']   = 0+$_POST['every'];
            break;
    }
    $item['hour']     = $_POST['hour'];
    $item['length']   = $_POST['length'];
    $item['priority'] = $_POST['priority'];
    if (!empty($_POST['startdate']))
        $item['startdate'] = strtotime($_POST['startdate']);
    if (!empty($_POST['enddate']))
        $item['enddate']   = strtotime($_POST['enddate']);

    if ($_POST['user']==0 | $_POST['usn']=="") {
        $warning[] = 'User must be specified';
    } else {
        $item['usn'] = $_POST['usn'];
    }
    $item['comment'] = $_POST['comment'];
    
    if ($item['length']==0) $warning[] = 'Length must be > 0';
    if ($item['hour']>23) $warning[] = 'Start Time must be < 23';
    if ($item['priority']<1 || $item['priority']=="") $warning[] = 'Priority must be > 0';

    switch ($_POST['submit']) {
        case "Add":
            if (count($warning)==0) {
                //READY TO ADD
                $data = getdefaults($resource);
                $data[] = $item;
                putdefaults($resource,$data);
                $display = "list";
            } else {
                $display = "add";
            }
            break;
        case "Update":    
            if (count($warning)==0) {
                //READY TO ADD
                $data = getdefaults($resource);
                $data[$_POST['line']] = $item;
                putdefaults($resource,$data);
                $display = "list";
            } else {
                $display = "update";
            }
            break;
    }    
    break;

case "delete":
    $data = getdefaults($resource);
    unset($data[$_POST['line']]);
    putdefaults($resource,$data);
    $display = "list";
    break;

case "edit":

    $data = getdefaults($resource);
    $data = $data[$_POST['line']];

    //decode line to set up form for editing
    $_POST['hour']     = date(timeformat,mktime($data['hour'],($data['hour']-floor($data['hour']))*60));
    $_POST['length']   = date("H:i",mktime($data['length'],($data['length']-floor($data['length']))*60)); //always 24 hour format
    if ($data['length']==24) $_POST['length'] = "24:00";
    $_POST['priority'] = $data['priority'];
    $_POST['usn']      = $data['usn'];
    $_POST['comment']  = $data['comment'];

    if (!empty($data['startdate']))
        $_POST['startdate'] = date(dateformat . "/y",$data['startdate']);
    if (!empty($data['enddate']))
        $_POST['enddate']   = date(dateformat . "/y",$data['enddate']);        

    switch ($data['datekey']) {
        case "n/j/y" :
            $_POST['freq'] = "once";
            $_POST['oneday'] = $data['value'];
            break;
        case "d" :
            $_POST['freq'] = "monthly";
            $_POST['monthly'] = $data['value'];
            break;
        case "w" :
            $_POST['freq'] = "every";
            $_POST['every'] = $data['value'];
            break;
        default :
            $_POST['freq'] = "daily";
            break;
    }
    $display = "edit";
    break;
    
case "addevent":
    $display = "add";
    break;

default :
    $display = "list";
    break;

}

echo "<b>Recurring Signups:</b><p>";

//Decide what kind of display to do
switch ($display) {
    case "add" :
    case "edit" :

        if ($_POST['hour']=="")     $_POST['hour']     = 0;
        if ($_POST['length']=="")   $_POST['length']   = 0;
        if ($_POST['priority']=="") $_POST['priority'] = 1;

        if (count($warning) > 0) {
            echo "<font color='#990000'><b>The event could not be added due to the following reason(s):<br>";
            foreach ($warning as $oneline) echo "&bull; ". $oneline . "<br>";
            echo "</b></font>";
        }
        ?>
        <form name="addevent" method="post" action="recurring.php">
          <table border="0" cellspacing="0" cellpadding="6">
            <tr> 
              <td bgcolor="#eeeeff" valign="top"> 
                <ul>
                <li>
                  <font size="-1">
                  <b>Resource:</b> 

                  <? if ($display=="add") { ?>
                      <select name="resource">
                        <?
                        foreach (sortby($resourcelist,'order') as $oneitem) {
                            echo "<option value=\"" . $oneitem['resource'] . "\"";
                            if ($resource == $oneitem['resource']) echo " selected";
                            echo ">${oneitem['long']}</option>\n";
                        }
                        ?> 
                      </select>
                      </li>
                  <? } else {
                    echo "&nbsp;" . $resourcelist[$_POST['resource']]['long']; ?>
                    </li>
                    <input type='hidden' name='resource' value='<? echo $_POST['resource']; ?>'>
                  <? } ?>

                  <li>
                  <b>Occurring:<br></b> 
                  &nbsp;&nbsp;&nbsp;<input type="radio" name="freq" value="once" <? if ($_POST['freq']=="once" || $_POST['freq']=="") echo "checked"; ?>>
                  <b>Once</b> on 
                  &nbsp;&nbsp;&nbsp;<input type="text" name="oneday" id="oneday" size="13" maxlength="20" value="<? echo $_POST['oneday']; ?>" onChange="addevent.freq[0].checked=1">
                  (date in <? echo dateformat; ?>/yy format)
                    <script type="text/javascript">
                        Calendar.setup({
                            inputField     :    "oneday",          // id of the input field
                            ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                            weekNumbers    :    false,            //do not show week numbers
                            showOthers     :    true,             //show other months
                        });
                    </script>
                  <br>
                  &nbsp;&nbsp;&nbsp;<input type="radio" name="freq" value="daily" <? if ($_POST['freq']=="daily") echo "checked"; ?>>
                  <b>Daily</b> <br>
                  &nbsp;&nbsp;&nbsp;<input type="radio" name="freq" value="monthly" <? if ($_POST['freq']=="monthly") echo "checked"; ?>>
                  <b>Monthly</b> on day <input type="text" name="monthly" size="4" maxlength="4" value="<? echo $_POST['monthly']; ?>" onChange="addevent.freq[2].checked=1">
                  of the month<br>
                  &nbsp;&nbsp;&nbsp;<input type="radio" name="freq" value="every" <? if ($_POST['freq']=="every") echo "checked"; ?>><b> 
                    Every Week</b> on 
                    <select name="every" onChange="addevent.freq[3].checked=1">
                    <option value="0" <? if ($_POST['every']==0 || $_POST['every']=="") echo "selected"; ?>>Sunday</option>
                    <option value="1" <? if ($_POST['every']==1) echo "selected"; ?>>Monday</option>
                    <option value="2" <? if ($_POST['every']==2) echo "selected"; ?>>Tuesday</option>
                    <option value="3" <? if ($_POST['every']==3) echo "selected"; ?>>Wednesday</option>
                    <option value="4" <? if ($_POST['every']==4) echo "selected"; ?>>Thursday</option>
                    <option value="5" <? if ($_POST['every']==5) echo "selected"; ?>>Friday</option>
                    <option value="6" <? if ($_POST['every']==6) echo "selected"; ?>>Saturday</option>
                  </select>
                  </li><li>
                  <b>Start time:</b> 
                  <input type="text" name="hour" size="7" maxlength="9" value="<? echo date(timeformat,strtotime($_POST['hour'])); ?>">
                  &nbsp;&nbsp;&nbsp;
                  <b>Length:</b> 
                  <input type="text" name="length" size="7" maxlength="9" value="<? echo $_POST['length']; ?>">
                  [hours]

                  </li><li>
                  <b>Starting Date:</b> 
                  &nbsp;&nbsp;&nbsp;<input type="text" name="startdate" id="startdate" size="13" maxlength="20" value="<? echo $_POST['startdate']; ?>" onChange="if (addevent.startdate.value=='') addevent.enddate.value='';">
                  (date in <? echo dateformat; ?>/yy format, or blank = all dates)
                  <br>
                  <b>Ending Date:</b> 
                  &nbsp;&nbsp;&nbsp;<input type="text" name="enddate" id="enddate" size="13" maxlength="20" value="<? echo $_POST['enddate']; ?>">
                  (date in <? echo dateformat; ?>/yy format)<br>
                    <script type="text/javascript">
                        Calendar.setup({
                            inputField     :    "startdate",          // id of the input field
                            ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                            weekNumbers    :    false,            //do not show week numbers
                            showOthers     :    true,             //show other months
                        });
                    </script>
                    <script type="text/javascript">
                        Calendar.setup({
                            inputField     :    "enddate",          // id of the input field
                            ifFormat       :    "<? echo javadateformat; ?>/%y",       // format of the input field
                            weekNumbers    :    false,            //do not show week numbers
                            showOthers     :    true,             //show other months
                        });
                    </script>

                  </li><li>
                  <b>Priority/Number of slots to fill:</b> 
                  <input type="text" name="priority" size="3" maxlength="3" value="<? echo $_POST['priority']; ?>">
                  </li><li>
                  <b>User:</b> 
                  &nbsp;&nbsp;&nbsp;<input type="hidden" name="user" value="1">
                  <? usermenu($_POST['usn']); ?>
                  </li><li>
                  <b>Comment:</b> 
                    <? if (!long_comment) {
                        echo '<input type="text" name="comment" size="25" maxlength="80"';
                        echo 'value="' . stripcslashes($_POST['comment']) . '">';
                    } else {
                        echo '<br><textarea valign="top" name="comment" cols="35" rows="4">';
                        echo stripcslashes($_POST['comment']) . '</textarea>';
                    } ?>
                    </font>
                </ul>
            
                <input type='hidden' name='line' value='<? echo $_POST['line']; ?>'>
                <input type="hidden" name="UID" <?php echo "value=\"$UID\""; ?> >

                <? if ($display=="add") { ?>
                    <input type="submit" name="submit" value="Add">
                <? } else { ?>
                    <input type="submit" name="submit" value="Update">                
                <? } ?>
        		<input type="submit" name="submit" value="Cancel">

              </td>
            </tr>
          </table>
        </form>
        <?
        break;
    
    case "list":

        echo '<a href="recurring.php?UID=' . $UID . '&submit=addevent"><font size="-1">[Add New Recurring Signup]</font></a>';
        echo "<table cellpadding=3 cellspacing=0 border=1>\n";
        echo "<tr><td align='center'><b>Resource</b></td>\n";
        echo "<td align='center'><b>Frequency</b></td>\n";
        echo "<td align='center'><b>Date Range</b></td>\n";
        echo "<td align='center'><b>Start at</b></td>\n";
        echo "<td align='center'><b>Length</b></td>\n";
        echo "<td align='center'><b># Slots</b></td>\n";
        echo "<td align='center'><b>User Name</b></td>\n";
        echo "<td align='center'><b>Comment?</b></td>\n";
        echo "<td align='center'><b>Actions</b></td></tr>\n";

        foreach (sortby($resourcelist,'order') as $oneitem) {
            $data = getdefaults($oneitem['resource']);
            foreach ($data as $linenumber=>$oneline) {
                echo "<tr><td>" . $oneitem['short'] . "</td>\n";
        
                switch ($oneline['datekey']) {
                    case "n/j/y" :
                        $freq = "Once on " . $oneline['value'];
                        break;
                    case "d" :
                        $freq = "Monthly on day " . $oneline['value'];
                        break;
                    case "w" :
                        $days = array('Sun','Mon','Tue','Wed','Thu','Fri');
                        $freq = "Every " . $days[$oneline['value']];
                        break;
                    default :
                        $freq = "Daily";
                        break;
                }

                echo "<td>" . $freq . "&nbsp;</td>\n";
                echo "<td align='center'><font size=-1>";
                if(empty($oneline['enddate']) && empty($oneline['startdate'])) {
                    echo "&lt;all&gt;";
                } else {
                    if(!empty($oneline['startdate']))
                        echo date(dateformat,$oneline['startdate']);
                        
                    if(empty($oneline['enddate']))
                        echo " and after ";
                    else {
                        echo " until ";
                        echo date(dateformat,$oneline['enddate']);
                    }
                }
                    
                echo "</font></td>\n";                   
                echo "<td>" . date(timeformat,mktime($oneline['hour'],($oneline['hour']-floor($oneline['hour']))*60)) . "&nbsp;</td>\n";
                echo "<td>";
                if ($oneline['length']==24) echo "24:00";
                else echo date("H:i",mktime($oneline['length'],($oneline['length']-floor($oneline['length']))*60));
                echo "&nbsp;</td>\n";
                echo "<td align='center'>" . $oneline['priority'] . "&nbsp;</td>\n";
                echo "<td><font size=-1>";

                if ($oneline['usn']==0 || $oneline['usn']=="") {
                    echo "&lt;none&gt;";
                } else {
                    echo $users[$oneline['usn']]['firstname'] . "&nbsp;" . $users[$oneline['usn']]['lastname'];
                }
                
                echo "</font></td>\n";
                echo "<td align='center'><font size='-1'>";
                if (!empty($oneline['comment']))
                    echo "Yes";
                else
                    echo "&nbsp;";
                echo "</font></td>\n";
                echo "<td><font size='-1'>";
                echo "<a href='recurring.php?UID=" . $UID . "&submit=edit&resource=" . $oneitem['resource'] . "&line=" . $linenumber . "'>[Edit]</a>";
                echo "&nbsp;<a href='recurring.php?UID=" . $UID . "&submit=delete&resource=" . $oneitem['resource'] . "&line=" . $linenumber . "'";
                echo " onClick=\"return(confirm('Are you sure you want to delete future incidents of this recurring signup?'))\"";
                echo ">[Delete]</a>";
                echo "</font></td>\n";
                echo "</tr>";
            }
        }
        
        echo "</table>";
        ?><p><font size="-1"><b>Please Note:</b>
        <li>Changes to these settings apply only to days which have no other signups. Once a day has other signups, a recurring signup becomes a regular signup and is edited in the calendar view.</li>
        <li>Recurring signups <b>without a user assgnment</b> can <b>never</b> be edited (whether or not other signups exist on the given day).</li>
        <li>Recurring signups can not span days.</li>
        <li>Recurring signups for a specified user do not count against user signup limits until (and unless) another signup is created on the same day.</li>
        </font><?

}

scheduleunlock();
printfooter();

//===============================================
//JMS 1/4/04
//  -created
//JMS 4/23/04
//  -do not allow recurring signups to span days (also, fix 0:00-24:00 problem)
//JMS 7/29/04
//  -fixed bug in "monthly on day" mode
//CBW 7/17/2010
// - Changed $HTTP_POST_VARS to $_POST in order to be compatible with PHP 5 
// - Changed $HTTP_GET_VARS to $_GET in order to be compatible with PHP 5 
// - Thanks to Paul Reasenberg for helping to debug and test the PHP 5 fix!