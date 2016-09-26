# SixArm.com » <br> Ansible examples of configuration files.

Contents:

* [Start](#start)
* [Host file](#host)
* [Ansible playbooks and tasks](#ansble-playbooks-and-tasks)
* [Output](#output)
* [Privilege escalation](#privilege-escalation)


<h2><a name="start">Start</a></h2>

Verify Ansible runs and its version is at least 2.1:

    $ ansible --version
    2.1.0.0

Run a simple command:

    $ ansible all -i localhost, -c local -m shell -a 'echo hello world'
    localhost | SUCCESS | rc=0 >>
    hello world

Clone this repo and go into it:

    $ git clone https://github.com/sixarm/sixarm_ansible_examples
    $ cd //github.com/sixarm/sixarm_ansible_examples


<h2><a name="ansible-playbooks-and-tasks">Ansible playbooks and tasks</a></h2>

This repository is oriented toward playbooks and tasks.

See the `playbooks` directory and `tasks` directory.

One playbook corresponds to one task. This is because Ansible currently can run one playbook, but cannot run one task without a playbook.

Run a simple playbook:

    $ ansible-playbook -i localhost, -c local playbooks/hello-world.yml

    PLAY [Hello World] ***********************************************************

    TASK [setup] *******************************************************************
    ok: [localhost]

    TASK [include] *****************************************************************
    included: tasks/hello-world.yml for localhost

    TASK [Hello World] ***********************************************************
    ok: [localhost] => {
      "msg": "Hello World"
    }

    PLAY RECAP *********************************************************************
    localhost                  : ok=3    changed=0    unreachable=0    failed=0


<h2><a name="host">Host file</a></h2> 

We typically use a host file, even when we are simply doing command line one-off scripts, because we like to have the flexibility of using multiple hostnames and multiple settings.

Create a `host` file with this text:

    localhost

Now you can use the host file like this:

    $ ansible-playbook -i host -c local playbooks/hello-world.yml


<h2><a name="out">Output</a></h2> 

Ansible does not print the result of commands. Instead, an Ansible script can register a variable that receives the output, and then a subsequent task can print the output by using the debug module.

Example task:

    - name: demo
      command: which python
      register: x
    - debug: var=x.stdout_lines

Run a playbook to print the current python interpreter path:

    $ ansible-playbook -i host -c local playbooks/which-python.yml
    …
    ok: [localhost] => {
        "x.stdout_lines": [
            "/usr/local/bin/python"
        ]
    }


<h2><a name="privilege-escalation">Privilege escalation</a></h2> 

Ansible documentation strongly recommends against setting any sudo password in plaintext, and instead using the command line option `--ask-become-pass` abbreviated `-K`. This option used to be called `--ask-sudo-pass`.

Run a playbook that uses the "become" setting, which does privilege escalation akin to the `sudo` command:

    $ ansible-playbook -K -i host -c local playbooks/demo-become.yml
    SUDO password:
    …

Another way to accomplish privilege escalation is to create a user on the target machine, then grant the user passwordless sudo privileges to either all commands or a restricted list of commands.

Run `sudo visudo` and enter a line like the below:

    %alice ALL= NOPASSWD: /usr/bin/service

The line means the user 'alice' does not enter a password when they run something like `sudo service xxxx start`.

Another way to accomplish privilege escalation is to create an Ansible vault: 

    ansible-vault create myvault

For more info see http://stackoverflow.com/questions/21870083/specify-sudo-password-for-ansible


<h2><a name="boto">Boto 3 - The AWS SDK for Python</a></h2> 

Some Ansible modules use the python 2.x language and the Boto Amazon Web Services (AWS) Software Development Kit (SDK).

Boto documentation: http://boto3.readthedocs.io/en/latest/

To install boto typically:

    $ pip install boto

If you get this kind of error:

    fatal: [localhost]:
    FAILED! => {
      "changed": false,
      "failed": true,
      "msg": "boto required for this module, install via pip or your package manager"
    }

Then you need to tell Ansible how to find the python interpreter and the boto package.

To tell Ansible how to find the python interpreter, we prefer using the `host` file like this:

    localhost ansible_python_interpreter="/usr/bin/env python"


Sidenote: to install boto3 globally to your system using python3, one way is to do is:

    $ sudo su -
    $ cd /opt
    $ git clone https://github.com/boto/boto3
    $ cd boto3
    $ python3 setup.py install


