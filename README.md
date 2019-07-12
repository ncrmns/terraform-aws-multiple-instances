# Create multiple instances with  terraform
## 1. Install terraform
### https://www.terraform.io/downloads.html

## 2. Create a .tfvars file
### a) (terraform.tfvars) contain the desired server names in an array:

    server_names = ["ncrmns-dev", "ncrmns-staging", "ncrmns-production"]

## 3. Create a .tf file for terraform
### a) (main.tf) declear the server names variable:

    variable "server_names" {
    type = list
    }

### b) (main.tf) set the instance creator to a loop of 3:  

    resource "aws_instance" "ncrmns-terraform" {
    	ami           = "ami-026c8acd92718196b"
    	instance_type = "t2.micro"
    		key_name = "${var.keyname}"
    		count = 3
    		tags = {
        	Name = var.server_names[count.index]
    		}

#### At count set the number of instances we specified in the server_names variable. Note the iteration at the tags section

## 4. Create dependencies:
### a) (main.tf) we use docker to run our application, copy the docker install script to the instance:

    provisioner "file" {
        source = "dockerInstall.sh"
        destination = "/tmp/dockerInstall.sh"
        connection {
            type = "ssh"
            host = self.public_ip
            user = "ubuntu"
            private_key = file("${var.privatekey}")
        }
    }

#### For destination use the /tmp folder it doesnt need super user permissions to write. We have to specify the connection, for host we can use self.public_ip if we are in the same block as the instance creation. For user use the username specified by the instance. Use a variable to hide your rout for the .pem file.

### b) (main.tf) run the docker install script:
 
     provisioner "remote-exec" {
        inline = [
            "bash /tmp/dockerInstall.sh"
        ]
        connection {
            type = "ssh"
            host = self.public_ip
            user = "ubuntu"
            private_key = file("${var.privatekey}")
        }
    }

## 5. Deploy application
### a) (main.tf) pull our docker image and run it:

    provisioner "remote-exec" {
        inline = [
            "sudo docker pull ncrmns/helloworld11",
            "sudo docker run -d -p  8080:8080 ncrmns/helloworld11"
        ]
        connection {
            type = "ssh"
            host = self.public_ip
            user = "ubuntu"
            private_key = file("${var.privatekey}")
        }
    }

#### run docker in detached mode with the desired port forwarding 

