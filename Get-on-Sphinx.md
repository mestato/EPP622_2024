In this section, we will go over how to log into Sphinx, one of two UTIA computational servers . 

# Server Access

1. As a student of the course, Dr. Staton has already submitted an OIT help ticket to add you to the ``statonutia`` group, so your account should be ready to go by the first day. If you signed up last minute or if you're having trouble logging in, please let Dr. Staton know as soon as possible. It takes about 24-48 hours for OIT to respond to a ticket.

2. Now you must set posixAccount attribute and associated metadata on your LDAP entry - this can be done by visiting [https://setuplinux.utk.edu/](https://setuplinux.utk.edu/). Make sure you click "Create Account" so you can access the server.

3. Log in (see below log-in instructions prior to this step)
```
ssh your_utk_username@sphinx.ag.utk.edu
```
**ssh** stands for secure shell, which allows you to securely login to a remote server.

The password for login will be the same as your UTK passcode. Please note that nothing appears as you type your password; this is expected, as it provides additional security. Press ``enter/return`` after typing. 

If you are a first-time user, you will receive the following message:
```
The authenticity of host 'sphinx.ag.utk.edu (160.36.238.143)' can't be established.
ECDSA key fingerprint is <string_of_numbers>.
Are you sure you want to continue connecting (yes/no/[fingerprint])
```
Type **yes** then enter. Now you have access to Sphinx!

4. Update your ``~/.bash_profile`` with your favorite text editor (i.e. ``nano`` or ``vim``) to include the following:
```
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

export SPACK_ROOT=/pickett_shared/spack
PATH=$PATH:$HOME/bin:$SPACK_ROOT/bin
. $SPACK_ROOT/share/spack/setup-env.sh

exec newgrp statonutia
```

# How to Log In From Off Campus

In order to log into sphinx (or later in the semester, ISAAC) from off campus, you must be connected to UTK's VPN, which can be done by installing Pulse Secure. Please follow [this link](https://utk.teamdynamix.com/TDClient/2277/OIT-Portal/KB/ArticleDet?ID=123517) for more details.




