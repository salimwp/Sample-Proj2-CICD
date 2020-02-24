This project is designed to leverage a micro-frontend/micro-services architecture.

First, go into the proj2-cicd directory and launch the Ansible playbook there.
```sh
$ cd proj2-cicd
$ mkdir keys
$ ssh-keygen -f keys/myKey
$ ssh-add keys/myKey
$ ansible-playbook -i hosts -k -K cicd.yaml
```

Once the playbook has completed there will be 3 Git repos on the CICD server.

Log into the Jenkins servers WebUI https://192.168.50.30 and select 'Credentials'. You will see a list of existing Credentials. Under to Domain, click on one of the '(global)' links and then on 'Add Credentials'

You will now need to create an account for Jenkins to access your newly created Git repoes. Under 'Kind' select 'SSH Username with Private Key'. For username use 'git-proj2net'. For 'Private Key', select 'Enter directly' and click on 'Add'. Open up your 'myKey' file and copy and paste the contents. Click on 'OK'. Repeat this process for 'git-proj2net'.

Navigate back to the home screen on Jenkins and click on 'New Item'. Enter in 'proj2-net' for the item name and select 'Pipeline'. Click on 'OK' to create this item. Navigate to 'Build Triggers' and check 'Trigger builds remotely (e.g. from scripts)' and enter in an 'Authentication Token' and note the calling URL.

Navigate down to 'Pipeline'. From Definition, select 'Pipeline script from SCM'. Select 'Git' from the SCM type. The repository URL is the URL of the Git repo clone string. For our example it will be ssh://git-proj2net@192.168.50.30/srv/git-proj2net/proj2net.git. Under credentials, select 'git-proj2net'. Everything else can stay as default and click on 'Save'.

You will need to log into your Git repo and configure the hook to communicate with Jenkins.

```sh
$ sudo -u vi git-proj2net /srv/git-proj2net/proj2net.git/hooks/post-receive
#!/bin/bash
curl -k --user <login>:<pass> https://192.168.50.30/job/proj2-net/build?token=<token>
$ sudo chmod +x  /srv/git-proj2net/proj2net.git/hooks/post-receive
```
We can now test out our Git/Jenkins setup by going into the proj2-net directory and configuring it for pushing to our Git/CICD server.

```sh
$ cd proj2-net
$ git init
$ git remote add origin ssh://git-proj2net@192.168.50.30/srv/git-proj2net/proj2net.git
$ git add .
$ git commit -m 'Initial commit'
$ git push origin master
```
You can watch the progress of the pipeline on Jenkins.

Now we can setup the CICD for the Wordpress installations.

Repeat the above but substite in git-proj2infra/proj2infra where you see proj2net.

```sh
$ cd proj2-infra
$ git init
$ git remote add origin ssh://git-proj2infra@192.168.50.30/srv/git-proj2infra/proj2infra.git
$ git add .
$ git commit -m 'Initial commit'
$ git push origin master
```

Don't forget to copy the Jenkins ssh key to all the servers.
```sh
$ sudo -u jenkins ssh-copy-id ansible@10.1.1.30 
# repeat for 10.1.2.29, 10.1.2.30, 10.1.2.31
```