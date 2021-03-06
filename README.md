yogesh174.ansible_role_provisioning
=========

This role provisions EC2 instances on AWS cloud meant for kubernetes HA cluster.

Requirements
------------

Make sure to install and configure aws shell with your credentials. Setup dynamic inventory in ansible with aws EC2 instances. Also make sure that the ansible control node can connect to newly provisioned EC2 instances(using group_vars) with the keypair you provide.

Role Variables
--------------

Compulsory variables in vars/main.yml:

region
vpc_subnet_id

Optional variables and their default values in vars/main.yml:

worker_count: 2
keypair: "k8s"
instance_type: "t2.micro"
image: "ami-047a51fa27710816e"
assign_public_ip: "yes"
sg: "k8s"

Example Playbook
----------------

"provision_master" task provisions master node with the necessary tags and "provision_worker" task provisions worker nodes with the necessary tags.

    - hosts: localhost
          
      tasks:

        - name: "Provision master node"
          import_role:
            name: provisioning
            tasks_from: provision_master
          vars:
            vpc_subnet_id: "subnet-2b3b2b4c"
            region: "us-east-1"
        
        - name: "Provision worker nodes"
          import_role:
            name: provisioning
            tasks_from: provision_worker
          vars:
            vpc_subnet_id: "subnet-2b3b2b4c"
            region: "us-east-1"

        - set_fact:
            ec2_instances: "{{ ec2_master.instances + ec2_master.tagged_instances + ec2_worker.instances + ec2_worker.tagged_instances }}"

        - name: wait for ssh to start
          wait_for:
            host: "{{ item.public_dns_name  }}"
            port: 22
            search_regex: OpenSSH
            delay: 20
            state: started
          loop: "{{ ec2_instances }}"

      
License
-------

BSD

Author Information
------------------

Surapaneni Yogesh - surapaneniyogesh11@gmail.com  
Connect with me on [LinkedIN](https://www.linkedin.com/in/surapaneni-yogesh-ba7303189/)
