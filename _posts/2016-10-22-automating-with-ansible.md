---
layout: post
title: Automating with Ansible - Part 1
---

Automating tasks for installing new machines or managing your different environments is a must in the days of
Continous Delivery. Most often I have seen these solutions done by chef or puppet. When I've worked in those projects
many of the tasks/recipes has already been setup and whenever I've gone in and update those I've felt that it's a bit of
 a jungle with a steep learning curve. I was therefore very interested when I got introduced to Ansible by a colleague.

# What is Ansible?
[Ansible](https://www.ansible.com/) is an IT automation tool to orchestrate, provision, deploy or install your infrastructure.
Their goal is to be "simple" as in easy to use, yet very powerful in its features.


As an example, let's say that you have an application that has a lot of prerequired steps to be installed on a fresh environment. In
a delivery pipeline you would have to automate those preqrequired steps so that your pipe would continously setup a new version
 of your application each time. Another example could be that you have a lot of different environments for production and test. Whenever
 you update a configuration on the environments you need to do it on all existing environments, a tidious and boring task. In such
 a situation Ansible could help you with automating the provisioning of the updated configuration.

## How does it work?

Ansible uses a control-machine, a machine were you've installed ansible to run your tasks towards all your target-hosts (e.g. production, test etc).
When Ansible executes, it utilizes SSH towards the client to perform its tasks.


 Ansible requires the control-machine to have python 2.6 or 2.7 installed and is currently not supported on Windows.

# Okey, enough talk, hands-on!

If you want to try it out in a sandboxed environment I've prepared a few docker images. Check out the source code
 at <https://github.com/olbpetersson/ansible-minimal-example>.

  *Note*: You will need to have docker and docker-compose installed.

## Start the system

From the top of the repository, perform <code>docker-compose up</code>. This will spin up a system containg of:
 * An ansible controller - this is were we will perform our provisioning
 * A dummy node - will be the target were we will execute our commands against
 * Another dummy node - same as above


The compose file:

~~~ yaml
version: '2'
services:
  controller:
    build: docker/ansible_controller
    links:
      - node1
      - node2
    tty: true
    volumes:
      - ./work:/home/
  node1:
    build: docker/ansible_target
    tty: true
  node2:
    build: docker/ansible_target
    tty: true
~~~

## Check the connection to your targets

To make sure that everything is correctly setup, execute <code>docker exec -ti ansibleminimalexample_controller_1 bash</code>. Now you should be in bash within the container
of the ansible controller.


 Let's see if we can reach our clients. Execute <code>ansible -m ping all</code>. You should now hopefully see three success responses in
your shell, one from each machine. If we have a closer look at the command, we sent in <code>-m ping</code> to use the predefined ansible module *ping*.
We also told ansible to perform the ping-action towards *all* hosts (see below).


Ansible comes with a lot of predefined modules which can help you with performing tasks at your target machines. To see a selection of them, visit
<http://docs.ansible.com/ansible/modules_by_category.html>.

## Defining our hosts
Ansible is working with an inventory file. This is a configuration file were we define our "machine-groups". A "machine-group"
 could for example be all our databases. We could define these in the group [Databases] with the host names of our databases listed.


  In our example,
 we have defined a_group which consist of the first target node (node1). We also have the group nodes as well as our controller. The controller is defined as a local
 connection since this is the machine from which we will execute our ansible commands.

~~~ yaml
[a_group]
node1

[nodes]
node1
node2

[controller]
controller ansible_connection=local
~~~
*Our inventory file defined in work/dev, referenced from work/ansible.cfg*

If we look back on the *ping*-command we used above, we could now send in the host name as *nodes* (instead of all) to only ping the nodes-group, or *node1* to only ping the node1 machine etc.


**This was the first part** in Ansible were we tried out Ansibles modules and inventory-file to define our system and ping different machines in our system. In an upcoming post we will examine how we can perform more ellaborate tasks using Ansibles Playbook concept.


