{{Languages|
{{en|Changing File Permissions}}
{{pt-br|Mudando Permissões de Arquivos}}
{{ja|ファイルパーミッションの変更}}
{{zh-cn|更改文件权限}}
}}

__TOC__
On computer filesystems, different files and directories have <strong>permissions</strong> that specify who and what can read, write, modify and access them. This is important because WordPress may need access to write to files in your <code>wp-content</code> directory to enable certain functions. 

== Permission Modes ==

<pre style="font-size:1.1em;font-weight:bold;border:1px solid #80C9FF;background-color:#BFE4FF;width:45%;color:#000;padding:.5em;">
  7       5     5
 user   group  world
 r+w+x  r+x    r+x
 4+2+1  4+0+1  4+0+1  = 755
</pre>

The permission mode is computed by adding up the following values for the user, the file group, and for everyone else.  The diagram shows how.
* '''R'''ead    4  - Allowed to read files
* '''W'''rite   2  - Allowed to write/modify files
*  e'''X'''ecute1  - Read/write/delete/modify/directory

<pre style="font-size:1.1em;font-weight:bold;border:1px solid #80FF80;background-color:#BFFFBF;width:45%;color:#000;padding:.5em;">
  7       4      4
 user   group  world
 r+w+x    r      r
 4+2+1  4+0+0  4+0+0  = 744
</pre>

=== Example Permission Modes ===
{|
|-  style="width:99%; margin:0 auto;" border="0" cellpadding="3" cellspacing="1" align="center" bgcolor="#eee" color="000" border="1"
! '''Mode''' !! '''Str Perms''' !! '''Explanation'''
|-
| '''0477''' ||-r--rwxrwx|| owner has read only (4), other and group has rwx (7)
|-
| '''0677''' || -rw-rwxrwx||owner has rw only(6), other and group has rwx (7)
|-
| '''0444''' || -r--r--r--||all have read only (4)
|-
| '''0666''' || -rw-rw-rw-||all have rw only (6)
|-
| '''0400''' || -r--------||owner has read only(4), group and others have no permission(0)
|-
| '''0600''' || -rw-------||owner has rw only, group and others have no permission
|-
| '''0470''' || -r--rwx---||owner has read only, group has rwx, others have no permission
|-
| '''0407''' || -r-----rwx||owner has read only, other has rwx, group has no permission
|-
| '''0670''' || -rw-rwx---||owner has rw only, group has rwx, others have no permission
|-
| '''0607''' || -rw----rwx||owner has rw only, group has no permission and others have rwx
|+See full list [http://www.askapache.com/security/chmod-stat.html#chmod-0-to-7777 0000 to 0777].
|}

== Permission Scheme for WordPress ==

Permissions will be different from host to host, so this guide only details general principles. It cannot cover all cases. This guide applies to servers running a standard setup (note, for shared hosting using "suexec" methods, see below).

Typically, all files should be owned by your user (ftp) account on your web server, and should be writable by that account. On shared hosts, files should never be owned by the webserver process itself (sometimes this is '''www''', or '''apache''', or '''nobody''' user).

Any file that needs write access from WordPress should be owned or group-owned by the user account used by the WordPress (which may be different than the server account). For example, you may have a user account that lets you FTP files back and forth to your server, but your server itself may run using a separate user, in a separate usergroup, such as '''dhapache''' or '''nobody'''. If WordPress is running as the FTP account, that account needs to have write access, i.e., be the owner of the files, or belong to a group that has write access. In the latter case, that would mean permissions are set more permissively than default (for example, 775 rather than 755 for folders, and 664 instead of 644).

The file and folder permissions of WordPress should be the same for most users, depending on the type of installation you performed and the umask settings of your system environment at the time of install.

{{Note| If you installed WordPress yourself, you likely do not need to modify file permissions.  Unless you are experiencing problems with permission errors, or you ''want to'', you probably should not mess with this.}}

{{Note 2| If you installed WordPress yourself, you likely DO need to modify file permissions.  Some files and directories should be "hardened" with stricter permissions, specifically, the wp-config.php file. This file is initially created with 644 permissions, and it's a hazard to leave it like that. See Security and Hardening. }}

Typically, all core WordPress files should be writable only by your user account (or the httpd account, if different). (Sometimes though, multiple ftp accounts are used to manage an install, and if all ftp users are known and trusted, i.e., not a shared host, then assigning group writable may be appropriate. Ask your server admin for more info.)  However, if you utilize mod_rewrite Permalinks or other .htaccess features you should make sure that WordPress can also write to your <code>/.htaccess</code> file.

If you want to use the built-in theme editor, all files need to be group writable.  Try using it before modifying file permissions, it should work. (This may be true if different users uploaded the WordPress package and the Plugin or Theme. This wouldn't be a problem for Plugin and Themes installed via the admin. When uploading files with different ftp users group writable is needed. On shared hosting, make sure the group is exclusive to users you trust... the apache user shouldn't be in the group and shouldn't own files.)

Some plugins require the /wp-content/ folder be made writeable, but in such cases they will let you know during installation. In some cases, this may require assigning 755 permissions.  The same is true for <code>/wp-content/cache/</code> and maybe <code>/wp-content/uploads/</code> (if you're using [[MultiSite]] you may also need to do this for <code>/wp-content/blogs.dir/</code>)

Additional directories under /wp-content/ should be documented by whatever plugin / theme requires them. Permissions will vary. 

<pre>
/   
|- index.php
|- wp-admin
|   `- wp-admin.css
|- wp-blog-header.php
|- wp-comments-post.php
|- wp-commentsrss2.php
|- wp-config.php
|- wp-content
|   |- cache
|   |- plugins
|   |- themes
|   `- uploads
|- wp-cron.php
|- wp-includes
`- xmlrpc.php
</pre>

=== Shared Hosting with suexec ===

The above may not apply to shared hosting systems that use the "suexec" approach for running PHP binaries. This is a popular approach used by many web hosts. For these systems, the php process runs as the owner of the php files themselves, allowing for a simpler configuration and a more secure environment for the specific case of shared hosting.

Note: suexec methods should NEVER be used on a single-site server configuration, they are more secure '''only''' for the specific case of shared hosting.

In such an suexec configuration, the correct permissions scheme is simple to understand.

* All files should be owned by the actual user's account, not the user account used for the httpd process. 
* Group ownership is irrelevant, unless there's specific group requirements for the web-server process permissions checking. This is not usually the case.
* All directories should be 755 or 750. 
* All files should be 644 or 640. Exception: wp-config.php should be 600 to prevent other users on the server from reading it.
* No directories should ever be given 777, even upload directories. Since the php process is running as the owner of the files, it gets the owners permissions and can write to even a 755 directory.

In this specific type setup, WordPress will detect that it can directly create files with the proper ownership, and so it will not ask for FTP credentials when upgrading or installing plugins.

== Using an FTP Client ==

[[FTP_Clients|FTP programs]] ("clients") allow you to set permissions for files and directories on your remote host. This function is often called <code>chmod</code> or <code>set permissions</code> in the program menu.

In a [[Installing_WordPress|WordPress install]], two files that you will probably want to alter are the index page, and the css which controls [[Blog_Design_and_Layout|the layout]]. Here's how you change index.php - ''the process is the same for any file''.

In the screenshot below, look at the last column - that shows the permissions. It looks a bit confusing, but for now just note the sequence of letters.
<br />
[[Image:podz_filezilla_12.gif|center|thumb|400px|Initial permissions]]

Right-click 'index.php' and select 'File Permissions'<br />
A popup screen will appear.
[[Image:podz_filezilla_13.gif|center|thumb|305px|Altering file permissions]]

Don't worry about the check boxes. Just delete the 'Numeric value:' and enter the number you need - in this case it's 666. Then click OK.
<br />
[[Image:podz_filezilla_14.gif|center|thumb|400px|Permissions have been altered]]
You can now see that the file permissions have been changed.

=== Unhide the hidden files ===

By default, most [[FTP_Clients|FTP Clients]], including [http://filezilla.sourceforge.net/ FileZilla], keep hidden files, those files beginning with a period (.), from being displayed.  But, at some point, you may need to see your hidden files so that you can change the permissions on that file.  For example, you may need to make your [[Glossary#.htaccess|.htaccess]] file, the file that controls [[Using Permalinks|permalinks]], writeable.  

To display hidden files in FileZilla, in it is necessary to select 'View' from the top menu, then select 'Show hidden files'.  The screen display of files will refresh and any previously hidden file should come into view.

To get FileZilla to always show hidden files - under Edit, Settings, Remote File List, check the Always show hidden files box.

In the latest version of Filezilla, the 'Show hidden files' option was moved to the 'Server' tab. Select 'Force show hidden files.'

== Using the Command Line ==

If you have shell/SSH access to your hosting account, you can use <code>chmod</code> to change file permissions, which is the preferred method for experienced users.  Before you start using <code>chmod</code> it would be recommended to read some tutorials to make sure you understand what you can achieve with it.  Setting incorrect permissions can take your site offline, so please take your time.

* [http://www.washington.edu/computing/unix/permissions.html Unix Permissions]
* [http://developer.apple.com/documentation/Darwin/Reference/ManPages/man1/chmod.1.html Apple Chmod Reference] 

You can make '''all''' the files in your <code>wp-content</code> directory writable in two steps, but before making every single file and folder writable you should first try safer alternatives like modifying just the directory.  Try each of these commands first and if they dont work then go recursive, which will make even your themes image files writable.  Replace '''DIR''' with the folder you want to write in

 chmod 746 -v DIR
 chmod 747 -v DIR
 chmod 756 -v DIR
 chmod 757 -v DIR
 chmod 764 -v DIR
 chmod 765 -v DIR
 chmod 766 -v DIR
 chmod 767 -v DIR

If those fail to allow you to write, try them all again in order, except this time replace -v with -R, which will recursively change each file located in the folder.  If after that you still cant write, you may now try 777.

=== About Chmod ===

<code>chmod</code> is a unix command that means "<strong>ch</strong>ange <strong>mod</strong>e" on a file. The <code>-R</code> flag means to apply the change to every file and directory inside of <code>wp-content</code>. 766 is the mode we are changing the directory to, it means that the directory is readable and writable by WordPress and any and all other users on your system. Finally, we have the name of the directory we are going to modify, <code>wp-content</code>.  If 766 doesn't work, you can try 777, which makes all files and folders readable, writable, and executable by all users, groups, and processes.

If you use [[Using Permalinks|Permalinks]] you should also change permissions of <tt>.htaccess</tt> to make sure that WordPress can update it when you change settings such as adding a new page, redirect, category, etc.. which requires updating the .htaccess file when mod_rewrite Permalinks are being used.

# Go to the main directory of WordPress
# Enter <code>chmod -v 666 .htaccess</code>

{{Note|From a security standpoint, even a small amount of protection is preferable to a world-writeable directory.  Start with low permissive settings like 744, working your way up until it works.  Only use 777 if necessary, and hopefully only for a temporary amount of time.}}

== The dangers of 777 ==

The crux of this permission issue is how your server is configured.  The username you use to FTP or SSH into your server is most likely not the username used by the server application itself to serve pages.

<pre style="font-size:1.2em;font-weight:bold;border:1px solid #FF9980;background-color:#FFCCBF;width:45%;color:#000;padding:.5em;">
  7      7      7
 user   group  world
 r+w+x  r+w+x  r+w+x
 4+2+1  4+2+1  4+2+1  = 777
</pre>

Often the Apache server is 'owned' by the '''dhapache''' or '''nobody''' user accounts.  These accounts have a limited amount of access to files on the server, for a very good reason.  By setting your personal files and folders owned by your user account to be World-Writable, you are literally making them World Writable.  Now the dhapache and nobody users that run your server, serving pages, executing php interpreters, etc.. will have full access to your user account files.  

This provides an avenue for someone to gain access to your files by hijacking basically any process on your server, this also includes any other users on your machine.   So you should think carefully about modifying permissions on your machine. I've never come across anything that needed more than 767, so when you see 777 ask why its necessary. 

=== The Worst Outcome ===

The worst that can happen as a result of using 777 permissions on a folder or even a file, is that if a malicious cracker or entity is able to upload a devious file or modify a current file to execute code, they will have complete control over your blog, including having your database information and password.

=== Find a Workaround ===

Its usually pretty easy to have the enhanced features provided by the impressive WordPress plugins available, without having to put yourself at risk.  Contact the Plugin author or your server support and request a workaround.

== Finding Secure File Permissions ==

The .htaccess file is one of the files that is accessed by the owner of the process running the server.  So if you set the permissions too low, then your server won't be able to access the file and will cause an error.  Therein lies the method to find the most secure settings.  Start too restrictive and increase the permissions until it works. 

=== Example Permission Settings ===

The following example has a ''custom compiled php-cgi binary'' and a ''custom php.ini'' file located in the cgi-bin directory for executing php scripts.  To prevent the interpreter and php.ini file from being accessed directly in a web browser they are protected with a .htaccess file.

Default Permissions (umask 022)

 644 -rw-r--r--  /home/user/wp-config.php
 644 -rw-r--r--  /home/user/cgi-bin/.htaccess
 644 -rw-r--r--  /home/user/cgi-bin/php.ini
 755 -rwxr-xr-x  /home/user/cgi-bin/php.cgi
 755 -rwxr-xr-x  /home/user/cgi-bin/php5.cgi

Secured Permissions

 600 -rw-------  /home/user/wp-config.php
 6'''0'''4 -rw----r--  /home/user/cgi-bin/.htaccess
 6'''00''' -rw-------  /home/user/cgi-bin/php.ini
 7'''11''' -rwx--x--x  /home/user/cgi-bin/php.cgi
 '''100''' ---x------  /home/user/cgi-bin/php5.cgi

==== .htaccess permissions ====

'''644 > 604''' - The bit allowing the group owner of the .htaccess file read permission was removed.   644 is normally required and recommended for .htaccess files.

==== php.ini permissions ====

'''644 > 600''' - Previously all groups and all users with access to the server could access the php.ini, even by just requesting it from the site.  The tricky thing is that because the php.ini file is only used by the php.cgi, we only needed to make sure the php.cgi process had access.   The php.cgi runs as the same user that owns both files, so that single user is now the only user able to access this file.

==== php.cgi permissions ====

'''755 > 711'''
This file is a compiled php-cgi binary used instead of mod_php or the default vanilla php provided by the hosting company.  The default permissions for this file are 755.

==== php5.cgi permissions ====

'''755 > 100''' - Because of the setup where the user account is the owner of the process running the php cgi, no other user or group needs access, so we disable all access except execution access.  This is interesting because it really works.  You can try reading the file, writing to the file, etc.. but the only access you have to this file is to run php scripts.  And as the owner of the file you can always change the permission modes back again.
 
 $ cat: php5.cgi: Permission denied
 ./php5.cgi:  Welcome

== See Also ==

* [http://wordpress.org/support/topic/50863 Support Forum thread]
* [[htaccess for subdirectories]]
* [http://codex.wordpress.org/Editing_wp-config.php#Override_of_default_file_permissions Override WordPress Default permissions]


[[Category:Installation]]
[[Category:Getting Started]]
[[Category:Advanced Topics]]
[[Category:Troubleshooting]]
[[Category:UI Link]]
