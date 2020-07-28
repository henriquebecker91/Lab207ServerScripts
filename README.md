# Lab207ServerScripts

The main scripts in this repository are: `send_to`, `do_in`, `send_to_all`, and `do_in_all`. As these scripts connect first to the portal machine, they work no matter if you are inside the laboratory (UFRGS intranet) or outside of it (at home, for example).

To work, all scripts need:

1. Editing the variables `user_portal` and `user_server` inside them (`user_portal` is your user in `portal.inf.ufrgs.br`, and `user_server` is your user in the servsers, i.e., shiva, ramuh, etc...);
2. You need to be able to login into the portal machine from your machine without typing your password (in linux just do `$ ssh-copy-id user_portal@portal.inf.ufrgs.br` in your machine, with your `portal_user`, your password will be asked).
3. You need to be able to login into the servers from the portal machine without typing your password (connect to the portal machine by ssh and then do the same you did for the previous step, but for your `user_server`, and using the server name instead of `user_portal@portal.inf.ufrgs.br`, the server password will be asked).
4. The command `ssh` is necessary for all scripts, and `rsync` is needed for the `send_to*` scripts.

The `do_in` and `send_to` scripts take a single `server` as first argument.

```bash
$ do_in ramuh ls
<your home contents in ramuh>
$ touch empty_file
$ send_to ramuh empty_file
from local to portal
...
from portal to ramuh
...
$ do_in ramuh ls
empty_file ...
```

The files are sent to your home directory, and a copy of them is kept inside the portal. The script support multiple files and/or folders. However, sending a folder, deleting some files from inside your local copy, and sending again, will not have the desired effect (i.e., the `send_to` will NOT remove the files from the portal and server folders). To do this you need to remove the folder from *both* portal and the server. This is the natural behaviour of `rsync`, and a failsafe. Can be worked around sending tarballs/zips (explicit files are always replaced).

To execute commands with redirection using `do_in`:
```bash
$ do_in ramuh sh -c 'echo abc > redirected_file'
```
You can even refer to variables that only exist when the command is running this way (just be careful to use single quotes and escapte the dollar signs):
```bash
$ do_in ramuh sh -c 'for letter in a b c; do echo \$letter > \${letter}_file; done'
```

The `do_in_all` and `send_to_all` scripts are similar to `do_in` and `send_to` (respectively), but:

1. They do not take a server name just the files/commands.
2. **If you just do `do_in_all` with no arguments, you will see the motd of all amchines. It is a great way to know which ones are free to use.**
3. If you try to use `do_in`/`send_to` inside a loop you will discover that the portal machine blocks you for some seconds/minutes if you rapidly open and closes connections to it. The `*_all` scripts make a single connection to `portal` and then make a connection to each of the servers from within this connection (avoiding the problem).
4. You can make a symbolic link called either `do_in_new` or `do_in_old` to `do_in_all` (copying and renaming also works). If you call `do_in_new` it will only act on the new machines (`ramuh`, `ixion`, `yojimbo`, `odin`, `leviathan`). The same applies to `do_in_old` and the old machines (`ifrit`, `shiva`, `malboro`, `tiamat0`, `tiamat1`, `tiamat2`). Finally, the analogue behavior comes from symbolic links `send_to_new` and `send_to_old` (pointing to `send_to_all`).
5. If you want to work only over a specific subset of servers, you can define the variable `LAB207SERVERS`. If this variable is defined, then the servers listed are used by `do_in_all`/`send_to_all`, and the name of the script/link is ignored.

```bash
$ touch empty_file
$ LAB207SERVERS="ifrit shiva" send_to_all empty_file
from local to portal
...
from portal to ifrit
...
from portal to ramuh
...
$ LAB207SERVERS="ifrit shiva" send_to_new empty_file
from local to portal
...
from portal to ifrit
...
from portal to ramuh
...
```

