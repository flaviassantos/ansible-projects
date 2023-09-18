# Installing Docker with Ansible on AWS EC2

Automated Ansible playbook to install Docker and Docker Compose on an AWS EC2 server.

## Steps:

1. **Creating the EC2 Instance**

    - Terraform script to automate instance creation.
    - Ensure that you have the correct variables set in the Terraform script.
    - The script creates a new key pair from your local public key for SSH access.

2. **Configuring Ansible to Connect**

    - Copy the public IP address of the EC2 server.
    - In the Ansible project's hosts file, create a new entry (e.g., `docker_server`) and paste the IP address.
    - Use your private key for authentication.
    - Connect as the EC2 user (typically `ec2-user`).

3. **Creating Ansible Playbooks**

    - Create an Ansible playbook (e.g., `deploy_docker.yml`).
    - Start with a play to install Docker on the EC2 instance:
    
    ```yaml
    - name: Install Docker
      hosts: docker_server
      tasks:
        - name: Install Docker
          yum:
            name: docker
            state: present
          become: yes
    ```

    - We use the `yum` module for package management instead of `apt`.
    - We switch to the root user (become: yes) to install packages.

4. **Installing Python 3 on EC2 Instance**

    - Python 3 is not the default on EC2, so we install it as a separate task:
    
    ```yaml
    - name: Install Python 3
      hosts: docker_server
      tasks:
        - name: Install Python 3
          yum:
            name: python3
            state: present
          become: yes
    ```

5. **Configuring Python Interpreter**

    - We configure Ansible to use Python 3 interpreter globally:
    
    ```yaml
    ansible_python_interpreter: /usr/bin/python3
    ```

    - This ensures Ansible uses Python 3 for tasks.

6. **Handling Yum Module and Python Versions**

    - Due to Yum module limitations, we must specify Python 2 for Yum tasks. We can set the interpreter at the task level:
    
    ```yaml
    vars:
      ansible_python_interpreter: /usr/bin/python3

    tasks:
      - name: Install Python 2 and Docker
        yum:
          name: 
            - python2
            - docker
          state: present
        become: yes
        become_method: sudo
        become_user: root
        vars:
          ansible_python_interpreter: /usr/bin/python2
    ```

    - This approach allows using Python 3 for other tasks while using Python 2 for Yum modules.

7. **Installing Docker Compose**

    - We install Docker Compose using a `get_url` module:
    
    ```yaml
    - name: Install Docker Compose
      hosts: docker_server
      tasks:
        - name: Download Docker Compose
          get_url:
            url: "https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64"
            dest: /usr/local/bin/docker-compose
            mode: 'a+x'
          become: yes
    ```

    - We use the `get_url` module to download the Docker Compose binary, set execute permissions, and place it in the desired directory.

8. **Starting Docker Daemon**

    - We start the Docker daemon with a systemd module:
    
    ```yaml
    - name: Start Docker Daemon
      hosts: docker_server
      tasks:
        - name: Start Docker
          systemd:
            name: docker
            state: started
    ```

    - Ensure Docker is running.

9. **Adding EC2 User to Docker Group**

    - To allow the EC2 user to execute Docker commands without `sudo`, we add the user to the Docker group:
    
    ```yaml
    - name: Add EC2 User to Docker Group
      hosts: docker_server
      tasks:
        - name: Add EC2 User to Docker Group
          user:
            name: ec2-user
            groups: docker
            append: yes
    ```

    - We use the `user` module to add the `ec2-user` to the `docker` group.

10. **Testing Docker Commands**

    - To test whether Docker commands work without `sudo`, create a test playbook:
    
    ```yaml
    - name: Test Docker Pull
      hosts: docker_server
      tasks:
        - name: Test Docker Pull
          command: docker pull redis
    ```

    - Remember to reset the SSH session using the `meta` module:
    
    ```yaml
    - name: Reconnect to Server Session
      hosts: docker_server
      tasks:
        - name: Reset SSH Connection
          meta: reset_connection
    ```

    - This ensures that changes take effect immediately.
