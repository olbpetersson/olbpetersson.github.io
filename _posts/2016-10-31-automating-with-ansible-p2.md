---
layout: post
title: Automating with Ansible - Part 2
---

In the [first part](http://olapetersson.se/blog/automating-with-ansible) of Automating with Ansible we had a look at how you can utilize Ansibles inventory-file and the ping module to communicate 
with the different machines in your environment. This time we'll have a look of how to utilize Ansibles Playbooks to leave the command line and automate our IT even further

# Playbooks
Ansible utilizes the concept of playbooks to organize and execute tasks for us. A playbook is a simple yml-file with tasks (instructions). Let's have a look at *work/playbooks/touchfile.yml* in the [repository](https://github.com/olbpetersson/ansible-minimal-example):

~~~ yaml
---
  - hosts: all
    tasks:
      - command: touch /tmp/removeMe.file
~~~

The first thing we do in our playbook is to define which hosts the playbook should target. As seen above, we aim to execute this playbook on all of our hosts.


On the next line we define our first task. The task consists of the command-module. The command-module is, just like the ping, one of the standard modules that Ansible ships with. The difference is that command executes a command on the target host instead of just sending a ping. We define the command to perform <code>touch /tmp/removeMe.file</code>. 

Execute the playbook with 
<code>ansible-playbook playbooks/touchfile.yml</code>. Hopefully you'll see the output below

~~~ yml
PLAY [all] *********************************************************************

TASK [setup] *******************************************************************
ok: [controller]
ok: [node1]
ok: [node2]

TASK [command] *****************************************************************
changed: [node2]
 [WARNING]: Consider using file module with state=touch rather than running touch

changed: [node1]
changed: [controller]

PLAY RECAP *********************************************************************
controller                 : ok=2    changed=1    unreachable=0    failed=0
node1                      : ok=2    changed=1    unreachable=0    failed=0
node2                      : ok=2    changed=1    unreachable=0    failed=0
~~~


As you can see, Ansible managed to execute the command on all of our nodes. If you look in the */tmp* directory on any of your nodes, you should now see a *removeMe.file*. However, Ansible doesn't seem quite happy as we can see in the "*[WARNING] Consider using file module with state=touch rather than running touch*". 

## Modules (continued)
As you might realize, executing touch to create files isn't recommended. Instead, we should utilize the [file module](http://docs.ansible.com/ansible/file_module.html). The file module is a building block which Ansible ships to take care of our file operations. Let's create a playbook using the file-module to remove the file we just created:

~~~ yml
---
  - hosts: all
    tasks:
      - name: Remove removeMe file from /tmp
        file: path=/tmp/removeMe.file state=absent
~~~
*work/playbooks/removefile.yml*

First of, we have a new attribute under our task: *name*. This is something we can add to get clearer output when we run Ansible. However, the interesting part comes at the 4th line: file: <code>path=/tmp/removeMe.file state=absent</code>. Instead of using the command line to remove our file, we use the file module and its API. By giving the *state=absent*, we are telling Ansible to "make the file absent", i.e. deleting it. Other choices that the modules exposes for the state parameter are *file*,*link*,*directory*,*hard* and *touch* (which we should have used in the first example).


Running the playbook (<code>ansible-playbook playbooks/removefile.yml</code>), you should now get a similar output as we did in the first playbook. However, take notice in the changed-attribute, it should display <code>changed=1</code>. If you run the command once again, you should now see this status set to <code>changed=0</code>. This is Ansibles way to communicate wether or not it affected the system in any way. Since we could not remove an already removed file, it displayed a 0.

## Dry runs

Sometimes it's desirable to see what tasks a playbook would run without actually executing it. Ansible allows dry runs by providing the *--check* argument. Make sure that the removeMe.file is gone from */tmp/* and try out the touchfile once more with the check-flag: <code>ansible-playbook playbooks/touchfile.yml --check</code>. Even though Ansible displayed the run dialog, if you look in /tmp/ the file wasn't created.


## Handlers
The last concept of this post will be handlers. Handlers are exactly as tasks with the only difference that they are executed *by* tasks. We can do this by 'notifying' a list of handlers from or task. Let's make a little update to our remove-file-playbook

~~~ yml
---
  - hosts: all
    tasks:
      - name: Rename removeMe.file to moved.file
        command: mv /tmp/removeMe.file /tmp/moved.file
        notify:
          - Remove moved.file from /tmp

    handlers:
      - name: Remove moved.file from /tmp
        file: path=/tmp/moved.file state=absent

~~~

As seen above, we moved the removal part down to a handlers-section. In our task, we now move the file with the command-module and then notify our handler to perform the deletion on our now moved file.




**This was a** brief look at Ansibles playbooks and how we can use them to automate tasks across several machines