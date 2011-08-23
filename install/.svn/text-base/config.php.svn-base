**** Administrator Account Information ****
admin_password,p,letmein, Administrator Password. (Clear this setting to disable main admin account)
configpassword,p,superpass, Password to modify the program configuration (these settings) & restore from backups (Clear this setting to allow all admin access)
admin_email1,t,,Primary administrator e-mail address
admin_email2,t,,Secondary administrator e-mail address
warning_monitor,t,,Administrator address to which emails of serious warning violations should be sent

**** Server Settings ****
timezone,nt,0*60*60, Offset of web server from timezone of users,-24*60*60,24*60*60
remote_auth,b,False, Attempt remote-authorization of users? <b>WARNING!</b> NEVER turn on when NOT using standard WebServer Authorization - severe security risk will result! (True = expect username from web server)
exit_URL,t,http://ors.sourceforge.net/, URL of page which should be loaded when [Home] is selected

**** General Configuration Options ****
enable_login,b,True, Allow login at all? (if False&, login allowed only by administrators through utilities.php)
demomode, b, False, Demonstration mode (True = do not save changes to user info or resources&, disable e-mail and backups)
public_calendar, b, True, Public view - non-logged-in users can view calendar (note that if set to TRUE&, individual resources can still be disabled from public view in the Resource Management page)

wait_for_lock,nt,60,  How long should we wait for a scheduler lock to be removed before we do it forceably? (>60 seconds is recommended),10
min_password_size,n,4, Define the minimum number of digits for the password length,0,100
uid_valid,nt,30*60, How long before an inactive session is expired,3*60,3*60*60

defalut_user_sec,n,1, Default new user authorities (1 = can sign up resources&, 0 = can not),0,1111
blocknewresource,b,True, Intially&, users are blocked from newly-created resources (True = Signups are blocked&, False = Signups are permitted)
save_deleteduser_signups, b,True, What happens to a deleted user's signups (True = reassigned as user "Abandoned"&, False = delete them)

-General Appearance Settings-
long_comment,b,False, Provide extra-long field for comment entry
header_image,t,scheduling.gif, Name of image to use on login form (relative to scheduling folder)
login_form_color,t,#BBBB66, Color for login forms (hex HTML format: #rrggbb)
menu_color,t,#BBBB66, Color for menus (hex HTML format: #rrggbb)
calendar_head_color,t,#6699CC, Color for calendar heading and controls line (hex HTML format: #rrggbb)
weekday_color,t,#DDDDDD, Color for weekday column headers in the day and month views (hex HTML format: #rrggbb)
weekend_color,t,#DDDD66, Color for weekend day column headers in the day and month views (hex HTML format: #rrggbb)
info_fieldname,t,Ratings, Name for user-defined informational field in user table
comment_text,t,Comment, Description/Name for comment field in signup form

-Memberlist Permissions-
nouserhide,b,False, Disable "hide in memberlist" feature (True = users cannot hide themselves from memberlist)
allow_memberlist_view,b,True, Non-administrators can view the list of members/users (True = Yes)
show_owner_contact,b,True, Show contact information when editing or viewing another user's signup (True = Yes)

-E-Mail Announcement Permissions-
email_as_user,b,True, Send e-mail announcements as the logged-in user. If False&, announcements will be sent as set in [email_from] (see below). False may be necessary for some e-mail servers.
allow_email_any,b,False, Allow users without calendar authority to use the e-mail announcement feature?
allow_mass_email,b,False, Allow users without calendar authority to make e-mail announcements to all users or all users of selected resources?

-Recurring Signup Permissions-
recurring_enabled,b,False, Enable the Recurring Signups feature?
recurring_calauth,b,True, Allow modification of Recurring Signups by calendar-authorized users? (Recurring Signups must be enabled above&, False = Only administrators)

-Signup Action Permissions-
enable_noshow,b,True, Offer "no-show" for signups (only on signups for which the start time has already passed)

-Signup Check-Out / Status Settings-
manage_status,b,false, Enable advanced management of signup status and check-out / check-in times
require_approval,b,false, Require signups to be approved by a calendar authorized user or an administrator (manage_status must be True)
allow_checkout_edit,b,False, Allow regular users to edit checkout times (False = only calendar-authorized users may edit check-in/check-out times)
editable_status,b,True, Allow administrators to visually edit a signup's status (via an edit box)
checkout_desc,t,Check Out, Text to use for "Check Out" button (Default is "Check Out")
checkout_interval,nt,1*60*60, How long before a signup starts should "Check Out" be offered as button when editing? (0 = only after start time),0
return_desc,t,Return Resource, Text to use for "Return Resource" button (Default is "Return Resource")
hard_return,b,True, Return of resource should truncate signup to return time (can be used to shorten signups only)
enable_return,b,True, Offer "Return Resource" button after signup has started even if manage_status is false

-Alternates and Promotion/Demotion Settings-
automanage_alternates,b,True, Automated handling of signup priorities&, reporting and notifying of alternates (includes promotion and demotion e-mails&, prohibits sign-up "under" other signups&, and other priority references)
echo_promotion,b,True, Show on-screen the list of users promoted when deleting or updating
echo_demotion,b,True, Show on-screen the list of users demoted when signing-up&, deleting or updating
echo_prodding,b,True, Show on-screen messages prodding users to contact primaries when they sign up as an alternate

**** Email Contact Options ****
email_primaries,b,False, E-mail the primary signup-holder when someone signs up as alternate? (True = Yes)
email_from,t,unknown@nowhere.org, From/Reply address for e-mails
scheduling_URL,t,http://ors.sourceforge.net/scheduling/, Base URL of the scheduling program (used in e-mails only)
resmonitor_timelimit,nt, 0*60*60*24, A resource monitor will be notified of signup changes/additions/deletions made within this time frame (0 = no time limit)

**** Schedule / Calendar Options ****
num_signups,n,3, How many users may signup per time block? (= 1 primary + alternates),1,15
day_start,n,6, When does the day begin [24Hour Formatted Hour 0-23],0,23
day_end,n,22, When does the day end [24Hour Formatted Hour 1-24],1,24

signup_interval,n,0.5, [Hours] Minimum interval allowed for a signup (0.1-24),0.1,24
dayview_interval,n,0, [Hours] Time interval for rows of day view (0 = same as signup_interval),0,24
weekview_interval,n,0, [Hours] Time interval for rows of week view (0 = same as 2*signup_interval),0,24
twentyfourhour_clock,b,True, Use 24 hour clock time format? (True = Use a 24 hour clock&, False = am/pm)
eurodate,b,False, Use European (inverted) date format? (True = use d/m/y format&, False = use m/d/y format)
default_view,t,week, Default view: [day&, week&, or month]
default_resource_all,b,True, Default resource selection: True = All&, False = 1st resource in list (faster page load)
default_week_length,n,7, [Days] Number of days to show in a "week" view,1,30
vertical_view,b,true,Show day and week views as vertical tables? (False = Horizontal view)
noweekends,b,False, Hide weekend days in calendar?

limit_day_view,b,False, Show only selected resources in day view (True = Yes&, False = always show all resources)
show_blocked_resources,b,False, Users can see resources they can not sign up for
hard_available,b,True, Require open availibity to list a resource when "Available Only" is checked. (True = require no signups for resource&, False = require any open priority spot)

-Signup labeling options-
show_name,b,True, Logged-in users - Always label signups in the calendar with the name of the owner (True = always show name&, False = show only if nothing else is being shown)
show_comment,b,False, Logged-in users - Label signups in the calendar with their comment for (True = show comment&, False = do not show comment)
show_first_name,b,False, Logged-in users - Show first name instead of last name
show_full_name,b,False, Logged-in users - Show full name (instead of first or last name only)
public_show_name,b,True, Public view - label signups in calendar with name of the owner (True = show name&, False = NEVER show name)
public_show_comment,b,False, Public view - label signups in the calendar with their comment (True = show comment&, False = do not show comment)
public_show_first_name,b,True, Public view - show first name instead of last name
public_show_full_name,b,False, Public view - show full name instead of first or last name only (True = show full name&, False = show first/last name only)
hyphen_length,n,8, Use word-wrap and hyphenation for names or comments longer than this,5,100
color_from_status,b,False, Determine signup color from status (see signup status settings&, above) of signup (True = color from Status&, False = color from Priority)

**** Scheduler Rule Definitions ****
allow_signup_offset,nt,0*60*60, Offset from present time at which users can sign up for a resource (>0),0
calendarauth_as_others, b,True, Allow calendar-authorized users to sign-up as others: True = Can sign up as other users&, False = allowed only to edit other user's signups
allow_signup_as_others, b,False, Allow normal users to sign-up as others: True = General users allowed to sign up for other general users (but still can not edit them)
allow_past_edit,b,False, Allow edit of past signups by regular users (limited options)
edit_other_users, b,False, Allow normal users to edit other user's signups: True = General users allowed to edit other's signups&, False = Not permitted
min_signup_time,n,1, [Hours] Shortest time allowed for a signup (must be > signup_interval),.2
warn_long_signup,nt,7*60*60*24, Signups over this length will cause us to give a WARNING,0
longest_signup,nt,14*60*60*24, Signups over this length will be REJECTED,0
warn_future_signup,nt,8*30*60*60*24, Start time further off than this will give a WARNING,0
disallow_future_signup,nt,36*30*60*60*24, Start time further off than this will be REJECTED,0
allow_poststart_mod,b,True, Allow changes to start time of signup after start time has already passed? (True = Yes)
allow_backtoback_signups,b,True,Allow Back-To-Back reservations (True = consider as single reservation&, False = indicate as rule violation)
near_signup_violations,b,False, Are rule violations on signups ending within the next 24-hours enforced? (True = Yes)
hard_enforce_rules,b,True, Hard-enforce Rules? (True = Rule violations cause a signup to be rejected&, False = Give Warning Only)
- Account Expiration -
hard_account_expiration,b,True, Should expired and restricted accounts be prohibited from booking double-bookable resources? (True = Yes - expired user can not sign up at all&, False = Allow expired users to signup&, but only along with indicated double-book resource (e.g. instructor or trainer resource))
errormsg_acntexpired,t,Your account is expired.  Please contact an administrator to renable your account.,Error Message when account expires
expire_explanation,t,Your account can expire for a variety of reasons.  An administrator can modify your expiration date.,Description of account expiration
warn_account_expire,nt,30*60*60*24, Give warning when account expiration will occur within this time period,60*60*24
warnmsg_acntexpire,t,Your account is about to expire.  Please contact an administrator to ensure that your account remains active.,Text of warning given when an account is about to expire.
- Account Inactivity -
inactive_account,nt,90*60*60*24, Time period defining account inactivity if the user does not log in or create signups.,60*60*24
errormsg_acntinactive,t,Due to inactivity your account has been marked as inactive.  Please contact an administrator to ensure that they don't restrict (disable) your account.,Error Message when account becomes inactive
errormsg_acntrestricted,t,Your account is restricted.  Please contact an administrator to remove your account restrictions.,Error Message when account has been restricted by an administrator
- Late Deletion of Signups -
warn_late_delete,nt,1*24*60*60, Signups deleted within this length of time prior to start time will give a warning (0=disabled)
warnmsg_late_delete,t,If the cancellation is not due to a justifiable reason you may still be charged 1/2 of the time scheduled., Possible penalty/other info related to late deletion of signup
- General Signup Rule Information -
errormsg_general,t,Please Be Advised... Sign-ups for this user break the following rules:, Message displayed to notify user of warnings or errors
exempt_users,t,Maintenance, Define usernames for which the reservation validation rules will not apply (usually non-user accounts; Give as &lt;firstname&gt;&lt;lastname&gt; no spaces&, names separated with commas)
- Comment for Any Signup -
require_comment,b,False, Should we require users to enter comments for all reservations? True = require comment
errormsg_comment_required,t,requires a comment,Error Message when users don't enter a comment for signup (follows "This signup...")
missing_comment_fatal,b,False, Missing comments (overnight or regular) are considered a reason to reject a signup (Hard_enforce_rules&, above&, must also be "True")
- Comment for Overnight Signups -
require_overnight_comment,b,True, Should we require a user to enter a comment for an overnight reservation? True = require comment
errormsg_comment_missing,t,requires a comment detailing over-night use,Error Message when users don't enter a comment for overnight signups (follows "This signup...")
- Maximum number of Signups -
signupcount,n,4, Maximum number of primary signups allowed per user,1
errormsg_toomany_primary,t, Maximum of four (4) primary sign-ups more than 24 hours in advance,Error Message when users exceed max allowable primary signups
signupcount2,n,6,Maximum number of primary + alternate signups allowed per user,1
errormsg_toomany_total,t, Maximum of six (6) total sign-ups (primary and alternate) more than 24 hours in advance,Error Message when users exceed max allowable total (primary+alternate) signups
count_dbook,b,False,Should reservations for resources that can be legally doublebooked (see resource management) be counted for the max tally? True=yes
- Double-booking -
allow_doublebook,b,False, Are double-bookings allowed (same time-block on different resources or alternate to self)?
hard_enforce_doublebook,b,False, Should double-bookings be hard-enforced? (True = double-bookings are rejected; False = double-bookings only give warning)
errormsg_doublebooked,t,Multiple sign-ups for the same time-block(s) (different resources or alternate to self),Error Message when user doublebooks two resources for same time slot
- Weekend Rules (number of days) -
max_weekend_days_single,n,3, [Days] Maximum number of weekend days allowed in a single reservation,0
errormsg_maxweekend_single,t,includes more than 3 weekend days,Error Message when users single signup exceeds max weekend days (follows "This signup..."
max_weekend_days_all,n,4, [Days] Maximum number of weekend days allowed for all reservations combined,0
errormsg_maxweekend_all,t,Sign-ups may not include more than a total of four (4) full weekend days,Error Message for signups that exceed max weeked days
- Weekend Rules (number of signups) -
max_weekend_reservations,n,2, Maximum number of reservations that are allowed to include full weekend days,0
errormsg_maxweekend,t,No more than two (2) separate sign-ups may include full weekend days.,Error Message when user exceeds max number of signups that include full weekend days
- Maximum Signup Length (Per resource) -
errormsg_maxsignup,t,This may require approval.,Error Message when signup length exceeds maximum for resource (see resource management page)

**** Backup and Log Settings ****
backup_interval,nt,24*60*60, How long between creation of backup files on-site (0=disable),0
mail_backup,b,True, Email backup files to designated users?
mail_action_log,b,True, Email action log when backup is sent?
mail_signups,b,True, Email human-readable version of signup listing when backup is sent?
- Log Lengths -
calendar_history,nt,90*24*60*60, How long to keep old calendar entrys,60*60*24
uid_history,nt,15*24*60*60, How long to keep user login history,60*60*24
loglength,nt,15*24*60*60, How long to keep action log history,60*60*24
backup_history,nt,7*24*60*60, How long to keep backup files on-site,60*60*24

**** Debug Settings ****
debug_locks,b,False, File Lock Debugging: True = view file lock actions
debug_mail,b,False, Mail Debugging: True = view e-mail messages and do NOT send
actionlog_details,b,False, Detailed Action Log: True = include raw data in action log (= longer log)

