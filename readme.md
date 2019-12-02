## Pre-requisites
Install VirtualBox

```brew cask install virtualbox```

Install Vagrant

```brew cask install vagrant```

Install Ansible

Either use: ```pip install ansible``` or ```brew install ansible```

## How to run the solution

In order to run the solution a single command of `vagrant up` is used.

Once the solution is up, you can visit:

http://10.99.25.10 for the main loadbalancer page - if you refresh you will see the line *Currently being served by backend-X* updates where X is the instance id.

http://10.99.25.10/test.html for the the test page - if you refresh you will see the line *Currently being served by backend-X* updates where X is the instance id.

---

## Process and Solution

### Overview
The solution leverages Vagrant for the deployment of servers and Ansible for provisioning the application.

### Implementation
The solution utilises a role per type of server being built (nignx / nginx_proxy). There is potential to have a shared role for items that are shared between the instances which if this was more than a PoC you may well want to look at.

A few different methods were investigated during the implementation for the testing phase as this is where the bulk of the time spent on the solution was spent. In the end, the Ansible URI method proved to be most useful to get to the header content that is reported back during the playbook. Other methods looked at involved extra script files or an inline `shell` step. Overall the URI method seemed easiest as it's a standard Ansible method.

### Testing
In the config for the nginx backends an extra header is added named `X-Who` which is set to the hostname of the server returning the page.

The testing takes place during the Ansible run by making use of the URI functionality to get the page content including headers and then just return the value of the `X-Who` header.

This test of returning the header happens in in both roles:
1. During the deployment of the backends to ensure that they've been deployed successfully.
2. During the deployment of the loadbalancer two calls are made to the loadbalancer itself to return `X-Who` headers from the backend nginx servers.

### Testing Output
The relevant output to look for while `vagrant up` is running looks like the below:

For the deployment of the backends:
```
TASK [nginx : get test page] ***************************************************
ok: [backend_1]

TASK [nginx : debug] ***********************************************************
ok: [backend_1] => {
    "page.x_who": "backend-1"
}
```
```
TASK [nginx : get test page] ***************************************************
ok: [backend_2]

TASK [nginx : debug] ***********************************************************
ok: [backend_2] => {
    "page.x_who": "backend-2"
}
```

For the deployment of the loadbalancer:
```
TASK [nginx_proxy : get test page_1] *******************************************
ok: [loadbalancer]

TASK [nginx_proxy : debug] *****************************************************
ok: [loadbalancer] => {
    "page_1.x_who": "backend-1"
}

TASK [nginx_proxy : get test page_2] *******************************************
ok: [loadbalancer]

TASK [nginx_proxy : debug] *****************************************************
ok: [loadbalancer] => {
    "page_2.x_who": "backend-2"
}
```

### Potential Improvments
1. Implement nginx configuration checking

   While the configuration files used here are simple, as they get more complext it would make sense to implement a step to do an nginx configuration check by utilising `nginx -t` in a step.

2. Allow for more than one loadbalancer
   
   The solution has only one loadbalancer built as per the requirement however in a more complex setup it would make sense to have potentially two or more loadbalancers with configuration syncing occuring. 