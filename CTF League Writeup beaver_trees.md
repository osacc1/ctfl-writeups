# Fall Challenge 1: beaver_trees

**Category**: web  
**Points**: 125 (2 flags)  
**Description**:  
Welcome to CTF League! As a reminder, you are looking for flags in the format `osu{i_4M_A_fL4G}`. Once you find one, submit it in your team channel with /submit `<flag>`.
I am excited to launch my new business selling trees to beavers! Surely beavers don't know how to hack right?

---

# Approach

The given webpage is a simple storefront selling trees.

The first three things that stood out to me when looking over the webpage were the "Admin Login" button, the "Flag Tree" item, and the discount code. I guessed that the two flags would come from logging in as admin, and successfully purchasing the flag tree, which turned out to be correct.

# Flag 1

Attempting to use the Admin Login with the network monitor open revealed that the form does not send any requests to the server. This indicates that credentials are checked client-side. Looking through the page source reveals a helpfully labled "Admin Login Script" which includes the admin username and password in plaintext.

```js
function handleAdminLogin(event) {
    event.preventDefault();
    const username = document.getElementById('username').value;
    const password = document.getElementById('password').value;

    if (username === 'Admin' && password === 'Tr33_L0V3r!') {
        ...
```

Logging in with these credentials adds the following text to the page, giving us our first flag.

> \*\***Admin Note**\*\*
> Use discount code "**osu{...}**" to make any purchase free

# Flag 2

I figured I needed to use this new discount code to purchase the Flag Tree, but there's nowhere to input a discount code in the purchase dialog. Attempting to purchase the tree anyways with the network monitor on shows us that a `POST` request is sent to the server with the following payload:

```json
{"product":"Flag Tree","discountCode":null}
```

Sending a modified `POST` request with Flag 1 in the payload's `"discountCode"` field elicits the following response, giving us our second flag.

```json
{"confirmationCode":"osu{...}","newBalance":250.0,"success":"true"}
```

# Retrospect

Obtaining Flag 2 requires you to modify and resend a `POST` request. There are many ways to do this, but in this challenge my teammates and I used our browser's developer tools. My teammates using Chrome had significant difficulty with this part of the challenge, so I have included instructions for modifying a resending a network request in both Firefox and Chrome.

## Firefox

right-click request -> click "Edit and Resend" -> modify payload -> click "Send"

## Chrome

right-click request -> click "Copy" -> click "Copy as fetch" -> go to "Console" tab -> modify payload -> enter -> go to "Network" tab -> click request -> go to "Response" sub-tab
