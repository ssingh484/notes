
## First digit denotes whether the response is good, bad or incomplete:

  

|Range|Purpose|
|---|---|
|`1xx`|Positive Preliminary reply<br><br>The requested action is being initiated; expect another reply before proceeding with a new command. (The user-process sending another command before the completion reply would be in violation of protocol; but server-FTP processes should queue any commands that arrive while a preceding command is in progress.) This type of reply can be used to indicate that the command was accepted and the user-process may now pay attention to the data connections, for implementations where simultaneous monitoring is difficult. The server-FTP process may send at most, one 1xx reply per command.|
|`2xx`|Positive Completion reply<br><br>The requested action has been successfully completed. A new request may be initiated.|
|`3xx`|Positive Intermediate reply<br><br>The command has been accepted, but the requested action is being held in abeyance, pending receipt of further information. The user should send another command specifying this information. This reply is used in command sequence groups.|
|`4xx`|Transient Negative Completion reply<br><br>The command was not accepted and the requested action did not take place, but the error condition is temporary and the action may be requested again. The user should return to the beginning of the command sequence, if any. It is difficult to assign a meaning to "transient", particularly when two distinct sites (Server- and User-processes) have to agree on the interpretation. Each reply in the 4xx category might have a slightly different time value, but the intent is that the user-process is encouraged to try again. A rule of thumb in determining if a reply fits into the 4xx or the 5xx (Permanent Negative) category is that replies are 4xx if the commands can be repeated without any change in command form or in properties of the User or Server (e.g., the command is spelled the same with the same arguments used; the user does not change his file access or user name; the server does not put up a new implementation.)|
|`5xx`|Permanent Negative Completion reply<br><br>The command was not accepted and the requested action did not take place. The User-process is discouraged from repeating the exact request (in the same sequence). Even some "permanent" error conditions can be corrected, so the human user may want to direct his User-process to reinitiate the command sequence by direct action at some point in the future (e.g., after the spelling has been changed, or the user has altered his directory status.)|
|`6xx`|Protected reply<br><br>RFC 2228 introduced the concept of protected replies to increase security over FTP communications. The 6xx replies are [Base64](https://en.wikipedia.org/wiki/Base64 "Base64") encoded protected messages that serves as responses to secure commands. When properly decoded, these replies fall into the above categories.|

## Second digit is a grouping digit and encodes the following information:

|Range|Purpose|
|---|---|
|`x0x`|Syntax<br><br>These replies refer to syntax errors, syntactically correct commands that don't fit any functional category, unimplemented or superfluous commands.|
|`x1x`|Information<br><br>These are replies to requests for information, such as status or help.|
|`x2x`|Connections<br><br>Replies referring to the control and data connections.|
|`x3x`|Authentication and accounting<br><br>Replies for the login process and accounting procedures.|
|`x4x`|Unspecified as of RFC 959.|
|`x5x`|File system<br><br>These replies indicate the status of the Server file system vis-a-vis the requested transfer or other file system action.|

## ALL Codes

|Code|Explanation|
|---|---|
|`100 Series`|**The requested action is being initiated, expect another reply before proceeding with a new command.**|
|`110`|Restart marker replay . In this case, the text is exact and not left to the particular implementation; it must read: `MARK yyyy = mmmm` where yyyy is User-process data stream marker, and mmmm server's equivalent marker (note the spaces between markers and "=").|
|`120`|Service ready in nnn minutes.|
|`125`|Data connection already open; transfer starting.|
|`150`|File status okay; about to open data connection.|
|`200 Series`|**The requested action has been successfully completed.**|
|`202`|Command not implemented, superfluous at this site.|
|`211`|System status, or system help reply.|
|`212`|Directory status.|
|`213`|File status.|
|`214`|Help message. Explains how to use the server or the meaning of a particular non-standard command. This reply is useful only to the human user.|
|`215`|NAME system type. Where NAME is an official system name from the [registry](https://www.iana.org/assignments/operating-system-names) kept by [IANA](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority "Internet Assigned Numbers Authority").|
|`220`|Service ready for new user.|
|`221`|Service closing control connection. Logged out if appropriate.|
|`225`|Data connection open; no transfer in progress.|
|`226`|Closing data connection. Requested file action successful (for example, file transfer or file abort).|
|`227`|Entering Passive Mode (h1,h2,h3,h4,p1,p2).|
|`228`|Entering Long Passive Mode (long address, port).|
|`229`|Entering Extended Passive Mode (\|port\|).|
|`230`|User logged in, proceed.|
|`232`|User logged in, authorized by security data exchange.|
|`234`|Server accepts the security mechanism specified by the client; no security data needs to be exchanged.|
|`235`|Server accepts the security data given by the client; no further security data needs to be exchanged.|
|`250`|Requested file action okay, completed.|
|`257`|"PATHNAME" created.|
|`300 Series`|**The command has been accepted, but the requested action is on hold, pending receipt of further information.**|
|`331`|User name okay, need password.|
|`332`|Need account for login.|
|`334`|Server accepts the security mechanism specified by the client; some security data needs to be exchanged.|
|`335`|Server accepts the security data given by the client; more security data needs to be exchanged.|
|`336`|Username okay, need password. [Challenge](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication "Challenge–response authentication") is "....".|
|`350`|Requested file action pending further information|
|`400 Series`|**The command was not accepted and the requested action did not take place, but the error condition is temporary and the action may be requested again.**|
|`421`|Service not available, closing control connection. This may be a reply to any command if the service knows it must shut down.|
|`425`|Can't open data connection.|
|`426`|Connection closed; transfer aborted.|
|`430`|Invalid username or password|
|`431`|Need some unavailable resource to process security.|
|`434`|Requested host unavailable.|
|`450`|Requested file action not taken.|
|`451`|Requested action aborted. Local error in processing.|
|`452`|Requested action not taken. Insufficient storage space in system. File unavailable (e.g., file busy).|
|`500 Series`|**Syntax error, command unrecognized and the requested action did not take place. This may include errors such as command line too long.**|
|`501`|Syntax error in parameters or arguments.|
|`502`|Command not implemented.|
|`503`|Bad sequence of commands.|
|`504`|Command not implemented for that parameter.|
|`530`|Not logged in.|
|`532`|Need account for storing files.|
|`533`|Command protection level denied for policy reasons.|
|`534`|Request denied for policy reasons.|
|`535`|Failed security check.|
|`536`|Data protection level not supported by security mechanism.|
|`537`|Command protection level not supported by security mechanism.|
|`550`|Requested action not taken. File unavailable (e.g., file not found, no access).|
|`551`|Requested action aborted. Page type unknown.|
|`552`|Requested file action aborted. Exceeded storage allocation (for current directory or dataset).|
|`553`|Requested action not taken. File name not allowed.|
|`600 Series`|**Replies regarding confidentiality and integrity**|
|`631`|Integrity protected reply.|
|`632`|Confidentiality and integrity protected reply.|
|`633`|Confidentiality protected reply.|
|`10000 Series`|**Common Winsock Error Codes**[[2]](https://en.wikipedia.org/wiki/List_of_FTP_server_return_codes#cite_note-2) _(These are not FTP return codes)_|
|`10054`|Connection reset by peer. The connection was forcibly closed by the remote host.|
|`10060`|Cannot connect to remote server.|
|`10061`|Cannot connect to remote server. The connection is actively refused by the server.|
|`10065`|No route to host / DNS cannot be resolved.|
|`10066`|Directory not empty.|
|`10068`|Too many users, server is full.|