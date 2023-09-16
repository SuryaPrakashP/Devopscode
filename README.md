# Devopscode
Codejudge devops internship Code
provider "aws" {
  region = "us-east-1"  # Update with your desired region
}

resource "aws_instance" "wordpress_vm" {
  ami           = "ami-0c55b159cbfafe1f0"  # Use an appropriate AMI ID
  instance_type = "t2.micro"                # Choose an instance type
  subnet_id     = "subnet-xxxxxxxxxxxx"    # Specify your public subnet ID
  key_name      = "your-key-pair-name"     # Add your SSH key pair name
  security_groups = ["your-security-group"] # Add your security group name
}

output "wordpress_vm_ip" {
  value = aws_instance.wordpress_vm.public_ip
}

# Use an official PHP runtime as a parent image
FROM php:7.4-apache

# Set environment variables for WordPress database connection
ARG DB_HOST
ARG DB_NAME
ARG DB_USER
ARG DB_PASSWORD

# Install necessary PHP extensions and WordPress dependencies
RUN docker-php-ext-install mysqli
RUN a2enmod rewrite

# Copy WordPress files to the container
COPY wordpress /var/www/html

# Set up WordPress configuration
RUN cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
RUN sed -i "s/database_name_here/${DB_NAME}/" /var/www/html/wp-config.php
RUN sed -i "s/username_here/${DB_USER}/" /var/www/html/wp-config.php
RUN sed -i "s/password_here/${DB_PASSWORD}/" /var/www/html/wp-config.php
RUN sed -i "s/localhost/${DB_HOST}/" /var/www/html/wp-config.php

# Expose port 80
EXPOSE 80

# Start the Apache web server
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

docker build -t your-dockerhub-username/wordpress .
docker push your-dockerhub-username/wordpress
