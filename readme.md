Create vm and install ansible tower using vagrant

# Variables need to be changed

You need to change these variables in `Vagrantfile`.

```config
$hostname = "tower-3-2-3"
$private_ip = "192.168.33.40"
$tower_bundle_download_url = "https://releases.ansible.com/ansible-tower/setup-bundle/ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"
$tower_bundle_install_file = "ansible-tower-setup-bundle-3.2.3-1.el7.tar.gz"
```

# Prepare tower lisence key

Goto [https://www.ansible.com/license](https://www.ansible.com/license) to get a free trial license file.

After you received license file by email, rename it to `lisence.txt` and copy it to same directory of this repository.

> If you want to try Workflow features, choose "FREE ANSIBLE TOWER TRIAL - ENTERPRISE FEATURES". [Here](https://www.ansible.com/products/tower/editions) is detailed comparation of Tower editions.


# How to run

Just use `vagrant up` to create a new guest vm. During vagrant provisioning, ansible tower will be installed.

It will take 15 - 30 min to install. If you saw following message said "The setup process completed successfully", means tower is ready.
```console
default: PLAY RECAP *********************************************************************
default: localhost                  : ok=131  changed=62   unreachable=0    failed=0
default: The setup process completed successfully.
default: Setup log saved to /var/log/tower/setup-xxxxxx.log
```

Use your browser to access https://{above_private_ip}
  - USER: admin
  - PASS: password

Use `vagrant halt` to shutdown this vm, or `vagrant destroy` to delete this vm.

# Enable remote access to postgresql

You can ssh into guest vm as user vagrant (`vagrant ssh`), then run command below.
```sh
# Enable remote access
echo "host  all  all  0.0.0.0/0  md5" | sudo tee -a /var/lib/pgsql/9.6/data/pg_hba.conf

# Restart service
sudo systemctl restart postgresql-9.6
```

Now, you can access awx db as from host os.
```sh
psql -h $private_ip -W awx awx
# Password is password
```
