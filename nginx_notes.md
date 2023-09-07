# NGINX Notes
- A server that serves web content to our web browser

### Features
- Reverse proxy
- Load Balancer
- Encryption
  - Instead of encrypting every single application server behind an nginx load balancer, perform encryption/decryption on the nginx server

### Configurations
- Listens on port 80 and 443

### Process (as a reverse proxy)
1. Web browser makes request to nginx server
2. nginx server makes request to application server
3. Application server responds to the nginx server's request
4. nginx server sends the application content back to the web browser

### Local installation
1. Configure docker-compose.yml file
   - Basic `docker-compose.yml` for a containerized nginx deployment:
     ```yml
     version: "3.8"

     services:
       nginx:
         image: nginx:latest
         ports:
           - "80:80"
         volumes:
           - myapp:/home
           - nginx:/etc/nginx 
     
     volumes:
       myapp:
       nginx:
     ```
2. Spin up docker nginx container using the `docker-compose.yml` file
   - Navigate to the directory where you just created the `docker-compose.yml` above
   - Run this command to build and run the nginx container in the background (`-d` option runs containers in background):
     ```
     docker-compose up -d
     ```
3. Use the `docker exec` command to enter the `bash` terminal of the nginx container:
   ```
   docker exec -it <name of your running container> bash
   ```
   > You should see the prompt change to the prompt of the nginx container
4. Install the `nano` program from the nginx container bash terminal
   ```
   apt-get update
   apt-get install nano -y
   ```
   > You can now use the `nano` program to view and edit files in the nginx container
   > If you re-start the nginx container, you will need to re-install `nano`
5. View the nginx configuration file: `nginx.conf` in the /etc/nginx/ directory
   - Command:
     ```
     nano /etc/nginx/nginx.conf
     ```
   - Example:
     ```
     user  nginx;
     worker_processes  auto;
 
     error_log  /var/log/nginx/error.log notice;
     pid        /var/run/nginx.pid;
 
 
     events {
         worker_connections  1024;
     }
 
 
     http {
         include       /etc/nginx/mime.types;
         default_type  application/octet-stream;
 
         log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
 
         access_log  /var/log/nginx/access.log  main;
 
         sendfile        on;
         #tcp_nopush     on;
 
         keepalive_timeout  65;
 
         #gzip  on;
 
         include /etc/nginx/conf.d/*.conf;
     }
     ```


### Serve a static file from a local directory
- Create the directory `/home/mywebpage` in the nginx container and change directory into it:
  ```
  cd /home
  mkdir mywebpage
  cd mywebpage
  ```
- Create a `index.html` file and save it to the `/home/mywebpage/` directory
  - `index.html`
    ```html
    <html>
        <body>
        <h1>Welcome to my first NGINX webpage!</h1>
        </body>
    </html>
    ```
- Edit the `nginx.conf` file to point to the locally saved static webpage
  - `nginx.conf`
    - Delete everyting in the `nginx.conf` file and add the code below:
      ```
      http {
          server {
              listen 80;
              root /home/mywebpage;
          }
      }
      ```
- Restart the nginx service
  ```
  nginx -s reload
  ```

### Configure the types - Manually
- A webpage will have different types of resources that will be referenced when working in nginx
- You must tell nginx what to do with certain types of files
- This is done in the `nginx.conf` file under the `types` key:
  ```
  http {
      types {
          text/css       css;
          text/html       html;
      }
  
      server {
          listen 80;
          root /home/mywebpage;
      }
  }
  ```
- The nginx service must be reloaded for the `nginx.conf` changes to take effect:
  ```
  nginx -s reload
  ```

### Configure the types - Automatically
- nginx comes with default `mime.types`
  - This file defines several common mime types
  - Located in the `/etc/nginx/mime.types` file
- Use the `include` keyword in the `nginx.conf` file to use the `mime.types` file entries:
  ```
  http {
      include mime.types;
  
      server {
          listen 80;
          root /home/mywebpage;
      }
  }
  ```

### Location context
- The `server` keyword sets the configuration for a virtual server
  - You can configure a proxy with the `listen 80;` syntax which defines the port to listen on
  - `root` keyword defines the base directory to server files from
  - `location` keyword:
    - Responds to webpage calls from the browser to the designated path
    - `root`
      - The `root` keyword in the `location` code section will add the `location` path onto the end of the `root` path
      - `root` example:
        ```
        http {
            include mime.types;
        
            server {
                listen 80;  # Listens for web browser traffic on port 80
                root /home/mywebpage;  # Serves up whatever files are in the /home/mywebpage directory
      
                location /fruits {  # Listens for calls to the http://webpage:80/fruits page
                    root /home/mywebpage;  # Responds with files in the /home/mywebpage/fruits directory
                }
            }
        }
        ```
    - `alias`
      - The `alias` keyword in the `location` code section will define a specific path for a web url path to go to
      - `alias` does not append anything to the end of the path defined after the `alias` keyword
      - `alias` example:
        ```
        http {
            include mime.types;
        
            server {
                listen 80;  # Listens for web browser traffic on port 80
                root /home/mywebpage;  # Serves up whatever files are in the /home/mywebpage directory
      
                location /fruits {  # Listens for calls to the http://webpage:80/fruits page
                    root /home/mywebpage;  # Responds with files in the /home/mywebpage/fruits directory
                }

                location /carbs {  # Listens for calls to the http://webpage:80/carbs page
                    alias /home/mywebpage/fruits;  # Responds with files in the /home/mywebpage/fruits directory
                }
            }
        }
        ```
    - `try_files`
      - The `try_files` keyword will list options for a web url to try
      - If the first option is not found, it will move on to the next option
      - You can define directory paths, specific file names, and even html return codes such as `404`
      - `try_files` example:
        ```
        http {
            include mime.types;
        
            server {
                listen 80;  # Listens for web browser traffic on port 80
                root /home/mywebpage;  # Serves up whatever files are in the /home/mywebpage directory
      
                location /fruits {  # Listens for calls to the http://webpage:80/fruits page
                    root /home/mywebpage;  # Responds with files in the /home/mywebpage/fruits directory
                }

                location /carbs {  # Listens for calls to the http://webpage:80/carbs page
                    alias /home/mywebpage/fruits;  # Responds with files in the /home/mywebpage/fruits directory
                }

                location /vegetables {  # Listens for calls to the http://webpage:80/vegetables page
                    root /home/mywebpage;  # Responds with files in the /home/mywebpage/vegetables directory
                    try_files /vegetables/veggies.html /index.html =404;
                    # Tries to respond with the file /vegetables/veggies.html. If it is not
                    #   found, tries to respond with the file /index.html. If it is not
                    #   found, responds with the 404 error code page.
                }
            }
        }
        ```
    - `Regular Expressions`
      - Can be used in the `location` section to define patterns for matching
      - Declared with `~*`
      - Example:
        ```
        http {
            include mime.types;

            server {
                listen 80;  # Listens for web browser traffic on port 80
                root /home/mywebpage;  # Serves up whatever files are in the /home/mywebpage directory
      
                location ~* /count/[0-9] {  # Listens for calls to the http://webpage:80/ page with any number 0 - 9
                    root /home/mywebpage;  # Serves up whatever files are in the /home/mywebpage/count/[number] directory
                    try_files /index.html =404;  # Tries to find an index.html file in the directory. If none, responds with 404
                }

                location /fruits {  # Listens for calls to the http://webpage:80/fruits page
                    root /home/mywebpage;  # Responds with files in the /home/mywebpage/fruits directory
                }
            }
        }
        ```

### Redirects and Rewrites
- Redirects
  - Allows you to send a return code redirect to a different location
  - Actually sends the web browser to the path location of the redirect
  - Example:
    ```
    html {
        include mime.types;

        server {
            listen 80;
            root /home/mywebpage;

            location /fruits {
                root /home/mywebpage;
            }

            location /crops {
                return 307 /fruits;  # Returns a 307 redirect to the /fruits location path
            }

        }
    }
    ```
- Rewrites
  - Keeps the web browser on the current page
  - Rewrites the content of the page from the specified rewrite location
  - Parameters of the url are assigned as a variable using the `(\w+)` syntax
  - Varaiables are consumed using the `$1` syntax
  - Example:
    ```
    http {
        include mime.types;

        server {
            listen 80;
            root /home/mywebpage;
            
            rewrite ^/number/(\w+) /count/$1;

            location ~* /count/[0-9] {
                root /home/mywebpage;
                try_files /index.html =404;
            }
        }
    }
    ```
### Load Balancer
- Allows scaling of applications behind load balancer
- nginx handles forwarding internet requests to application servers using an alorithm
- Example:
  ```
  http {
    include mime.types;

    upstream backendserver {  # Defines the name of the upstream server pool for load-balancing
        server 127.0.0.1:1111;
        server 127.0.0.1:2222;
        server 127.0.0.1:3333;
        server 127.0.0.1:4444;
    }

    server {
        listen 8080;
        root /home/mywebpage
    
        location / {
            proxy_pass http://backendserver/;  # Defines the upstream server group to use for load-balancing
        }
    }
  }
  ```
  - In the example above, you have already created 4 web servers that are running locally on ports 1111, 2222, 3333, and 4444



# Working with docker containers
### Configuring nginx in container
- Docker creates a docker network dns entry for the image name (service) that is created
  - Example `docker-compose.yml` file:
    ```yml
    version: '3.8'
    services:
      nginx:
        image: nginx:latest
        ports:
          - "80:80"
        volumes:
          - myapp:/home
          - nginx:/etc/nginx 
    
      my-webpage:
        build: ./my-webpage  # built from a dockerfile in the ./my-webpage directory
        restart: unless-stopped
        ports:
          - "4444:8080"
      
      another-webpage:
        build: ./another-webpage  # built from a dockerfile in the ./another-webpage directory
        restart: unless-stopped
        ports:
          - "5555:8080"
    
    volumes:
      myapp:
      nginx:
    ```
  - In the `nginx.conf` file you can route to the name of the docker services using the interior port (service port in docker)
    - Example `nginx.conf` file using services create in the above `docker-compose.yml`
      ```
      http {
        include mime.types;
        
        server {
            listen 80;
            server_name localhost 127.0.0.1
        
            location / {
                proxy_pass          http://my-webpage:8080;
            }

            location /another/ {
                proxy_pass          http://another-webpage:8080/;
            }
        }
      }

      events {}
      ```
    - There is a small gotcha to watch out for in the example above:
      - You must add the `/` after `/another/`
      - You must also add the `/` after `http://another-webpage:8080/`
      - If you leave either one of these off, then the Chrome broswer will complain about not being able to `Get` your webpage

### Round-robin load balancing in nginx
- Using the above `docker-compose.yml` example, you can configure nginx for load balancing across multiple application containers
- Example `nginx.conf` configuration:
  ```
  http {
    include mime.types;
    
    upstream allapps {
        server my-webpage:8080;
        server another-webpage:8080;
    }

    server {
        listen 80;
        server_name localhost 127.0.0.1
    
        location / {
            proxy_pass          http://allaps/;
        }

        location /webpage/ {
            proxy_pass          http://my-webpage:8080/;
        }
        
        location /another/ {
            proxy_pass          http://another-webpage:8080/;
        }
    }
  }
  ```
- In the example above:
  - The `upstream` keyword is used with the `allapps` upstream custom label to define the load-balancing application server group
  - Specific server endpoints of the load-balancing group are defined in the `{}` of the `upstream` section
  - The `upstream allapps` load-balancing group is routed from the `location / { ... }` declaration in which the `proxy_pass` is pointed to the `upstream allapps` load-balancing group
  - The `upstream allapps` load-balancing group routes to a diffent endpoint in the load-balancing group using the `round-robin` algorithm

### Deploying nginx using templates
- In the above `docker-compose.yml` there is no nginx configurations, and you are required to use the `docker exec -it <name of container> bash` command to get into the nginx running container's bash terminal
- Instead of performing this manual process, you can create a template file outside of the images and containers and then copy this template file or directory to the `/etc/nginx/templates` directory
- Steps to create:
  1. Create an nginx template directory in your container build directories that are accessable from your `docker-compose.yml` file
     ```
     my_application
         ├── application_files
         ├── dev
         │   └── nginx
         │       └── templates
         │           └── default.conf.template
         └── docker-compose.yml
     ```
  2. Edit and save the `default.conf.template` file to reflect the nginx routing that you desire
     - Example:
       ```
        server {
            listen 80;
            server_name localhost 127.0.0.1
        
            location / {
                proxy_pass          http://my-webpage:8080;
            }

            location /another/ {
                proxy_pass          http://another-webpage:8080/;
            }
        }
       ```
  3. Copy the `./dev/nginx/templates` directory from your local computer to the `/etc/nginx/templates` directory of your nginx container at build time by using the `docker-compose.yml` file's entry: `- ./dev/nginx/templates:/etc/nginx/templates:ro`  :ro designate "read only" on the destination directory:
     - Example:
       ```
       ...
       services:
         
         my-nginx:
           image: nginx
           ports:
             - "80:80"
           volumes:
             - nginx:/etc/nginx
             - ./dev/nginx/templates:/etc/nginx/templates:ro
         
         < add other apps as necessary >
       
       volumes:
         nginx:
       ...
       ```
    4. Run the `docker-compose up -d` command to build and run your images and containers.
       - Your nginx configurations should now be in your nginx container
       - If your nginx container is crash looping or just crashing, there is most likely a problem with your `default.conf.template` file