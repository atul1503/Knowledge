Notes:

1. **Ansible is like a remote control for your servers.** You write instructions once, and it runs them on multiple machines automatically. No need to SSH into each server and run commands manually.

2. **Inventory file tells Ansible which servers to manage.** It's like your contact list of servers. Create a file called `inventory.ini` or `hosts.ini`:
   ```ini
   [webservers]                    # Group name (you choose this)
   web1.example.com               # Server hostname or IP address
   web2.example.com
   192.168.1.10                   # You can use IP addresses too

   [databases]                    # Another group
   db1.example.com ansible_user=admin    # ansible_user specifies which user to connect as
   db2.example.com ansible_port=2222     # ansible_port specifies SSH port if not 22

   [production:children]          # Group of groups (contains other groups)
   webservers                     # Include the webservers group
   databases                      # Include the databases group
   ```

3. **Playbooks are your instruction manuals.** They contain a series of tasks to run on your servers. Create a file ending in `.yml`:
   ```yaml
   ---                            # YAML files start with three dashes
   - name: Install and start nginx          # Human-readable description of what this playbook does
     hosts: webservers                      # Which group from inventory to run on
     become: yes                            # Run commands as root (like sudo)
     
     tasks:                                 # List of tasks to execute
       - name: Install nginx package       # Description of this specific task
         package:                           # Module name (package installs software)
           name: nginx                      # Which package to install
           state: present                   # present = install, absent = remove
           
       - name: Start nginx service         # Another task
         service:                           # Module for managing services
           name: nginx                      # Service name
           state: started                   # started = running, stopped = not running
           enabled: yes                     # enabled = start on boot
   ```

4. **Modules are the building blocks that do actual work.** Each task uses a module. Common ones include `package`, `service`, `copy`, `template`, `user`, `file`.
   ```yaml
   tasks:
     - name: Create a user
       user:                              # Module for managing users
         name: deploy                     # Username to create
         shell: /bin/bash                 # Which shell to give them
         groups: sudo                     # Which groups to add them to
         
     - name: Copy a file
       copy:                              # Module for copying files
         src: /local/path/config.txt      # Source file on your machine
         dest: /remote/path/config.txt    # Where to put it on remote server
         owner: deploy                    # Who should own the file
         mode: '0644'                     # File permissions (read/write for owner, read for others)
         
     - name: Create a directory
       file:                              # Module for managing files and directories
         path: /app/logs                  # Path to create
         state: directory                 # directory = create folder, file = create file, absent = delete
         owner: deploy
         group: deploy
   ```

5. **Variables make your playbooks flexible.** Define them in the playbook, separate files, or pass them when running.
   ```yaml
   ---
   - name: Deploy application
     hosts: webservers
     vars:                                # Variables defined in the playbook
       app_name: myapp                    # Variable name and value
       app_port: 8080
       app_user: deploy
       
     tasks:
       - name: Create app directory
         file:
           path: "/opt/{{ app_name }}"    # Use variables with {{ variable_name }}
           state: directory
           owner: "{{ app_user }}"
           
       - name: Install app dependencies
         package:
           name: "{{ item }}"             # {{ item }} is special - used with loops
           state: present
         loop:                            # loop runs this task multiple times
           - python3                      # First iteration: item = python3
           - python3-pip                  # Second iteration: item = python3-pip
           - git                          # Third iteration: item = git
   ```

6. **Variable files keep your playbooks clean.** Store variables in separate YAML files and include them.
   ```yaml
   # vars/main.yml
   app_name: myapp
   app_port: 8080
   database_host: db.example.com
   database_name: myapp_prod
   
   # In your playbook
   ---
   - name: Deploy application
     hosts: webservers
     vars_files:                          # Load variables from external files
       - vars/main.yml                    # Path to variable file
       
     tasks:
       - name: Configure database connection
         template:                        # template module processes Jinja2 templates
           src: database.conf.j2          # Template file on your machine
           dest: /etc/myapp/database.conf # Where to put the processed file
   ```

7. **Templates let you create dynamic configuration files.** Use Jinja2 templating to insert variables into config files.
   ```jinja2
   # templates/database.conf.j2
   [database]
   host = {{ database_host }}             # Variable will be replaced with actual value
   port = {{ database_port | default(5432) }}    # Use default value if variable not set
   name = {{ database_name }}
   
   {% if environment == 'production' %}   # Conditional content
   ssl_mode = require
   {% else %}
   ssl_mode = prefer
   {% endif %}
   
   # List all servers in webservers group
   {% for host in groups['webservers'] %}
   backup_server_{{ loop.index }} = {{ host }}
   {% endfor %}
   ```

8. **Handlers run only when notified and only once per playbook.** Perfect for restarting services when config files change.
   ```yaml
   tasks:
     - name: Update nginx config
       template:
         src: nginx.conf.j2
         dest: /etc/nginx/nginx.conf
       notify: restart nginx             # This triggers the handler below
       
     - name: Update app config
       template:
         src: app.conf.j2
         dest: /etc/myapp/app.conf
       notify: restart nginx             # Same handler can be notified multiple times
       
   handlers:                             # Handlers section (runs at end of playbook)
     - name: restart nginx               # Handler name (must match notify name)
       service:
         name: nginx
         state: restarted                # restarted = stop then start
   ```

9. **Conditionals let you run tasks only when certain conditions are met.** Use `when` to check variables, facts, or command results.
   ```yaml
   tasks:
     - name: Install package on Ubuntu
       package:
         name: apache2
         state: present
       when: ansible_distribution == "Ubuntu"    # Only run on Ubuntu servers
       
     - name: Install package on CentOS
       package:
         name: httpd
         state: present
       when: ansible_distribution == "CentOS"    # Only run on CentOS servers
       
     - name: Check if file exists
       stat:                              # stat module gets file information
         path: /etc/myapp/config.txt
       register: config_file              # Save the result in a variable
       
     - name: Create config if missing
       copy:
         content: "default config"
         dest: /etc/myapp/config.txt
       when: not config_file.stat.exists  # Only run if file doesn't exist
   ```

10. **Facts are information Ansible gathers about your servers.** Things like OS version, IP addresses, memory, disk space. Access them with `ansible_` prefix.
    ```yaml
    tasks:
      - name: Show server information
        debug:                           # debug module prints information
          msg: |
            Server: {{ ansible_hostname }}
            OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
            Memory: {{ ansible_memtotal_mb }}MB
            IP: {{ ansible_default_ipv4.address }}
            
      - name: Install different packages based on OS
        package:
          name: "{{ item }}"
          state: present
        loop:
          - "{% if ansible_distribution == 'Ubuntu' %}apache2{% else %}httpd{% endif %}"
          - "{% if ansible_distribution == 'Ubuntu' %}mysql-server{% else %}mariadb-server{% endif %}"
    ```

11. **Roles organize your playbooks into reusable components.** Like functions in programming - write once, use everywhere.
    ```bash
    # Create role structure
    ansible-galaxy init webserver        # Creates folder structure for 'webserver' role
    
    # This creates:
    webserver/
    ├── tasks/main.yml                   # Main tasks for this role
    ├── handlers/main.yml                # Handlers for this role
    ├── vars/main.yml                    # Variables for this role
    ├── defaults/main.yml                # Default variable values
    ├── templates/                       # Template files
    ├── files/                           # Static files to copy
    └── meta/main.yml                    # Role metadata and dependencies
    ```
    
    ```yaml
    # webserver/tasks/main.yml
    ---
    - name: Install web server
      package:
        name: "{{ web_package }}"         # Variable defined in defaults/main.yml
        state: present
        
    - name: Start web server
      service:
        name: "{{ web_service }}"
        state: started
        enabled: yes
        
    # webserver/defaults/main.yml
    ---
    web_package: nginx
    web_service: nginx
    web_port: 80
    
    # Use the role in a playbook
    ---
    - name: Setup web servers
      hosts: webservers
      roles:                             # List of roles to apply
        - webserver                      # Apply the webserver role
        - { role: database, db_name: myapp }    # Apply role with custom variables
    ```

12. **Run playbooks with ansible-playbook command.** Various options control how it runs.
    ```bash
    # Basic playbook run
    ansible-playbook -i inventory.ini playbook.yml
    
    # Run with extra variables
    ansible-playbook -i inventory.ini playbook.yml -e "app_name=myapp environment=prod"
    
    # Run only on specific hosts
    ansible-playbook -i inventory.ini playbook.yml --limit webservers
    
    # Dry run (show what would happen without doing it)
    ansible-playbook -i inventory.ini playbook.yml --check
    
    # Run with different user
    ansible-playbook -i inventory.ini playbook.yml -u deploy
    
    # Ask for sudo password
    ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
    ```

13. **Ad-hoc commands run single tasks without a playbook.** Good for quick one-off tasks.
    ```bash
    # Run a command on all servers
    ansible all -i inventory.ini -m command -a "uptime"
    
    # Install a package
    ansible webservers -i inventory.ini -m package -a "name=htop state=present" --become
    
    # Copy a file
    ansible databases -i inventory.ini -m copy -a "src=backup.sql dest=/tmp/backup.sql"
    
    # Restart a service
    ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become
    
    # Gather facts about servers
    ansible all -i inventory.ini -m setup
    ```

14. **Vault encrypts sensitive data like passwords and API keys.** Never store secrets in plain text.
    ```bash
    # Create encrypted file
    ansible-vault create secrets.yml
    
    # Edit encrypted file
    ansible-vault edit secrets.yml
    
    # Encrypt existing file
    ansible-vault encrypt vars/passwords.yml
    
    # Decrypt file
    ansible-vault decrypt vars/passwords.yml
    ```
    
    ```yaml
    # Use encrypted variables in playbook
    ---
    - name: Deploy with secrets
      hosts: webservers
      vars_files:
        - secrets.yml                    # Load encrypted variables
        
      tasks:
        - name: Configure database
          template:
            src: db.conf.j2
            dest: /etc/app/db.conf
          vars:
            db_password: "{{ vault_db_password }}"    # Use encrypted variable
    ```
    
    ```bash
    # Run playbook with vault password
    ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
    
    # Or use password file
    ansible-playbook -i inventory.ini playbook.yml --vault-password-file ~/.vault_pass
    ```

15. **Error handling controls what happens when tasks fail.** By default, playbook stops on first error.
    ```yaml
    tasks:
      - name: Try to start service
        service:
          name: myapp
          state: started
        ignore_errors: yes               # Continue even if this task fails
        
      - name: Download file with retry
        get_url:                         # Module for downloading files
          url: https://example.com/file.zip
          dest: /tmp/file.zip
        retries: 3                       # Try up to 3 times
        delay: 5                         # Wait 5 seconds between retries
        
      - name: Run command and handle failure
        command: /opt/myapp/backup.sh
        register: backup_result          # Save result to check later
        failed_when: backup_result.rc != 0 and "No space" not in backup_result.stderr
        # Only fail if return code isn't 0 AND error isn't about disk space
        
      - name: Always run cleanup
        file:
          path: /tmp/tempfile
          state: absent
        tags: always                     # Run this task even if others fail
    ```

16. **Tags let you run only specific parts of a playbook.** Useful for running just updates or just configuration changes.
    ```yaml
    tasks:
      - name: Install packages
        package:
          name: "{{ item }}"
          state: present
        loop:
          - nginx
          - mysql-server
        tags:                            # Tags for this task
          - packages                     # Tag name (you choose these)
          - install
          
      - name: Configure nginx
        template:
          src: nginx.conf.j2
          dest: /etc/nginx/nginx.conf
        tags:
          - config                       # Different tag
          - nginx
        notify: restart nginx
        
      - name: Start services
        service:
          name: "{{ item }}"
          state: started
        loop:
          - nginx
          - mysql
        tags:
          - services
          - start
    ```
    
    ```bash
    # Run only tasks with specific tags
    ansible-playbook -i inventory.ini playbook.yml --tags "config"
    
    # Run multiple tags
    ansible-playbook -i inventory.ini playbook.yml --tags "config,services"
    
    # Skip specific tags
    ansible-playbook -i inventory.ini playbook.yml --skip-tags "packages"
    ```

17. **Blocks group tasks together and handle errors for the whole group.** Like try/catch in programming.
    ```yaml
    tasks:
      - name: Database setup block
        block:                           # Group of tasks that run together
          - name: Create database user
            mysql_user:
              name: appuser
              password: "{{ db_password }}"
              state: present
              
          - name: Create database
            mysql_db:
              name: myapp
              state: present
              
          - name: Import database schema
            mysql_db:
              name: myapp
              state: import
              target: /tmp/schema.sql
              
        rescue:                          # Run these tasks if any task in block fails
          - name: Log database setup failure
            debug:
              msg: "Database setup failed, rolling back"
              
          - name: Remove database
            mysql_db:
              name: myapp
              state: absent
              
        always:                          # Always run these tasks (success or failure)
          - name: Clean up temp files
            file:
              path: /tmp/schema.sql
              state: absent
              
        when: setup_database | default(false)    # Only run this block if variable is true
    ```

18. **Dynamic inventory gets server lists from cloud providers or other sources.** Instead of static inventory files.
    ```bash
    # Use AWS dynamic inventory
    ansible-playbook -i aws_ec2.yml playbook.yml
    
    # Use custom script
    ansible-playbook -i ./dynamic_inventory.py playbook.yml
    ```
    
    ```yaml
    # aws_ec2.yml - AWS dynamic inventory config
    plugin: aws_ec2                     # Which dynamic inventory plugin to use
    regions:                             # Which AWS regions to search
      - us-east-1
      - us-west-2
    keyed_groups:                        # Create groups based on instance properties
      - key: tags.Environment            # Group by Environment tag
        prefix: env                      # Group names will be env_production, env_staging, etc.
      - key: instance_type               # Group by instance type
        prefix: type                     # Group names will be type_t3_micro, type_t3_small, etc.
    ```

19. **Ansible configuration file controls default behavior.** Create `ansible.cfg` in your project directory.
    ```ini
    [defaults]
    inventory = inventory.ini            # Default inventory file
    remote_user = deploy                 # Default SSH user
    private_key_file = ~/.ssh/deploy_key # Default SSH key
    host_key_checking = False            # Don't verify SSH host keys (useful for cloud instances)
    retry_files_enabled = False          # Don't create .retry files on failure
    gathering = smart                    # Only gather facts when needed (faster)
    
    [ssh_connection]
    ssh_args = -o ControlMaster=auto -o ControlPersist=60s    # Reuse SSH connections (faster)
    pipelining = True                    # Send multiple commands in one SSH connection (faster)
    ```

20. **Best practices for writing maintainable playbooks:**
    ```yaml
    # Use descriptive names
    - name: Install and configure nginx web server for production environment
      # Not: Install nginx
      
    # Use variables instead of hardcoded values
    vars:
      app_port: 8080
      app_user: deploy
      
    # Group related tasks
    - name: Package installation
      package:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - python3
        - git
        
    # Use handlers for service restarts
    notify: restart nginx
    
    # Add comments for complex logic
    - name: Configure firewall rules
      # Allow HTTP traffic on port 80 and HTTPS on port 443
      # Block all other incoming traffic except SSH
      ufw:
        rule: allow
        port: "{{ item }}"
      loop:
        - 22    # SSH
        - 80    # HTTP
        - 443   # HTTPS
        
    # Use check mode compatible tasks
    - name: Ensure directory exists
      file:
        path: /opt/myapp
        state: directory
      # This works with --check flag
      
    # Test your playbooks
    # 1. Run with --check first
    # 2. Run on test environment
    # 3. Use --limit to test on single server
    # 4. Then run on production
    ``` 

## Cookbook - Common Scenarios

21. **Deploy a web application with database** - Complete setup from scratch.
    ```yaml
    ---
    - name: Deploy LAMP stack application
      hosts: webservers
      become: yes
      vars:
        app_name: mywebapp
        app_user: www-data
        db_name: mywebapp_db
        db_user: webapp_user
        
      tasks:
        # Install required packages
        - name: Install web server and database
          package:
            name: "{{ item }}"
            state: present
          loop:
            - apache2
            - mysql-server
            - php
            - php-mysql
            - python3-pymysql          # Needed for mysql modules
            
        # Create application directory
        - name: Create app directory
          file:
            path: "/var/www/{{ app_name }}"
            state: directory
            owner: "{{ app_user }}"
            group: "{{ app_user }}"
            mode: '0755'
            
        # Download and extract application
        - name: Download application code
          get_url:
            url: "https://github.com/mycompany/{{ app_name }}/archive/main.zip"
            dest: "/tmp/{{ app_name }}.zip"
            
        - name: Extract application
          unarchive:
            src: "/tmp/{{ app_name }}.zip"
            dest: "/var/www/{{ app_name }}"
            remote_src: yes             # File is already on remote server
            owner: "{{ app_user }}"
            
        # Configure database
        - name: Create database
          mysql_db:
            name: "{{ db_name }}"
            state: present
            
        - name: Create database user
          mysql_user:
            name: "{{ db_user }}"
            password: "{{ vault_db_password }}"    # From encrypted vars
            priv: "{{ db_name }}.*:ALL"
            state: present
            
        # Configure Apache
        - name: Create Apache virtual host
          template:
            src: apache-vhost.conf.j2
            dest: "/etc/apache2/sites-available/{{ app_name }}.conf"
          notify: restart apache
          
        - name: Enable site
          command: "a2ensite {{ app_name }}"
          notify: restart apache
          
        - name: Disable default site
          command: a2dissite 000-default
          notify: restart apache
          
      handlers:
        - name: restart apache
          service:
            name: apache2
            state: restarted
    ```

22. **Zero-downtime deployment with rolling updates** - Update application without service interruption.
    ```yaml
    ---
    - name: Rolling deployment
      hosts: webservers
      serial: 1                         # Update one server at a time
      max_fail_percentage: 0            # Stop if any server fails
      vars:
        app_version: "{{ version | default('latest') }}"
        health_check_url: "http://{{ inventory_hostname }}/health"
        
      tasks:
        # Remove server from load balancer
        - name: Remove from load balancer
          uri:                          # Make HTTP requests
            url: "http://{{ load_balancer_ip }}/remove/{{ inventory_hostname }}"
            method: POST
          delegate_to: localhost        # Run this on your machine, not the server
          
        # Wait for connections to drain
        - name: Wait for connections to drain
          wait_for:                     # Wait for specific conditions
            timeout: 30                 # Wait up to 30 seconds
            
        # Deploy new version
        - name: Stop application
          service:
            name: myapp
            state: stopped
            
        - name: Backup current version
          copy:
            src: /opt/myapp/
            dest: "/opt/myapp.backup.{{ ansible_date_time.epoch }}"
            remote_src: yes
            
        - name: Download new version
          get_url:
            url: "https://releases.mycompany.com/myapp-{{ app_version }}.tar.gz"
            dest: /tmp/myapp-new.tar.gz
            
        - name: Extract new version
          unarchive:
            src: /tmp/myapp-new.tar.gz
            dest: /opt/myapp/
            remote_src: yes
            
        - name: Start application
          service:
            name: myapp
            state: started
            
        # Health check
        - name: Wait for application to start
          uri:
            url: "{{ health_check_url }}"
            status_code: 200
          retries: 10
          delay: 5
          
        # Add back to load balancer
        - name: Add back to load balancer
          uri:
            url: "http://{{ load_balancer_ip }}/add/{{ inventory_hostname }}"
            method: POST
          delegate_to: localhost
    ```

23. **Backup and restore database** - Automated database backup with rotation.
    ```yaml
    ---
    - name: Database backup and maintenance
      hosts: databases
      become: yes
      vars:
        backup_dir: /var/backups/mysql
        retention_days: 7
        
      tasks:
        # Create backup directory
        - name: Create backup directory
          file:
            path: "{{ backup_dir }}"
            state: directory
            owner: mysql
            group: mysql
            mode: '0750'
            
        # Backup all databases
        - name: Backup databases
          mysql_db:
            name: all                   # Backup all databases
            state: dump                 # Create SQL dump
            target: "{{ backup_dir }}/all-databases-{{ ansible_date_time.date }}.sql"
            
        # Compress backup
        - name: Compress backup
          archive:                      # Create compressed archive
            path: "{{ backup_dir }}/all-databases-{{ ansible_date_time.date }}.sql"
            dest: "{{ backup_dir }}/all-databases-{{ ansible_date_time.date }}.sql.gz"
            format: gz                  # Use gzip compression
            remove: yes                 # Remove original file after compression
            
        # Clean old backups
        - name: Find old backups
          find:                         # Find files matching criteria
            paths: "{{ backup_dir }}"
            patterns: "*.sql.gz"
            age: "{{ retention_days }}d"    # Files older than X days
          register: old_backups
          
        - name: Remove old backups
          file:
            path: "{{ item.path }}"
            state: absent
          loop: "{{ old_backups.files }}"
          
        # Upload to cloud storage (optional)
        - name: Upload to S3
          aws_s3:                       # AWS S3 module
            bucket: my-database-backups
            object: "mysql/{{ inventory_hostname }}/all-databases-{{ ansible_date_time.date }}.sql.gz"
            src: "{{ backup_dir }}/all-databases-{{ ansible_date_time.date }}.sql.gz"
            mode: put
          when: upload_to_s3 | default(false)
    ```

24. **Server hardening and security** - Apply security best practices.
    ```yaml
    ---
    - name: Server security hardening
      hosts: all
      become: yes
      vars:
        allowed_ssh_users:
          - deploy
          - admin
        ssh_port: 2222
        
      tasks:
        # Update system packages
        - name: Update all packages
          package:
            name: "*"
            state: latest
            
        # Configure SSH security
        - name: Configure SSH
          lineinfile:                   # Modify lines in files
            path: /etc/ssh/sshd_config
            regexp: "{{ item.regexp }}"
            line: "{{ item.line }}"
            backup: yes                 # Create backup before changing
          loop:
            - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
            - { regexp: '^#?PasswordAuthentication', line: 'PasswordAuthentication no' }
            - { regexp: '^#?Port', line: "Port {{ ssh_port }}" }
            - { regexp: '^#?AllowUsers', line: "AllowUsers {{ allowed_ssh_users | join(' ') }}" }
          notify: restart ssh
          
        # Configure firewall
        - name: Install UFW firewall
          package:
            name: ufw
            state: present
            
        - name: Configure UFW defaults
          ufw:
            direction: "{{ item.direction }}"
            policy: "{{ item.policy }}"
          loop:
            - { direction: 'incoming', policy: 'deny' }
            - { direction: 'outgoing', policy: 'allow' }
            
        - name: Allow SSH on custom port
          ufw:
            rule: allow
            port: "{{ ssh_port }}"
            proto: tcp
            
        - name: Enable UFW
          ufw:
            state: enabled
            
        # Install security tools
        - name: Install security packages
          package:
            name: "{{ item }}"
            state: present
          loop:
            - fail2ban              # Intrusion prevention
            - unattended-upgrades   # Automatic security updates
            - aide                  # File integrity monitoring
            
        # Configure fail2ban
        - name: Configure fail2ban for SSH
          copy:
            content: |
              [sshd]
              enabled = true
              port = {{ ssh_port }}
              filter = sshd
              logpath = /var/log/auth.log
              maxretry = 3
              bantime = 3600
            dest: /etc/fail2ban/jail.local
          notify: restart fail2ban
          
      handlers:
        - name: restart ssh
          service:
            name: ssh
            state: restarted
            
        - name: restart fail2ban
          service:
            name: fail2ban
            state: restarted
    ```

25. **Docker container deployment** - Deploy and manage Docker containers.
    ```yaml
    ---
    - name: Deploy Docker application
      hosts: docker_hosts
      become: yes
      vars:
        app_name: myapp
        app_version: latest
        app_port: 8080
        
      tasks:
        # Install Docker
        - name: Install Docker dependencies
          package:
            name: "{{ item }}"
            state: present
          loop:
            - apt-transport-https
            - ca-certificates
            - curl
            - software-properties-common
            
        - name: Add Docker GPG key
          apt_key:                      # Manage APT repository keys
            url: https://download.docker.com/linux/ubuntu/gpg
            state: present
            
        - name: Add Docker repository
          apt_repository:               # Manage APT repositories
            repo: "deb https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
            state: present
            
        - name: Install Docker
          package:
            name: docker-ce
            state: present
            
        - name: Start Docker service
          service:
            name: docker
            state: started
            enabled: yes
            
        # Deploy application
        - name: Pull Docker image
          docker_image:                 # Manage Docker images
            name: "mycompany/{{ app_name }}"
            tag: "{{ app_version }}"
            source: pull                # Pull from registry
            
        - name: Stop old container
          docker_container:             # Manage Docker containers
            name: "{{ app_name }}"
            state: stopped
          ignore_errors: yes            # Don't fail if container doesn't exist
          
        - name: Remove old container
          docker_container:
            name: "{{ app_name }}"
            state: absent
          ignore_errors: yes
          
        - name: Start new container
          docker_container:
            name: "{{ app_name }}"
            image: "mycompany/{{ app_name }}:{{ app_version }}"
            state: started
            restart_policy: always      # Restart if container stops
            ports:
              - "{{ app_port }}:8080"   # Map host port to container port
            env:                        # Environment variables for container
              DATABASE_URL: "{{ database_url }}"
              API_KEY: "{{ vault_api_key }}"
            volumes:                    # Mount directories into container
              - "/var/log/{{ app_name }}:/app/logs"
              - "/etc/{{ app_name }}:/app/config:ro"    # :ro = read-only
              
        # Health check
        - name: Wait for application to be ready
          uri:
            url: "http://{{ inventory_hostname }}:{{ app_port }}/health"
            status_code: 200
          retries: 10
          delay: 5
    ```

26. **SSL certificate management with Let's Encrypt** - Automated SSL certificate setup.
    ```yaml
    ---
    - name: Setup SSL certificates
      hosts: webservers
      become: yes
      vars:
        domain_name: myapp.example.com
        email: admin@example.com
        
      tasks:
        # Install Certbot
        - name: Install snapd
          package:
            name: snapd
            state: present
            
        - name: Install certbot via snap
          snap:                         # Manage snap packages
            name: certbot
            classic: yes                # Allow classic confinement
            
        # Stop web server temporarily
        - name: Stop nginx for certificate generation
          service:
            name: nginx
            state: stopped
            
        # Generate certificate
        - name: Generate SSL certificate
          command: >
            certbot certonly
            --standalone
            --non-interactive
            --agree-tos
            --email {{ email }}
            -d {{ domain_name }}
          args:
            creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"
            
        # Configure nginx with SSL
        - name: Configure nginx SSL
          template:
            src: nginx-ssl.conf.j2
            dest: /etc/nginx/sites-available/{{ domain_name }}
          notify: restart nginx
          
        - name: Enable SSL site
          file:
            src: "/etc/nginx/sites-available/{{ domain_name }}"
            dest: "/etc/nginx/sites-enabled/{{ domain_name }}"
            state: link
          notify: restart nginx
          
        # Setup automatic renewal
        - name: Setup certificate renewal cron job
          cron:                         # Manage cron jobs
            name: "Renew SSL certificates"
            minute: "0"
            hour: "2"
            job: "certbot renew --quiet && systemctl reload nginx"
            user: root
            
      handlers:
        - name: restart nginx
          service:
            name: nginx
            state: restarted
    ```

27. **Multi-environment deployment** - Deploy to dev, staging, and production with different configs.
    ```yaml
    # group_vars/dev.yml
    ---
    environment: development
    database_host: dev-db.internal
    app_replicas: 1
    debug_mode: true
    
    # group_vars/staging.yml
    ---
    environment: staging
    database_host: staging-db.internal
    app_replicas: 2
    debug_mode: false
    
    # group_vars/production.yml
    ---
    environment: production
    database_host: prod-db.internal
    app_replicas: 5
    debug_mode: false
    
    # deploy.yml
    ---
    - name: Deploy application to {{ environment }}
      hosts: "{{ target_env }}"         # Pass target_env when running playbook
      become: yes
      vars:
        app_name: myapp
        
      tasks:
        - name: Validate environment
          assert:                       # Ensure conditions are met
            that:
              - environment is defined
              - environment in ['development', 'staging', 'production']
            fail_msg: "Invalid or undefined environment"
            
        - name: Show deployment info
          debug:
            msg: |
              Deploying to: {{ environment }}
              Database: {{ database_host }}
              Replicas: {{ app_replicas }}
              Debug: {{ debug_mode }}
              
        - name: Deploy application config
          template:
            src: app-config.j2
            dest: "/etc/{{ app_name }}/config.yml"
          vars:
            db_host: "{{ database_host }}"
            replicas: "{{ app_replicas }}"
            debug: "{{ debug_mode }}"
          notify: restart app
          
        # Production-only tasks
        - name: Setup monitoring (production only)
          include_tasks: monitoring.yml
          when: environment == 'production'
          
        # Development-only tasks
        - name: Install debug tools (development only)
          package:
            name: "{{ item }}"
            state: present
          loop:
            - strace
            - tcpdump
            - htop
          when: environment == 'development'
          
      handlers:
        - name: restart app
          service:
            name: "{{ app_name }}"
            state: restarted
    ```
    
    ```bash
    # Run deployment commands
    # Deploy to development
    ansible-playbook -i inventory.ini deploy.yml -e "target_env=dev"
    
    # Deploy to staging
    ansible-playbook -i inventory.ini deploy.yml -e "target_env=staging"
    
    # Deploy to production (with confirmation)
    ansible-playbook -i inventory.ini deploy.yml -e "target_env=production" --ask-vault-pass
    ```

28. **Log aggregation setup** - Centralized logging with ELK stack.
    ```yaml
    ---
    - name: Setup centralized logging
      hosts: log_servers
      become: yes
      vars:
        elasticsearch_version: 7.17.0
        
      tasks:
        # Install Java (required for Elasticsearch)
        - name: Install Java
          package:
            name: openjdk-11-jdk
            state: present
            
        # Install Elasticsearch
        - name: Add Elasticsearch repository key
          apt_key:
            url: https://artifacts.elastic.co/GPG-KEY-elasticsearch
            state: present
            
        - name: Add Elasticsearch repository
          apt_repository:
            repo: "deb https://artifacts.elastic.co/packages/7.x/apt stable main"
            state: present
            
        - name: Install Elasticsearch
          package:
            name: elasticsearch
            state: present
            
        - name: Configure Elasticsearch
          lineinfile:
            path: /etc/elasticsearch/elasticsearch.yml
            regexp: "{{ item.regexp }}"
            line: "{{ item.line }}"
          loop:
            - { regexp: '^#?network.host:', line: 'network.host: 0.0.0.0' }
            - { regexp: '^#?discovery.type:', line: 'discovery.type: single-node' }
          notify: restart elasticsearch
          
        # Install Logstash
        - name: Install Logstash
          package:
            name: logstash
            state: present
            
        - name: Configure Logstash pipeline
          copy:
            content: |
              input {
                beats {
                  port => 5044
                }
              }
              
              filter {
                if [fileset][module] == "nginx" {
                  grok {
                    match => { "message" => "%{NGINXACCESS}" }
                  }
                }
              }
              
              output {
                elasticsearch {
                  hosts => ["localhost:9200"]
                  index => "logs-%{+YYYY.MM.dd}"
                }
              }
            dest: /etc/logstash/conf.d/main.conf
          notify: restart logstash
          
        # Install Kibana
        - name: Install Kibana
          package:
            name: kibana
            state: present
            
        - name: Configure Kibana
          lineinfile:
            path: /etc/kibana/kibana.yml
            regexp: "{{ item.regexp }}"
            line: "{{ item.line }}"
          loop:
            - { regexp: '^#?server.host:', line: 'server.host: "0.0.0.0"' }
            - { regexp: '^#?elasticsearch.hosts:', line: 'elasticsearch.hosts: ["http://localhost:9200"]' }
          notify: restart kibana
          
        # Start services
        - name: Start and enable services
          service:
            name: "{{ item }}"
            state: started
            enabled: yes
          loop:
            - elasticsearch
            - logstash
            - kibana
            
      handlers:
        - name: restart elasticsearch
          service:
            name: elasticsearch
            state: restarted
            
        - name: restart logstash
          service:
            name: logstash
            state: restarted
            
        - name: restart kibana
          service:
            name: kibana
            state: restarted
            
    # Configure log shipping on application servers
    - name: Setup log shipping
      hosts: webservers
      become: yes
      vars:
        logstash_host: "{{ groups['log_servers'][0] }}"
        
      tasks:
        - name: Install Filebeat
          get_url:
            url: "https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.17.0-amd64.deb"
            dest: /tmp/filebeat.deb
            
        - name: Install Filebeat package
          apt:
            deb: /tmp/filebeat.deb
            
        - name: Configure Filebeat
          template:
            src: filebeat.yml.j2
            dest: /etc/filebeat/filebeat.yml
          notify: restart filebeat
          
        - name: Enable and start Filebeat
          service:
            name: filebeat
            state: started
            enabled: yes
            
      handlers:
        - name: restart filebeat
          service:
            name: filebeat
            state: restarted
    ``` 