# W1seGuy 

TWO CTF IN ONE DAY??!?!?! Perhaps I've lose my marbles after all...

## Skills used: 

## Prompt:
Your friend told me you were wise, but I don't believe them. Can you prove me wrong?

When you are ready, click the Start Machine button to fire up the Virtual Machine. Please allow 3-5 minutes for the VM to start fully.

The server is listening on port 1337 via TCP. You can connect to it using Netcat or any other tool you prefer.

## Recon:
For this challenge, we're given what seems to be like a Python decrypting program. To get the flag 1 and flag 2, we would need to provide a flag.txt file, containing what is seemingly the encrypted message

Cause life is short, I used rustscan `rustscan -a 10.201.95.22 -- A -sS -sV` to gather any information regarding the ports and the server.

[insert recon01.png]

Not surprisingly, UDP/TCP port (1337) is open along with SSH port (22), but interestingly enough, from our rustscan script, we got some 
