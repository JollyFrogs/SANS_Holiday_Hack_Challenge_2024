Click the snowball on your avatar -> 'Story' -> scroll down to 'ACT I' and click 'Go to Act I':
 
> ![Pasted image 20241127232940](attachments/Pasted%20image%2020241127232940.png)
 
Click the 'cURLing' object to start the cURLing challenge:

> ![Pasted image 20241127233314](attachments/Pasted%20image%2020241127233314.png)
> 
> ### Silver Medal
> 
> Enter the following commands in the terminal's bottom panel in order to progress the challenge
> 
> Are you ready to begin? [y]es: 
> 
```
y
```
> 
> 1) Unlike the defined standards of a curling sheet, embedded devices often have web servers on non-standard ports.  Use curl to retrieve the web page on host "curlingfun" port 8080.
> If you need help, run the 'hint' command.

```
curl curlingfun:8080
```

> 2) Embedded devices often use self-signed certificates, where your browser will not trust the certificate presented.  Use curl to retrieve the TLS-protected web page at https://curlingfun:9090/
 
```
curl --insecure https://curlingfun:9090/
```

> 3) Working with APIs and embedded devices often requires making HTTP POST requests. Use curl to send a request to https://curlingfun:9090/ with the parameter "skip" set to the value "alabaster", declaring Alabaster as the team captain.

```
curl --insecure --data "skip=alabaster" https://curlingfun:9090/
```

> 4) Working with APIs and embedded devices often requires maintaining session state by passing a cookie.  Use curl to send a request to https://curlingfun:9090/ with a cookie called "end" with the value "3", indicating we're on the third end of the curling match.

```
curl --insecure --cookie "end=3" https://curlingfun:9090/
```

> 5) Working with APIs and embedded devices sometimes requires working with raw HTTP headers.  Use curl to view the HTTP headers returned by a request to https://curlingfun:9090/

```
curl --insecure --head https://curlingfun:9090/
```

> 6) Working with APIs and embedded devices sometimes requires working with custom HTTP headers.  Use curl to send a request to https://curlingfun:9090/ with an HTTP header called "Stone" and the value "Granite".

```
curl --insecure --header "Stone: Granite" https://curlingfun:9090/
```

> 7) curl will modify your URL unless you tell it not to.  For example, use curl to retrieve the following URL containing special characters: https://curlingfun:9090/../../etc/hacks

```
curl --insecure --path-as-is https://curlingfun:9090/../../etc/hacks
```

> Great work! 
> 
> Once HHC grants your achievement, you may close this terminal.
> 
> After entering the last command, you are rewarded the silver medal.
> 
> ### Gold Medal
> 
> There is a file called 'HARD-MODE.txt' that explains the objective of Gold:

```
ls -al
```

> ~~~
> total 32
> drwxr-x--- 1 alabaster alabaster 4096 Oct 30 18:03 .
> drwxr-xr-x 1 root      root      4096 Oct 30 18:02 ..
> -rw-r--r-- 1 alabaster alabaster  220 Jan  6  2022 .bash_logout
> -rw-r--r-- 1 alabaster alabaster 3129 Oct 30 18:02 .bashrc
> -rw-r--r-- 1 alabaster alabaster  807 Jan  6  2022 .profile
> -rw-r--r-- 1 root      root       234 Oct 30 18:02 HARD-MODE.txt
> -rw-r--r-- 1 root      root       168 Oct 25 16:04 HELP
> ~~~

```
cat HARD-MODE.txt
```

> Prefer to skip ahead without guidance?  Use curl to craft a request meeting these requirements:
> - HTTP POST request to https://curlingfun:9090/
> - Parameter "skip" set to "bow"
> - Cookie "end" set to "10"
> - Header "Hack" set to "12ft"

```
curl --insecure --data "skip=bow" --cookie "end=10" --header "Hack: 12ft" https://curlingfun:9090/
```

> Excellent!  Now, use curl to access this URL: https://curlingfun:9090/../../etc/button

```
curl --insecure --path-as-is https://curlingfun:9090/../../etc/button
```

> Great!  Finally, use curl to access the page that this URL redirects to: https://curlingfun:9090/GoodSportsmanship

```
curl --insecure --location https://curlingfun:9090/GoodSportsmanship
```

> Excellent work, you have solved hard mode!  You may close this terminal once HHC grants your achievement.
> 
> The Gold medal is awarded after entering the last command.
> 
> Medal progress:
> 
> ![Pasted image 20241127233945](attachments/Pasted%20image%2020241127233945.png)