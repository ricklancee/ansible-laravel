# Ansible docker image to provision servers for laravel

*NOTE: be careful while provisioning servers ;)*

This runs ansible inside a docker container so that you can provision
your servers easily. You only need docker, a server to ssh into
and a domain name (required for ssl certifcates). The setup here is loosly based on the excellent tutorials on [serversforhackers.com](https://serversforhackers.com/)

This setup is for ubuntu servers; specifically ubuntu 16.04.

Includes:
- Nginx configured with https and http2
- SSL certificates via letsencrypt
- Uncomplicated Firewall Setup (allowed ports are 80, 443 and 22)
- Automatic server security updates (runs daily)
- Laravel requirements (php7.1, composer)
- MySQL server
- Supervisor for laravel queues (this will need to be reread and updated when laravel is installed)
- an admin and deploy user
- Todo: Redis

### Usage

#### Server setup
1. Get an ubuntu server (digitalocean or AWS etc).
2. Install python 2.7 on the server (ansible requires servers to have python installed)

#### Ansible setup (requires docker)
1. Build the docker image with `docker build --rm -f Dockerfile.ansible -t ansible:dockerfile .` this will build ansible inside a docker image.
2. Copy the public ssh key that ansible generates (output in terminal) and copy it to your server so that ansible can access it. (you will need to copy a new ssh key if you rebuild the image)

#### Provision servers
1. Define your server ip in the `./ansible/hosts` file.
2. Set the variables in `./ansible/servers.yml`; you will need a real domain for the SSL cert to work
3. Set a admin and deploy user password; and their public key in `./ansible/users/vars/main.yml` and encrypt it with `ansible-vault`
4. Set a mysql root password in `./ansible/users/vars/main.yml` and encrypt it with `ansible-vault`

Run ansible commands:

```sh
docker run -v $(pwd)/ansible:/etc/ansible -it --rm ansible:dockerfile ansible
docker run -v $(pwd)/ansible:/etc/ansible -it --rm ansible:dockerfile ansible-playbook
docker run -v $(pwd)/ansible:/etc/ansible -it --rm ansible:dockerfile ansible-vault
```

The argument `-v $(pwd)/ansible:/etc/ansible` will mount the `./ansible` directory to the image `/etc/ansible` directory. This is so that you can make changes to the ansible directory without needing to rebuild the image.

 If you've encrypted the user and mysql role `vars/main.yml` files (which you should do) you will need store the password in a vault password file `.vaultpass` and use the argument `--vault-password=".vaultpass"`, or let ansible ask for the password by ommiting this.

Provisioning the servers can be done with the following command, this will provisions all servers defined in `ansible/hosts`.

```sh
docker run -v $(pwd)/ansible:/etc/ansible -it --rm ansible:dockerfile ansible-playbook --vault-password=".vaultpass" servers.yml
```
