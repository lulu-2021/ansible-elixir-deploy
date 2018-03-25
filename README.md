# Netflakes Ansible Elixir

This project has ansible playbooks for:

* Elixr Build Server - it has roles for Erlang, Elixir and Nodejs, Bower and Postgres. Basically, all the dependencies you
will need to deploy an Elixir Phoenix App
* Present_me is just the name of an app I was building - use this as a sample app.. rename at your leisure..

## Requirements

### Control machine (your computer)

* Install [Ansible](https://www.ansible.com/)

* Download roles:

  ```shell
  $ ansible-galaxy install -r requirements.yml
  ```

* Generate `vault_pass.txt` file into this repository. You need it to be able to encrypt/decrypt secrets.

  :warning: For security reasons, the `vault_pass.txt` file should not be committed into the repository. It's ignored in `.gitignore`.

  ```shell
  $ openssl rand -base64 256 > vault_pass.txt
  ```

* Generate your DB password and put output to `apps/phoenix-website/host_vars/sample.appserver...`

  ```shell
  $ ansible-vault encrypt_string --name db_password "YOUR_DB_PASSWORD"
  ```

### Target machine (server)

* Server with [Ubuntu](https://www.ubuntu.com/) 16.04 LTS.

## Public keys

My public key is in the [public_keys/] directory. This set of keys is uploaded to the server during each provisioning
and overwrites the list of authorized keys, so proper people have access to the server. It is important to keep
the list of keys up to date. NB - generate your own and remove mine!!

## App deployement

## Run playbooks

NB: I am using the name `sample_app_name` here - naturally you will need to rename the files under `host_vars` and in the relevant `inventory` for this to work for you. AFTER you have `cloned` this repo.

This command will provision all servers listed in inventory file for particular app `apps/sample_app_name`.

```shell
$ ./play apps/sample_app_name
```

If you want to provision only specific machine do (it's useful if your app is deployed to multiple servers like staging and production):

```shell
# Warning: There must be comma and the end of the hosts list!
$ ansible-playbook -i 'example-staging.example.com,' apps/sample_app_name/playbook.yml
```

## Provisioning logs

You can check when and with what git commit the host was provisioned in log file: `/var/log/provision.log` (stored on the target machine).

## System users

There are different types of users on the server:

* `root` - for provisioning
* `deployer` - user has the sudo access

## Add playbook for new app

* create new app directory in the `app` directory
* in this new directory create `playbook.yml` and `inventory` files
* in the `inventory` file put host names to provision (see [Ansible docs](http://docs.ansible.com/ansible/intro_inventory.html))
* implement `playbook.yml`

## Secrets

We store secrets in encrypted version using [Vault]. If you are adding new secrets, make sure you commit them to the repository in the encrypted form.

* Encrypting single values (that can be placed inside a "clear text" YAML file, using the `!vault` tag):

  ```shell
  $ ansible-vault encrypt_string --name pass_to_some_service "secret"  # stdout encrypted string
  ```

* Encrypting whole YAML files:

  ```shell
  $ ansible-vault encrypt secret.yml   # encrypt unencrypted file
  $ ansible-vault edit secret.yml      # edit encrypted file
  $ ansible-vault decrypt secret.yml   # decrypt encrypted file
  ```

## Roles

### Role versioning

We use roles versioning the simplest possible way, we just add version subdirectories under every role directory.

```shell
roles/role-name/role-version/ # e.g. roles/webserver/0.3.2/
```

To create a new version just copy an existing one, bump the role version and modify it.
Please, respect [Semantic Versioning 2.0.0].

