# Setup

This was created from the ansible tutorial [here](https://www.udemy.com/course/mastering-ansible)

There are many ways to setup your cluster for this to work. In my case I created LXC containers on my homelab. This is all run from my control machine, a linux laptop.

I have created a handful of hosts. The apps could use hostname, but the IP's are currently needed for nginx because i haven't figured out the best way to template the hosts file.

### Hosts

- lb1 - nginx load balancer
- app01 - HA apache running WSGI/flask
- app02 - HA apache running WSGI/flask
- db01 - mysql

# Creating a vault password

```
echo "some secret" > ~/.vault_pass.txt
chmod 0600 ~/.vault_pass.txt
```

# Running the playbooks

```
ansible-playbook site.yml
```
