---
layout: post
title: "Connect to a Remote Server and Setup a Jupyter Notebook with Port Forwarding"
date: 2025-01-21 10:00:00 +000
categories: til
tags: [ssh, linux, jupyter, port]
excerpt:
  "Some simple commands and instructions for connecting to remote linux server
  with ssh, setting up a jupyter notebook and then setting up port forwarding so you
  can edit the notebook on the remote machine directly locally in the browser or VSCode."
---

### Step 1. Set up an SSH tunnel:

From your local machine run the following bash command:

```bash
$ ssh -p [SSH PORT] -i [FILENAME].pem -L 8888:localhost:8888 [USERNAME]@[IP ADDRESS]
```

`-p [SSH PORT]`: specifies a custom SSH port. The default port is 22 but if the server is setup to use a different port, you must specify it.

`-i [FILENAME].pem`: specifies the file or path-to-file for the Privacy-Enhanced-Mail (.pem) file containing a private key which is paired with a public key on the server. You must make sure that the .pem file has the correct permissions (`chmod 400 /path/to/[FILENAME].pem` sets the file to read by the owner only) otherwise you will receive a "connection to open" error.
`-L 8888:localhost:8888`: maps port 8888 on your local machine to port 8888 on the remote machine. This allows you to access services running on the remote server's port 8888 as if they were local.
`[USERNAME]@[IP ADDRESS]`: connects to the server as user [USERNAME] at the specified IP address [IP ADDRESS].

### Step 2. Start the Jupyter notebook on the remote machine

From the ssh, on the remote machine run the following command:

```bash
$ jupyter notebook --ip=127.0.0.1 --port=8888 --no-browser
```

`--ip=127.0.0.1`: binds Jupyter to the localhost, making it accessible only from the server itself.
`--port=8888`: specifies the port, which matches th port forwarded via SSH.
`--no-browser`: stops Jupyter from opening a browser on the remote server

After running this command, Jupter will privide a URL like `http://127.0.0.1:8888/tree?token=<token>` which can be used locally to access the notebook. You can also use `http://localhost:8888`, because you have linked the local port 8888 with the remote port 888. Both these URLs can be used in a browser on you local machine to open the notebook interface to the remote machine.

If port 8888 is already in use by your local machine or the remote side (e.g. you have local notebooks running on port 8888), you can set a different port such as 9999 in the SSh and Jupyter commands.

### Step 3. [Optional] Set a zshrc alias

Setting an alias in `~/.zshrc` allows you to condense a long command or set of commands into a single command, allowing you to to connect to the remote server using ssh without typing out the long command each time.

```bash
$ alias connect_jupyter="ssh -p [SSH PORT] -i [FILENAME].pem -L 8888:localhost:8888 [USERNAME]@[IP ADDRESS]"

# reload the shell with the new alias
$ source ~/.zshrc

# connect with a single command
$connect_jupyter
```

You still have to set up the jupyter notebook, but this saves you a bit of trouble if you are connecting/re-connecting regularly.

### Step 4 [Optional]. Update the ~/.ssh/config

Updating the `~/.ssh/config` allows you to open the connection and start the jupyter notebook easily. The `~/.ssh/config` file allows you to define configurations for SSH connections. This streamlines repetitive SSH tasks, reduces errors and allows for easy scaling if you need to manage multiple servers with different configurations.

If `~/.ssh/config` doesn't exist, create it, and then add the following lines:

```txt
Host [NAME]
  HostName [IP ADDRESS]
  Port [SSH PORT]
  IdentityFile [PATH/TO/FILENAME].pem
  User [USERNAME]
  LocalForward 8888 localhost:8888
```

The `~/.ssh/config` can contain multiple host configurations.

Make sure the `config` file has the correct permissions (at least read by the file owner, you):

```bash
$ chmod 600 ~/.ssh/config
```

Then test the connection:

```bash
$ ssh NAME
```
