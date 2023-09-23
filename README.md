# project-12
## ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)
#### Step 1 – Jenkins job enhancement
Before we begin, let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.
- Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.
-sudo mkdir /home/ubuntu/ansible-config-artifact
- Change permissions to this directory, so Jenkins could save files there <chmod -R 0777 /home/ubuntu/ansible-config-artifact>
- Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins
- Create a new Freestyle project and name it save_artifacts.
- This project will be triggered by completion of your existing ansible project. Configure it accordingly
#### NOTE:
You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.
The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-config-artifact as a target directory.
Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).
![](https://github.com/BigTesty8/project-12/assets/137091610/2e0acdcb-89b8-4e74-8936-0ae9dd41dff0)
If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-config-artifact directory and it will be updated with every commit to your master branch.
![](https://github.com/BigTesty8/project-12/assets/137091610/5498528d-c728-4750-a834-f21b0b911877)
![](https://github.com/BigTesty8/project-12/assets/137091610/3ecad6a6-f7a8-4d1c-a21a-1102f3133de4)

## REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML
#### Step 2 – Refactor Ansible code by importing other playbooks into site.yml
Before starting to refactor the codes, ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.
Let see code re-use in action by importing other playbooks.
- Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously. Dont worry, you will understand more what this means shortly.
- Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon. For now, just follow along.
- Move common.yml file into the newly created static-assignments folder.
- Inside site.yml file, import common.yml playbook.
![](https://github.com/BigTesty8/project-12/assets/137091610/bb467f42-1f10-418f-918a-ffe409144344)
The code above uses built in import_playbook Ansible module.
- Run ansible-playbook command against the dev environment
Since you need to apply some tasks to your dev servers and wireshark is already installed – you can go ahead and create another playbook under static-assignments and name it common-del.yml. In this playbook, configure deletion of wireshark utility.
- update site.yml with - import_playbook: ../static-assignments/common-del.yml instead of common.yml and run it against dev servers:
![](https://github.com/BigTesty8/project-12/assets/137091610/1d806f5d-bd45-426c-b491-765bb3546b40)
![](https://github.com/BigTesty8/project-12/assets/137091610/453d4914-5b3f-475b-9ae7-1783a5277ebc)
## CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’
#### Step 3 – Configure UAT Webservers with a role ‘Webserver’
We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.
- Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.
To create a role, you must create a directory called roles/, relative to the playbook file or in /etc/ansible/ directory.
There are two ways how you can create this folder structure:
- The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy
![](https://github.com/BigTesty8/project-12/assets/137091610/bfed28ea-b6a4-4c70-8401-8cde3925eac9)
- Update your inventory ansible-config-mgt/inventory/uat.yml file with IP addresses of your 2 UAT Web servers
In /etc/ansible/ansible.cfg file uncomment roles_path string and provide a full path to your roles directory roles_path    = /home/ubuntu/ansible-config-mgt/roles, so Ansible could know where to find configured roles.

![](https://github.com/BigTesty8/project-12/assets/137091610/19495da0-a875-4609-921c-190c509c22f8)

roles_path    = /home/ubuntu/ansible-config-mgt/roles was later corrected to roles_path    '= /home/ubuntu/ansible-config-artifacts/roles'
It is time to start adding some logic to the webserver role. Go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

- Install and configure Apache (httpd service)
- Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
Make sure httpd service is started.
Your main.yml may consist of following tasks:
![](https://github.com/BigTesty8/project-12/assets/137091610/a02bd343-b295-42bd-8ced-a375d391e3a9)
## REFERENCE WEBSERVER ROLE
#### Step 4 – Reference ‘Webserver’ role
Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.
![](https://github.com/BigTesty8/project-12/assets/137091610/1cb1ece9-b516-4e42-8d5d-95577ea6ca71)
Remember that the entry point to our ansible configuration is the site.yml file. Therefore, you need to refer your uat-webservers.yml role inside site.yml.
So, we should have this in site.yml
![](https://github.com/BigTesty8/project-12/assets/137091610/6f36e25a-c2f4-4391-9863-9b9ccf6284c9)
### Step 5 – Commit & Test
Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into /home/ubuntu/ansible-config-mgt/ directory.

![](https://github.com/BigTesty8/project-12/assets/137091610/bcd62c8c-78f3-4ecd-bda2-76c9fe5912cd)

Now run the playbook against your uat inventory and see what happens:

![](https://github.com/BigTesty8/project-12/assets/137091610/f39c94dc-3ff2-4053-8a99-5b75800ae55d)


You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
or
http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php
![](https://github.com/BigTesty8/project-12/assets/137091610/f5a1bd04-d96a-4529-9c3f-cadbfc8d635d)










  
