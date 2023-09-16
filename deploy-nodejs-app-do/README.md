# Automating Node.js Application Deployment with Ansible

Automates the deployment of a [simple Node.js application](https://github.com/flaviassantos/simple-nodejs-app) on a Digital Ocean server using Ansible. The process will involve creating a Digital Ocean droplet, installing Node.js and NPM, copying and unpacking the Node.js application on the server, and starting the application.

Modules used:
- [ansible.builtin.apt module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
- [ansible.builtin.copy module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [ansible.builtin.unarchive module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/unarchive_module.html)
- [Asynchronous actions and polling](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_async.html)
### Setting Up the Environment

1. **Create a Digital Ocean Droplet with Terraform**
   - source code [here](https://github.com/flaviassantos/terraform-projects/tree/master/simple-digital-ocean-droplet).

2. **Ansible Inventory**:
   - In the Ansible inventory file (hosts), add the IP address of the new droplet.
   - Configure it with an SSH key so Ansible can access it with a root user.

### Ansible Playbook: `deploy-nodejs-app.yaml`

#### Play 1: Install Node.js and NPM
- **Name**: `install_node_and_npm`
- **Hosts**: The IP address of the droplet.
- **Tasks**:
   - Update the APT package manager repository and cache using the `apt` module.
   - Install Node.js and NPM using the `apt` module.
   - The playbook ensures that the APT cache is updated only if needed and installs the required packages.

#### Play 2: Deploy Node.js Application
- **Name**: `deploy_nodejs_app`
- **Hosts**: The IP address of the droplet.
- **Tasks**:
  **Unpack Node.js Application**:
      - Use the `unarchive` module to unpack the tar file from your local machine to the remote server (clone the app [here](https://github.com/flaviassantos/simple-nodejs-app)).
  - **Install Dependencies**:
  **Module**: NPM
    **Path**: The path to the `package.json` file on the remote server.
    - The task gets executed on the remote server.
    - This task uses the NPM module to install the dependencies listed in the `package.json` file. It ensures that the `node_modules` folder is created.

    
    Starting the Node.js Application: with the dependencies installed, we can proceed to start the Node.js application. This involves executing the `node server.js` command on the remote server.

- **Task: Start Node.js Application**

- **Module**: Command

- **Command**: `node /path/to/server.js`
    - The task gets executed on the remote server.
    - This task executes the `node` command with the path to the `server.js` file, effectively starting the Node.js application.

    
    Asynchronous Execution: one challenge with starting the Node.js application is that the `node server.js` command typically blocks the terminal. To allow Ansible to proceed without waiting for the command to finish, we use asynchronous execution. By setting `async` and `poll`, the `node` command is executed asynchronously, allowing Ansible to continue with the playbook without waiting for the command to finish.

- **Task: Ensure Application is Running**
  
- **Module**: Shell

- **Command**: `ps aux | grep node`
    - This task uses the Shell module to execute the `ps aux | grep node` command, which checks if the Node.js application is running.


    Registering Variables: to capture the output of the `ps aux | grep node` command and display it in the playbook results, we use the `register` attribute.

- **Attribute**: `register`
- **Variable Name**: `app_status`
- The `register` attribute captures the output of the Shell module command in the `app_status` variable.


    Displaying Results: to display the results of the `app_status` variable, we use the Debug module.

#### Task: Display Application Status
- **Module**: Debug
- **Message**: `stdout_lines: "{{ app_status.stdout_lines }}"`
- This task uses the Debug module to display the standard output lines from the `app_status` variable.

In this part of the lecture, we are enhancing the security of our Node.js application deployment by creating a new Linux user to run the application, instead of using the root user. This is an important security best practice to minimize the potential damage caused by a security breach.

Here are the key points from this part of the lecture:

### Security Best Practice: Using Non-Root Users

- Running applications as the root user is not recommended due to the extensive privileges and permissions granted to the root user.
- It's a security best practice to create dedicated users for each application or team member.
- Each application should run under its own user account with limited permissions.

### Play: Create New Linux User for Node App

- **Hosts**: The same remote server.
- **Tasks**: Create a new Linux user using the `user` module.
- **Attributes**:
  - `name`: The name of the new user (e.g., "nana").
  - `description`: An optional description for the user (e.g., "nana admin").
  - `group`: The group to which the user should belong (e.g., "admin").

#### Changing Ownership and Execution Permissions

After creating the new Linux user, we need to ensure that the Node.js application files are owned by this user, and that the user has the necessary permissions to execute the application.
- We use the `become` and `become_user` attributes to switch to the new user context when executing tasks related to the Node.js application.


### Executing the Playbook

- Run the Ansible playbook using the command:
  ```bash
  ansible-playbook deploy-nodejs-app.yaml
