<div align="center">

## SERVER\-TO\-SERVER site \(or single dir\) transfer


</div>

### Description

Place this script in a directory of your site and it will transfer via ftp all nested dirs and files to another server (of course it may be the same server).Don't lose time downloading and the uploading.Let the server do it all.Instructions and comments to code included.
 
### More Info
 
no input needed.

Just put the script in the directory you want to transfer.

FTP function must be available on source server. Destination server must be accessible via FTP.

Status messages.

No side effects known.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Raffaele Marranzini](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/raffaele-marranzini.md)
**Level**          |Intermediate
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |PHP 3\.0, PHP 4\.0
**Category**       |[Files](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/files__8-2.md)
**World**          |[PHP](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/php.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/raffaele-marranzini-server-to-server-site-or-single-dir-transfer__8-780/archive/master.zip)





### Source Code

```
<?php
/*
Note: my english is not very good.sorry! ;-)
Hi there,
i was bored in a sunday afternoon and so i created this script... ;-)
Hope you find it useful..
It simplified the task of moving a site from a server to another...
I'm behind a 56k modem, not enought bandwidth to transfer a whole site
in a reasonable time. ;-)
*/
//------------------------------------------------------------------------------
/*              INSTALLATION
   1 - Edit "Configuration",a few lines below this.Save changes.
   2 - Upload this file to the directory you want to transfer.
     Of course, also all nested dirs and files will be transfered.
     Note that no files will be deleted from source server.
     You have to do this manually.
   3 - Point your browser to this file.
   4 - Wait until page is loaded( "! Done !" will appear ).
     This may take a few.That's it.
               REQUIREMENTS
     * This script uses ftp function so ,on source server, php
     must support ftp function.
     * You must have permission on source server to read files and dirs you
     want to transfer. i.e. fopen(file, "r") must not fail.
     * You must have FTP access to destination server.
                LICENSE
  DCAR License agreement:
  I *D*on't *C*are *A* *R*ap of what you do with my script.
  But if you gain a billion EUROs with this, please remember of an old friend.
  NOTE: I'm not responsible for any damage this script may cause ;-)
  Jokes apart: this is only a beta version but ,for me, has worked perfectly.
  blu_aqua@libero.it
*/
//------------------------------------------------------------------------------
//------------------------------------------------------------------------------
//     START CONFIGURATION
//------------------------------------------------------------------------------
  // ftp address of destination server
  // ex: ftp.mysite.com
   $ftp_server = "";
  // user name to access ftp destination server
  // ex: administrator
   $ftp_user = "";
  // password to access ftp destination server
  // ex: secret
   $ftp_pass = "";
  // *ABSOLUTE* (means "starting from the root") FTP path to the directory
  // where local file system will be replicated.
  // Path must begin with / and must *NOT* end with /
  // ex: /html
   $ftp_initial_dir = "";
//------------------------------------------------------------------------------
//     END Configuration
//------------------------------------------------------------------------------
//uncomment this if you wish some debugging and comment subsequent line
//error_reporting(E_ALL);
error_reporting(0);
//******************************************************************************
//   WARNING : You must not edit anything under this line
//   (of course, if you need, you can modify the code)
//******************************************************************************
//------------------------------------------------------------------------------
//      Utility functions
//------------------------------------------------------------------------------
// function to forbid recursion through particular dirs
function not_black_list($dir)
{
 $black_list = array();
 if(in_array($dir, $black_list))
 return false;
 else
 return true;
}
// my ftp_mkdir ......
function lello_mkdir($dir, $ftp_stream)
{
 global $ftp_server, $ftp_user, $ftp_pass, $ftp_initial_dir;
 //removing trailing backslash
 if(strrpos($dir, "/") == strlen($dir) && strlen($dir) > 0)
 $dir = substr($dir,0,strrpos($dir, "/")-1);
 $tree = explode("/", $ftp_initial_dir.$dir);
 for($i=0; $i<count($tree); $i++)
 {
  if($tree[$i] != "" && $tree[$i] != "/")
  {
   if($i == 1)
   $base = "/";
   else
   $base = "";
   if(ftp_chdir($ftp_stream, $base.$tree[$i]) == false)
   {
    ftp_mkdir($ftp_stream, $tree[$i]);
    ftp_chdir($ftp_stream, $tree[$i]);
   }
  }
 }
}
// ftp connection function
function connetti()
{
 global $ftp_server, $ftp_user, $ftp_pass, $ftp_initial_dir;
 $conn_id = ftp_connect("$ftp_server");
 $login_result = ftp_login($conn_id, "$ftp_user", "$ftp_pass");
 if ((!$conn_id) || (!$login_result))
 die("Ftp connection error!<br> server name: $ftp_server <br> user name: $ftp_user <br> password: $ftp_pass ");
 else
 {
  lello_mkdir("", $conn_id);
  return $conn_id;
 }
}
// my ftp_fput
function lello_put($file, $conn_id, $fp, $dir)
{
 //as you can see i don't like using reg expr ;-) ... add estensions if needed
 $img_type = array ("jpg","JPG","gif","GIF","png","PNG","jpeg","JPEG","swf","SWF","mov","MOV","mp3","MP3","mid","MID");
 $ext = explode(".",$file);
 if (in_array($ext[1],$img_type))
 $ftp_mode = FTP_BINARY;
 else
 $ftp_mode = FTP_ASCII;
 if(strpos($file, "%20") > 0)
 $file = str_replace("%20", " ", $file);
 if(ftp_fput($conn_id, $file, $fp, $ftp_mode) == false)
 echo("Error while transferring file: ".$dir.$file."<br>");
}
// file transferring function
function carica($dir, $file, $conn_id)
{
 global $local_base, $ftp_server, $ftp_user, $ftp_pass, $ftp_initial_dir;
 $fp = fopen($local_base.$dir."/".$file, "r");
 if($fp == false)
 echo("cannot open file: ".$local_base.$dir."/".$file."<br>");
 lello_mkdir($dir, $conn_id);
 lello_put($file, $conn_id, $fp, $dir);
}
//------------------------------------------------------------------------------
//      END - Utility functions
//------------------------------------------------------------------------------
//getting local base dir
$local_base = dirname($PATH_TRANSLATED);
if(strrpos($local_base, "/") == strlen($local_base))
 $local_base = substr($local_base,0,strrpos($local_base,"/"));
//ftp connection to destination server
$ftp_stream = connetti();
//main recursion throught filesystem
function RecurseDir($directory)
 {
 global $ftp_stream, $local_base;
 if ($dir = opendir($local_base.$directory))
  {
  $i = 0;
  while ($file = readdir($dir))
   {
   if (($file != ".")&&($file != ".."))
    {
	$tempDir = $directory."/".$file;
	if (is_dir($local_base."/".$tempDir))
     {
     if(not_black_list($file))
      RecurseDir($tempDir);
     }
    else
     {
     //file transfer
     carica($directory, $file, $ftp_stream);
	 }
 	$i++;
    }
   }//end while
  if ($i == 0)
   {
   // empty directory
   }
  }
 else
  {
  // directory could not be accessed
  echo "cannot open dir ".$local_base.$directory."<br>";
  }
}
?>
<!doctype html public "-//W3C//DTD HTML 4.0 //EN">
<html>
<head>
    <title>Hauling system</title>
</head>
<body>
<b>Activity Report</b><br>
<?php
//main call
RecurseDir();
//closing ftp connection
ftp_quit($ftp_stream);
?>
<br><br><b> ! DONE ! </b>
</body>
</html>
```

