# Fall Challenge 2: sw-elevator

**Category**: misc  
**Points**: 130 (2 flags)  
**Description**:  
Presenting the SW Elevator, where each level is a linux command line challenge! Solve each one to find credentials for the next level. Make it to the top, and your dreams (of linux command line superiority) will come true!

---

# Solutions
## level 0

> [!NOTE] README
> Have you seen my cat?
> Once you find credentials, log in with su --login usernamehere

```bash
cd creds_level1 # change directory
cat creds_level1.txt # print contents of file
```
## level 1

> [!NOTE] README
> 99% of gamblers give up right before they find creds
> hint: find? grep? linux gives u tons of tools to run around directories like you're a little monkey and they're trees

For this challenge you can use `find` to correct file, or you can use `grep -r` to search all the files recursively for the user and password strings. The `-e` flag lets you search for multiple strings.

```bash
find -iname *level2*
# OR
grep -r -e "next user" -e "password"
```

## level 2

> [!NOTE] README
> rocky raccoon checked into his room...
> hint: grep.

```bash
grep creds_level3.txt -e "next user" -e "password"
```

## level 3

> [!NOTE] README
> Can you get creds_level4.sh to run?

`chmod` changes file permissions. `chmod +x` adds the execute permission for all users, allowing us to execute the script.

```bash
chmod +x creds_level4.sh
./creds_level4.sh
```

## level 4
> [!NOTE] README
> Have you seen my file?
> hint: ls has a lot of options :)

```bash
ls -a # list hidden files
cat .hidden_creds_level5/.creds_level5.txt
```

## level 5
> [!NOTE] README
> filename is weird!
> can u read it anyway?

```bash
cat 'look in here'/'creds in here!' # use single quotes for filenames with spaces
```

## level 6
> [!NOTE] README
> I saw a secret key in an environment variable :o

```bash
printenv
./creds_level7 $SECRET_KEY
# OR
strings creds_level7 | grep -e user -e password
```

## level 7
> [!NOTE] README
> Oh no ive been hacked and my files have been base64 encoded
>
> also, there's a flag thingy at the root. what could that be?

The `base64 -d` command decodes base64 encoded messages. 

```bash
cat creds_level8.txt | base64 -d
ls / # list contents of the root directory
cat /flag_1.txt
```

This gives us the first flag of the challenge.

## level 8
> [!NOTE] README
> I think I might've accidentally exposed my password when starting my server.
> Could you please check for me?

The `ps x` command shows running processes, and the command that invoked them, including in this case, `--username` and `--passoword` flags.

```bash
ps x
```

## level 9

This level begins inside vim, a text editor. To progress you must exit by typing:

```
:q
```

## level 10
> [!NOTE] README
> Woah! A wild executable file! I wonder if it has any STRINGS in it...

`strings` outputs all the printable character sequences that are at least 4 characters long and can be used to find human readable text in otherwise unreadable files, like exectutables.

```bash
strings creds_level11 | grep user
strings creds_level11 | grep password
```

## level 11
> [!NOTE] README
> Oh, no! My computer crashed while I was writing down the next credentials!
> Maybe you can recover them somehow?

When Vim crashes, as `.swp` file is created. For this challange, you can recover the `.swp` or just use strings to recover the credentials.

```bash
vim -r .creds_level12.txt.swp
# OR
strings .creds_level12.txt.swp
```

## level 12
> [!NOTE] README
> localhost is serving something really cool on port 42756. How can I reach it from the command line?

```bash
curl localhost:42756 # Retrieve credentials being served
```

## level 13
> [!NOTE] README
> Oops! All compressed tar archive!

```bash
tar -xf compressed_creds_level14.tar.gz # Decompress tar archieve
```

## level 14
> [!NOTE] README
> You made it to the penthouse!
>
> What's in /flag_2.txt?

```bash
cat /flag_2.txt
```

Finally, we have our second and final flag.

# Retrospective

When I was attempting this challenge, I did not get flag_1 on level_7 because I solved the level without reading the README. When I got to level 14 and listed the contents of the root I saw `flag_1.txt` was owned by level7, so I switched back to the level7 user, who's credentials I thankfully still had and retrieved the flag.

Always read the README.