
# ALU Webstack - HTTPS/SSL

This project contains configurations and scripts for setting up SSL certificates, HAProxy SSL termination, and HTTP to HTTPS redirection. SSL (Secure Sockets Layer) is used to encrypt data between a web server and a client to ensure secure communication.

## Cloning the Repository

To clone this repository, follow these steps:

1. Open a terminal.
2. Navigate to the directory where you want to clone the repository.
3. Run the following command:

   ```bash
   git clone https://github.com/DLOADIN/alu-webstack.git
   ```

4. Once cloned, navigate into the project directory:

   ```bash
   cd alu-webstack/https_ssl
   ```

## Repository Structure

### 1. `0-world_wide_web/`
This directory likely contains configurations for a basic web stack setup before implementing SSL. It is intended to demonstrate how to create and configure a basic web server infrastructure.

- **Purpose**: Set up a standard web environment as a base for SSL implementation.
- **Files**: Contains basic web server setup configurations (likely without SSL).

### 2. `1-haproxy_ssl_termination/`
This folder contains configurations for terminating SSL at the HAProxy level. HAProxy SSL termination means that the encrypted traffic is decrypted at the load balancer (HAProxy), and the communication between the load balancer and the backend servers is unencrypted.

- **Purpose**: Offload SSL processing to HAProxy, simplifying SSL certificate management and reducing the load on backend servers.
- **Files**: Contains HAProxy configuration scripts for setting up SSL termination.

### 3. `2-redirect_http_to_https/`
This directory holds scripts and configurations for redirecting HTTP traffic to HTTPS. HTTP is not secure, and this step forces all users to use a secure HTTPS connection instead.

- **Purpose**: Ensure that all traffic is securely encrypted by redirecting HTTP requests to HTTPS.
- **Files**: Configuration files that redirect traffic from port 80 (HTTP) to port 443 (HTTPS).

### 4. `README.md`
This is the documentation file explaining the structure and usage of the project.

## SSL Certificate Setup

### Generating a Self-Signed SSL Certificate
If you don't have a certificate from a Certificate Authority (CA), you can generate a self-signed certificate for testing purposes.

1. **Generate a private key**:
   ```bash
   openssl genrsa -out /etc/ssl/private/mydomain.key 2048
   ```

2. **Create a certificate signing request (CSR)**:
   ```bash
   openssl req -new -key /etc/ssl/private/mydomain.key -out /etc/ssl/certs/mydomain.csr
   ```

3. **Generate the self-signed certificate**:
   ```bash
   openssl x509 -req -days 365 -in /etc/ssl/certs/mydomain.csr -signkey /etc/ssl/private/mydomain.key -out /etc/ssl/certs/mydomain.crt
   ```

   - The generated certificate will be valid for one year (`-days 365`).
   - Store the `.crt` file and `.key` file in a secure location.

### Installing the SSL Certificate in HAProxy

1. Combine the private key and certificate into a single `.pem` file:
   ```bash
   cat /etc/ssl/private/mydomain.key /etc/ssl/certs/mydomain.crt > /etc/haproxy/certs/mydomain.pem
   ```

2. Edit your HAProxy configuration (`haproxy.cfg`) to include the SSL certificate:
   ```bash
   frontend https_front
       bind *:443 ssl crt /etc/haproxy/certs/mydomain.pem
       default_backend servers

   backend servers
       server webserver1 192.168.1.10:80 check
   ```

3. Restart HAProxy to apply the changes:
   ```bash
   sudo systemctl restart haproxy
   ```

### Redirecting HTTP to HTTPS

To force HTTP traffic to redirect to HTTPS:

1. Edit your HAProxy configuration (`haproxy.cfg`):
   ```bash
   frontend http_front
       bind *:80
       redirect scheme https if !{ ssl_fc }
   ```

2. Save the configuration and restart HAProxy:
   ```bash
   sudo systemctl restart haproxy
   ```

## How to Use

1. **Generate SSL Certificates**:
   - Follow the steps in the "SSL Certificate Setup" section to create a self-signed certificate or obtain a certificate from a trusted CA.

2. **Configure HAProxy SSL Termination**:
   - Refer to the `1-haproxy_ssl_termination/` directory for detailed instructions on setting up SSL termination in HAProxy.

3. **Set Up HTTP to HTTPS Redirection**:
   - Configure redirection by following the steps in the `2-redirect_http_to_https/` directory.
