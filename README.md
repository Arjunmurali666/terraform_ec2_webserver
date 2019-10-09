# terraform_ec2_webserver

## Terraform - This is a terraform code written to create a webserver in aws. The newly created ec2 will be ready serve HTTP requests since HTTP will be installed on it at the creation time.

```terraform
provider "aws" {
    access_key = [your_key_here]
    secret_key = [your_key_here]
    region = "us-east-2"
}

# Create web server
resource "aws_instance" "web_server" {
    ami = "ami-0d8f6eb4f641ef691"
    vpc_security_group_ids = ["default"]
    instance_type = "t2.micro"
    key_name      = "ansible"
    tags = {
        Name = "web_server"
    }

  connection {
    user         = "ec2-user"
    host         = "${self.public_ip}"
    private_key  = "${file("~/ansible.pem")}"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum install httpd -y",
      "sudo service httpd start",
      "sudo yum install php -y",
      "sudo chkconfig httpd on",
      "sudo touch /var/www/html/index.php",
      "sudo chmod 777 /var/www/html/index.php"
    ]
  }

  provisioner "file" {
    source = "healthcheck.html"
    destination = "/var/www/html/index.php"
  }

  provisioner "remote-exec" {
    inline = [
      "sudo chmod 644 /var/www/html/index.php",
      "sudo service httpd restart"
    ]
  }

  # Save the public IP for testing
  provisioner "local-exec" {
    command = "echo ${aws_instance.web_server.public_ip} > public-ip.txt"
  }
}
```
