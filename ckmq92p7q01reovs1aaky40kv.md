## Disable Root And Password Login via SSH

I shouldn't need to tell you why login as root via SSH can be dangerous. If somebody (hacker/unwanted person) were to get into your server and they have root access, that can be a sign of trouble. You see with root access they can do virtually anything. So disabling root access can be a good move for your server security.

In order to disable root login, you must first ensure that there is a normal user account available for you to log in after root login is disabled.

**Creating a new user**


```bash
sudo adduser "username"
``` 

Change the `username` with the desired username for your new user. Also, don't apply quotes. After executing the command you'll be presented with a form asking for the user's information, just enter whatever you want.

Now that the user is created, open the `/etc/ssh/sshd_config` inside your favourite text editor.

```bash
sudo nano /etc/ssh/sshd_config
```

In this file under `Authentication`, you'll find `PermitRootLogin` you need to change it's value to `no`.

```
PermitRootLogin no
```

In case you also want to disable password-based authentication as well, you can also change the `PasswordAuthentication` to `no`

```
PasswordAuthentication no
```

But make sure you've set up ssh key-based authentication before disabling PasswordAuthentication. I'll link up the article here for setting up ssh-key-based authentication.

Now, you need to restart your `SSH` service.

```bash
sudo service ssh restart
```

