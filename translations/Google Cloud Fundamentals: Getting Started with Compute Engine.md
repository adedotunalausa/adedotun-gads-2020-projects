# Google Cloud Fundamentals: Getting Started with Compute Engine

## Objectives:

In this lab, you will learn how to perform the following tasks:

  - Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.

  - Create a Compute Engine virtual machine using the gcloud command-line interface.

  - Connect between the two instances.

## Steps:

1. Create a virtual machine (VM) using the GCP (Google Cloud Platform) Console

    - Create the VM instance:

        gcloud compute instances create my-vm-1 --zone=us-central1-a --machine-type=n1-standard-1 --image-project=debian-cloud --image=debian-9-stretch-v20200902 --subnet "default" --tags=http-server

    - Create firewall rules to target the tag from above action and allow HTTP traffic:

        gcloud compute firewall-rules create default-allow-http --action=ALLOW --direction=INGRESS --network=default --rules=http:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

2. Create VM using the glcoud command line interface

    - Display the list of the zones in the us-central1 region: 

        gcloud compute zones list | grep us-central1

    - Set default zone to us-central1-f:

        gcloud config set compute/zone us-central1-f
    
    - Create a VM instance called my-vm-2 in the default zone set above:

        gcloud compute instances create "my-vm-2" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"

3. Connect between the two VM instances created above

    - Connect to my-vm-2 using the secure shell: 

        gcloud compute ssh my-vm-2

    - Confirm that my-vm-2 can reach my-vm-1 over the network:

        ping -c 4 my-vm-1

    - Open a command promt on my-vm-1 from my-vm-2 using the ssh comman:

        ssh my-vm-1 

    - Install Nginx web server on my-vm-1: 

        sudo apt-get install nginx-light -y
    
    - Add a custom message to the home page of the web server using the nano text editor:

        sudo nano /var/www/html/index.nginx-debian.html

    - Add text to the line just below the h1 header by using the arrow keys to move the cursor that line:

        Hi from adedotun

    - Save changes made and exit the text editor by pressing Ctrl+O and then pressing Enter to save the edited file, and then pressing Ctrl+X.

    - Confirm that the web server is serving the new page by executing this command at the command prompt on my-vm-1:

        curl http://localhost/

        The response will be the HTML source of the web server's home page, including the line of custom text.

    - Execute this command to exit the command prompt on my-vm-1 and return to the command prompt on my-vm-2: 

        exit 

    - Execute this command at the command prompt on my-vm-2 to confirm that my-vm-2 can reach the web server on my-vm-1: 

        curl http://my-vm-1/

        The response will again be the HTML source of the web server's home page, including your line of custom text.

    - Execute this command to exit the command prompt on my-vm-2: 

        exit

    - Execute this command to get the external IP address for my-vm-1: 

        gcloud compute instances list --zone us-central1-a 

    - Copy the External IP address for my-vm-1 and paste it into the address bar of a new browser tab:

        The response will be the web server's home page, including the added custom text
