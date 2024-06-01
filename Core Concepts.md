1. **Modules**:
   - **Definition**: Predefined tasks that automate system operations and deployments.
   - **Example**: Installing a package using the `apt` module.
     ```yaml
     - name: Install nginx
       ansible.builtin.apt:
         name: nginx
         state: present
     ```

2. **Playbooks**:
   - **Definition**: YAML files that define a series of tasks to be executed on managed hosts.
   - **Example**: A simple playbook to install and start nginx.
     ```yaml
     ---
     - name: Install and start nginx
       hosts: webservers
       tasks:
         - name: Install nginx
           ansible.builtin.apt:
             name: nginx
             state: present

         - name: Start nginx
           ansible.builtin.service:
             name: nginx
             state: started
     ```

3. **Roles**:
   - **Definition**: A method of automatically loading certain variable files, tasks, and handlers based on a known file structure.
   - **Example**: Directory structure for a role and sample task.
     ```yaml
     # Directory structure
     roles/
       common/
         tasks/
           main.yml
         handlers/
           main.yml
         templates/
           ntp.conf.j2
         files/
           foo.sh
         vars/
           main.yml
         defaults/
           main.yml
         meta/
           main.yml

     # Example task in main.yml
     - name: Ensure NTP is installed
       ansible.builtin.apt:
         name: ntp
         state: present
     ```

4. **Variables**:
   - **Definition**: Parameters that can be used to customize tasks and playbooks.
   - **Example**: Defining variables in group_vars and using them in a playbook.
     ```yaml
     # group_vars/all.yml
     nginx_port: 80

     # playbook.yml
     - name: Configure nginx
       hosts: webservers
       tasks:
         - name: Set nginx port
           ansible.builtin.template:
             src: nginx.conf.j2
             dest: /etc/nginx/nginx.conf
     ```

5. **Inventories**:
### Inventory Definition
Files that list the managed hosts and groups of hosts.
#### Static Inventory (INI Format)
This example shows how to define both Windows and Linux hosts in a static inventory file.

```ini
# inventory/hosts

[linux_servers]
linux1 ansible_host=192.168.1.101 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa
linux2 ansible_host=192.168.1.102 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa

[windows_servers]
windows1 ansible_host=192.168.1.201 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore
windows2 ansible_host=192.168.1.202 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

#### Dynamic Inventory (YAML Format)
This example demonstrates a dynamic inventory where hosts can be grouped dynamically based on certain attributes.

```yaml
# inventory/hosts.yml

plugin: yaml
groups:
  linux_servers:
    hosts:
      linux1:
        ansible_host: 192.168.1.101
        ansible_user: your_linux_user
        ansible_ssh_private_key_file: ~/.ssh/id_rsa
      linux2:
        ansible_host: 192.168.1.102
        ansible_user: your_linux_user
        ansible_ssh_private_key_file: ~/.ssh/id_rsa
  windows_servers:
    hosts:
      windows1:
        ansible_host: 192.168.1.201
        ansible_user: Administrator
        ansible_password: your_password
        ansible_connection: winrm
        ansible_winrm_transport: ntlm
        ansible_winrm_server_cert_validation: ignore
      windows2:
        ansible_host: 192.168.1.202
        ansible_user: Administrator
        ansible_password: your_password
        ansible_connection: winrm
        ansible_winrm_transport: ntlm
        ansible_winrm_server_cert_validation: ignore
  all:
    vars:
      ansible_python_interpreter: /usr/bin/python3
```

### Sample Playbook Using Both Linux and Windows

```yaml
---
- name: Configure Linux and Windows servers
  hosts: all
  tasks:
    - name: Ensure Nginx is installed on Linux
      ansible.builtin.yum:
        name: nginx
        state: present
      when: ansible_os_family == "RedHat"

    - name: Ensure IIS is installed on Windows
      win_feature:
        name: Web-Server
        state: present
      when: ansible_os_family == "Windows"
```

### Explanation:

- **Static Inventory**:
  - Defines separate groups for Linux and Windows servers.
  - Uses appropriate connection details for each host type.

- **Dynamic Inventory**:
  - Similar structure to static inventory but defined in YAML format.
  - Uses the `plugin: yaml` directive to enable dynamic grouping.

- **Playbook**:
  - Runs tasks based on the operating system.
  - Installs Nginx on Linux servers and IIS on Windows servers, using conditional checks with `when`.


6. **Handlers**:
   - **Definition**: Special tasks that run when triggered by the `notify` directive.
   - **Example**: Restart nginx when configuration changes.
     ```yaml
     tasks:
       - name: Copy nginx config file
         ansible.builtin.template:
           src: nginx.conf.j2
           dest: /etc/nginx/nginx.conf
         notify:
           - Restart nginx

     handlers:
       - name: Restart nginx
         ansible.builtin.service:
           name: nginx
           state: restarted
     ```

7. **Templates**:
   - **Definition**: Files that combine data from variables and static content to generate dynamic content.
   Jinja2 templates in Ansible are powerful tools for creating dynamic files based on variables and other data. Templates are typically used with the `template` module in Ansible to generate configuration files, scripts, and other text files.

### Example Scenario

Let's consider a scenario where we need to generate an Nginx configuration file for both Linux and Windows servers using Jinja2 templates.

### Inventory

```ini
# inventory/hosts

[linux_servers]
linux1 ansible_host=192.168.1.101 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa
linux2 ansible_host=192.168.1.102 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa

[windows_servers]
windows1 ansible_host=192.168.1.201 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore
windows2 ansible_host=192.168.1.202 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### Playbook

```yaml
---
- name: Configure Nginx on Linux and Windows servers
  hosts: all
  vars:
    nginx_port: 80
  tasks:
    - name: Deploy Nginx configuration on Linux
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      when: ansible_os_family == "RedHat"

    - name: Deploy Nginx configuration on Windows
      template:
        src: nginx.conf.j2
        dest: C:\nginx\conf\nginx.conf
      when: ansible_os_family == "Windows"
```

### Template File (`templates/nginx.conf.j2`)

```jinja2
server {
    listen {{ nginx_port }};
    server_name {{ ansible_hostname }};
    
    location / {
        root {{ nginx_root }};
        index index.html index.htm;
    }
    
    {% if ansible_os_family == 'RedHat' %}
    error_log /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    {% elif ansible_os_family == 'Windows' %}
    error_log C:/nginx/logs/error.log;
    access_log C:/nginx/logs/access.log;
    {% endif %}
}
```

### Explanation

- **Inventory**:
  - Defines both Linux and Windows servers with appropriate connection details.
  
- **Playbook**:
  - Uses the `template` module to deploy the Nginx configuration file.
  - Sets the `nginx_port` variable to be used in the template.
  - Uses a `when` clause to ensure the configuration is applied to the correct OS type.

- **Template (`nginx.conf.j2`)**:
  - Uses Jinja2 syntax to insert variables (`{{ variable }}`) and conditionals (`{% if ... %}`) into the Nginx configuration file.
  - The `listen` directive uses the `nginx_port` variable.
  - The `server_name` directive uses the `ansible_hostname` fact.
  - The `root` directive uses a variable that should be defined elsewhere (e.g., in the playbook or inventory).
  - Conditional blocks set different paths for error and access logs based on the OS family.

### Additional Examples of Jinja2 Templates

#### Template for a Configuration File with a Loop

```jinja2
# templates/app_config.conf.j2

[application]
name = MyApp
version = {{ app_version }}

[servers]
{% for server in servers %}
{{ loop.index }} = {{ server }}
{% endfor %}
```

#### Playbook Using the Above Template

```yaml
---
- name: Generate application config
  hosts: all
  vars:
    app_version: "1.0.0"
    servers:
      - server1.domain.com
      - server2.domain.com
      - server3.domain.com
  tasks:
    - name: Deploy application config
      template:
        src: app_config.conf.j2
        dest: /etc/myapp/app_config.conf
```

In this example, the template loops through a list of servers and generates numbered entries in the configuration file.

By using Jinja2 templates, you can create highly dynamic and flexible configuration files tailored to the specific needs of each host in your inventory.

8. **Tasks**:
   - **Definition**: The basic unit of action in Ansible, defining a single action to be executed.
   - **Example**: Installing a package.
     ```yaml
     - name: Install nginx
       ansible.builtin.apt:
         name: nginx
         state: present
     ```

9. **Blocks**:
   - **Definition**: A group of tasks that can be executed together, often with shared error handling or tags.
   - **Example**: Grouping tasks to install and configure nginx.
     ```yaml
     - name: Ensure nginx is running
       block:
         - name: Install nginx
           ansible.builtin.apt:
             name: nginx
             state: present

         - name: Start nginx
           ansible.builtin.service:
             name: nginx
             state: started
       rescue:
         - name: Install apache as fallback
           ansible.builtin.apt:
             name: apache2
             state: present
     ```

10. **Tags**:
    - **Definition**: Labels that can be used to control which tasks run when executing a playbook.
    - **Example**: Using tags to run specific tasks.
      ```yaml
      - name: Install and configure webserver
        hosts: webservers
        tasks:
          - name: Install nginx
            ansible.builtin.apt:
              name: nginx
              state: present
            tags: nginx

          - name: Install apache
            ansible.builtin.apt:
              name: apache2
              state: present
            tags: apache
      ```

11. **Facts**:
    - **Definition**: Automatic variables gathered by Ansible about the managed hosts.
    - **Example**: Gathering and using facts.
      ```yaml
      - name: Gather facts
        ansible.builtin.setup:

      - name: Print OS information
        debug:
          var: ansible_facts['os_family']
      ```

12. **Lookups**:
    - **Definition**: Mechanisms to pull data from outside of Ansible into playbooks.
    - **Example**: Reading a file into a variable.
      ```yaml
      - name: Read a file into a variable
        ansible.builtin.set_fact:
          file_content: "{{ lookup('file', '/path/to/file.txt') }}"
      ```

13. **Filters**:
    - **Definition**: Tools that modify data and variables within Jinja2 templates.
    - **Example**: Using filters to format data.
      ```yaml
      - name: Example with filters
        debug:
          msg: "{{ some_list | join(', ') }}"
      ```

14. **Loops & with_items**:
Using loops in Ansible allows you to iterate over a list of items and perform tasks for each item. The `with_items` directive is one way to achieve this. Here are examples of using loops with both Linux and Windows servers:

### Example Playbook with `with_items` for Linux

```yaml
---
- name: Install multiple packages on Linux servers
  hosts: linux_servers
  tasks:
    - name: Install required packages
      ansible.builtin.yum:
        name: "{{ item }}"
        state: present
      with_items:
        - nginx
        - git
        - unzip
```

### Example Playbook with `with_items` for Windows

```yaml
---
- name: Install multiple features on Windows servers
  hosts: windows_servers
  tasks:
    - name: Install Windows features
      win_feature:
        name: "{{ item }}"
        state: present
      with_items:
        - Web-Server
        - Web-WebSockets
        - Web-ASP-Net45
```

### Full Inventory and Playbook Example

#### Inventory

```ini
# inventory/hosts

[linux_servers]
linux1 ansible_host=192.168.1.101 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa
linux2 ansible_host=192.168.1.102 ansible_user=your_linux_user ansible_ssh_private_key_file=~/.ssh/id_rsa

[windows_servers]
windows1 ansible_host=192.168.1.201 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore
windows2 ansible_host=192.168.1.202 ansible_user=Administrator ansible_password=your_password ansible_connection=winrm ansible_winrm_transport=ntlm ansible_winrm_server_cert_validation=ignore

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

#### Playbook

```yaml
---
- name: Configure Linux and Windows servers
  hosts: all
  tasks:
    - name: Install required packages on Linux
      ansible.builtin.yum:
        name: "{{ item }}"
        state: present
      with_items:
        - nginx
        - git
        - unzip
      when: ansible_os_family == "RedHat"

    - name: Install Windows features
      win_feature:
        name: "{{ item }}"
        state: present
      with_items:
        - Web-Server
        - Web-WebSockets
        - Web-ASP-Net45
      when: ansible_os_family == "Windows"
```

### Explanation:

- **Inventory**:
  - Defines Linux and Windows servers with necessary connection details.
  
- **Playbook**:
  - Uses `with_items` to iterate over a list of packages (for Linux) and features (for Windows).
  - The `when` clause ensures tasks are executed only on the appropriate type of server (Linux or Windows).

By using `with_items`, you can efficiently manage multiple installations or configurations in a single task, making your Ansible playbooks cleaner and more manageable.
