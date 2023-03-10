When working with remote servers over an SSH connection, it can be frustrating when the connection becomes unresponsive due to network issues. This is especially true when using a VPN to connect to a remote server, as disconnecting the VPN can cause the SSH session to become unresponsive and eventually timeout after several minutes.

Fortunately, there is a way to configure the SSH client to immediately fail the connection when it detects that the network is unreachable. In this blog post, we'll show you how to set up your SSH client to send keepalive packets and terminate the session if it does not receive a response from the server.

### Step 1: Open your SSH configuration file

The first step is to open your SSH configuration file. This file is usually located at ~/.ssh/config, and you can edit it using a text editor like Nano or Vim. If the file doesn't exist, you can create it by running the command:

``` bash
$ touch ~/.ssh/config
```

### Step 2: Set the ServerAliveInterval and ServerAliveCountMax options

In your SSH configuration file, add the following lines:


``` bash
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3

```


These options specify how often the SSH client should send a "keepalive" packet to the server to ensure that the connection is still active. The ServerAliveInterval option sets the number of seconds between keepalive packets, while the ServerAliveCountMax option sets the number of missed packets before the client assumes the connection is lost.

In this example, the SSH client will send a keepalive packet every 60 seconds, and if it does not receive a response after three consecutive packets, it will terminate the session.

### Step 3: Save and close the SSH configuration file

After adding the ServerAliveInterval and ServerAliveCountMax options to your SSH configuration file, save and close the file. You can do this by pressing Ctrl+X to exit Nano or Vim, then typing "Y" to save the changes and pressing Enter to confirm.

### Step 4: Test the SSH connection

Now that you've configured your SSH client to send keepalive packets, you can test the connection by connecting to a remote server and disconnecting your VPN. If the network becomes unreachable, the SSH session should immediately fail and return you to the command prompt.

### Conclusion:

Configuring your SSH client to send keepalive packets and terminate the session when the network is unreachable can help you avoid unresponsive SSH sessions and save you time and frustration. By following the steps outlined in this blog post, you can set up your SSH client to work more efficiently and reliably, even when working with remote servers over a VPN connection.

