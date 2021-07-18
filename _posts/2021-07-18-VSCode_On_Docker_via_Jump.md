Long history short, in my workplace I had to use VSCode on a Container, running inside a docker host, which is accessible only via a Jump Server (security measures). Looked up at many tutorials on the net to get this done, but more or less all of them involved installing docker on my window laptop. Tinkered around a few steps and made it work. Below given is the step by step instruction on how to do this, if you had a similar problem.

![image-20210718203123454](C:\Users\sheikmoh\AppData\Roaming\Typora\typora-user-images\image-20210718203123454.png)

I've already setup passwordless login to the Jump server and docker host, there are many articles on doing this, so I am going to skip that part.

So first, start the container on the docker host and setup SSH Server in the container.

```bash
#On Container:
#install ssh server
apt update && apt install  openssh-server 

#change root password
passwd 

#Enable to login as root via ssh
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config

#Start SSH Server
/etc/init.d/ssh start
```

Once done get the IP address of the container with the following command

```bash
#From Host
docker inspect -f "{{ .NetworkSettings.IPAddress }}" bold_cori
```

Check if the login works with the IP identified in the earlier step.

```bash
ssh root@<container_IP}>
```

Next we need to install '[Remote - SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)' in VSCode. This help with connecting you local VSCode to VS Code Server on the remote system. 

![SSH Architecture](https://code.visualstudio.com/assets/docs/remote/ssh/architecture-ssh.png)

Configure `ssh_config` in VSCode, so that you can login without password to the main server 

```bash
Host Jump
  HostName <Jump_host_IP>
  User <Jump_host_username>
  IdentityFile <Location_of_id_rsa_configured_for_passwordless_login>
Host DockerHost
  HostName <DockerHost_IP>
  User <DockerHost_Username>
  ProxyCommand ssh.exe -q -W %h:%p Jump
  IdentityFile <Location_of_id_rsa_configured_for_passwordless_login>
Host DockerContainer
  HostName <DockerContainer_IP>
  User <DockerHost_Username>
  ProxyCommand ssh.exe -q -W %h:%p DockerHost
```

Once the setup is completed, login to the docker container from VSCode.

If you are using Jupyter with VSCode ensure to [Disable experiments](https://github.com/microsoft/vscode-python/issues/14977#issuecomment-831304980) to make sure the python connect works with .

```bash
#Edit in settings.json

{
    "notebook.experimental.useMarkdownRenderer": false,
    "terminal.integrated.experimentalLinkProvider": false,
    "python.languageServer": "Pylance",
    "terminal.integrated.shell.linux": "/bin/bash",
    "python.defaultInterpreterPath": "/usr/bin/python3",
    "python.experiments.enabled": false
}
```



Note: If re-connecting to a same IP address with a new container:

```bash
Edit path of "known_hosts" given in output window to remove entry for the IP address.
```

