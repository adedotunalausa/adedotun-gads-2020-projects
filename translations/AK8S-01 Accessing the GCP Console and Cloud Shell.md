# AK8S-01 Accessing the GCP Console and Cloud Shell

# Objectives:

In this lab, you learn how to perform the following tasks:

  - Learn how to access the GCP Console and Cloud Shell

  - Become familiar with the GCP Console

  - Become familiar with Cloud Shell features, including the Cloud Shell code editor

  - Use the GCP Console and Cloud Shell to create buckets and VMs and service accounts

  - Perform other commands in Cloud Shell

## Steps:

1. Explore the GCP Console

    - Verify that your project is selected:

        - Login to your the gcp from the command line interface: 

            gcloud auth login 

            After authentication is successful, the current project will be displayed.

        - List all projects:

            gcloud projects list

        - Copy the [desired_project_id], paste it somewhere because it'll still be used in creating a storage bucket and execute this command:

            gcloud config set project [desired_project_id]

    - Navigate to Google Cloud Storage and create a bucket:

        - Create a bucket:

            gsutil mb gs://[desired_project_id]/

        - Create a virtual machine instance:

            - Create the instance:
            
                gcloud compute instances create first-vm --zone=us-central1-c --machine-type=f1-micro --image-project=debian-cloud --image=debian-9-stretch-v20200902 --subnet "default" --tags=http-server

            - Explore the VM details:

                gcloud compute instances describe first-vm

        - Create an IAM service account:

            gcloud iam service-accounts create test-service-account --display-name "test-service-account"

            - Specify role as project editor and create key:

                - Execute the following command and copy the email in the email column corresponding to the service account created above:

                    gcloud iam service-accounts list

                - Execute the following command by replacing [copied_email] with the email copied from above:

                    gcloud iam service-accounts add-iam-policy-binding [copied_email] --member="serviceAccount:[copied_email]" --role="roles/editor"

                    gcloud iam service-accounts keys create ~/key.json --iam-account [copied_email]

                    The output will show the location of the key file downloaded on your local computer. In a later step, you'll find this key file and upload it to the VM

2. Explore Cloud Shell

    - Use the following commands to define the environment variables used in this task:

        Note: a. Replace [BUCKET_NAME] with the name of the first bucket from task 1.
              b. Replace [BUCKET_NAME_2] with a globally unique name. You can append a 2 to the globally unique bucket name that you used previously.

        MY_BUCKET_NAME_1=[BUCKET_NAME]
        MY_BUCKET_NAME_2=[BUCKET_NAME_2]
        MY_REGION=us-central1

    - Create a second Cloud Storage bucket and verify it in the GCP Console:

        gsutil mb gs://$MY_BUCKET_NAME_2

    - Use the gcloud command line to create a second virtual machine:

        - Execute the following command to list all the zones in a given region:

            gcloud compute zones list | grep $MY_REGION

        - Select a zone from the first column of the list. Notice that GCP zones' names consist of their region name, followed by a hyphen and a letter. You may choose a zone that is the same as or different from the zone that you used for the first VM in task 1.

        - Execute the following command to store your chosen zone in an environment variable and replace [ZONE] with your selected zone:

            MY_ZONE=[ZONE]

        - Set this zone to be your default zone by executing the following command:

            gcloud config set compute/zone $MY_ZONE

        - Execute the following command to store a name in an environment variable you will use to create a VM. You will call your second VM second-vm:

            MY_VMNAME=second-vm

        - Create a VM in the default zone that you set earlier in this task using the new environment variable to assign the VM name:

            gcloud compute instances create $MY_VMNAME --machine-type "n1-standard-1" --image-project "debian-cloud" --image-family "debian-9" --subnet "default"

        - List the virtual machine instances in your project:

            gcloud compute instances list

            Note: Copy the external IP address of the first VM created and paste it in a browser. Your browser will present a Connection refused message in a new browser tab. This message occurs because, although there is a firewall port open for HTTP traffic to your VM, no Web server is running there. Close the browser tab you just created.
                    
    - Use the gcloud command line to create a second service account:

        - Execute the following command to create a new service account:

            gcloud iam service-accounts create test-service-account2 --display-name "test-service-account2"

        - Grant the second service account the Project viewer role:

            - Execute the following command and copy the email in the email column corresponding to the second service account created above:

                gcloud iam service-accounts list

            - Execute the following command by replacing [copied_email] with the email copied from above:

                gcloud iam service-accounts add-iam-policy-binding [copied_email] --member="serviceAccount:[copied_email]" --role="roles/viewer"

3. Work with Cloud Storage in Cloud Shell

    - Download a file to Cloud Shell and copy it to Cloud Storage:

        - Copy a picture of a cat from a Google-provided Cloud Storage bucket to your local computer:

            gsutil cp gs://cloud-training/ak8s/cat.jpg cat.jpg

        - Copy the file into one of the buckets that you created earlier:

            gsutil cp cat.jpg gs://$MY_BUCKET_NAME_1

        - Copy the file from the first bucket into the second bucket:

            gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg gs://$MY_BUCKET_NAME_2/cat.jpg

    - Set the access control list for a Cloud Storage object:

        - To get the default access list that's been assigned to cat.jpg (when you uploaded it to your Cloud Storage bucket), execute the following two commands:

            gsutil acl get gs://$MY_BUCKET_NAME_1/cat.jpg  > acl.txt 
            cat acl.txt

        - To change the object to have private access, execute the following command:

            gsutil acl set private gs://$MY_BUCKET_NAME_1/cat.jpg

        - To verify the new ACL that's been assigned to cat.jpg, execute the following two commands:

            gsutil acl get gs://$MY_BUCKET_NAME_1/cat.jpg  > acl-2.txt
            cat acl-2.txt

    - Authenticate as a service account in Cloud Shell

        - Execute the following command to view the current configuration:

            gcloud config list

        - Execute the following command to change the authenticated user to the first service account (which you created in an earlier task) through the credentials that you downloaded to your local machine and then uploaded into Cloud Shell (credentials.json).

            gcloud auth activate-service-account --key-file credentials.json

        - To verify the active account, execute the following command:

            gcloud config list

        - To verify the list of authorized accounts in Cloud Shell, execute the following command:

            gcloud auth list

        - To verify that the current account (test-service-account) cannot access the cat.jpg file in the first bucket that you created, execute the following command:

            gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg ./cat-copy.jpg

                Because you restricted access to this file to the owner earlier in this task you should see output that looks like the following example.

                Output: Copying gs://test-bucket-123/cat.jpg...
                        AccessDeniedException: 403  KiB]

        - Verify that the current account (test-service-account) can access the cat.jpg file in the second bucket that you created:

            gsutil cp gs://$MY_BUCKET_NAME_2/cat.jpg ./cat-copy.jpg

                Because access has not been restricted to this file you should see output that looks like the following example.

                Output: Copying gs://test-bucket-123/cat.jpg...
                        - [1 files][ 81.7 KiB/ 81.7 KiB]
                        Operation completed over 1 objects/81.7 KiB.

        - To switch to the lab account, execute the following command and replace [USERNAME] with your email address used authenticating the sdk:

            gcloud config set account [USERNAME]

        - To verify that you can access the cat.jpg file in the [BUCKET_NAME] bucket (the first bucket that you created), execute the following command:

            gsutil cp gs://$MY_BUCKET_NAME_1/cat.jpg ./copy2-of-cat.jpg

                You should see output that looks like the following example. Your email account created the bucket and object and remained an Owner when the object access control list (ACL) was converted to private, so the lab account can still access the object.

                Output: Copying gs://test-bucket-123/cat.jpg...
                        - [1 files][ 81.7 KiB/ 81.7 KiB]
                        Operation completed over 1 objects/81.7 KiB.

        - Make the first Cloud Storage bucket readable by everyone, including unauthenticated users by running the following commands:

            gsutil iam ch allUsers:objectViewer gs://$MY_BUCKET_NAME_1

        - To get the public link of the cat.jpg file

            gsutil ls gs://$MY_BUCKET_NAME_1/ | sed 's|gs://|https://storage.googleapis.com/|'

        - Open an incognito browser tab and paste the link into its address bar. You will see a picture of a cat. Leave this browser tab open.

4. Explore the Cloud Shell code editor

    - Execute the following command to clone a git repository to your local computer:

        git clone https://github.com/googlecodelabs/orchestrate-with-kubernetes.git

    - Execute the following command to create a test directory: 

        mkdir test

            The test folder now appears in your local computer.

    - Change directory into the location of the cleanup.sh file and edit it with the nano text editor using the following commands:

        cd orchestrate-with-kubernetes
        nano cleanup.sh  

    - Use the arrow keys tp navigate and add the following text as the last line of the cleanup.sh file:

        echo Finished cleanup!

    - Press [Ctrl+O] and [Enter] to save the changes and [Ctrl+X] to exit the editor.

    - Execute the following commands to display the contents of cleanup.sh:

        cat cleanup.sh

    - Verify that the output of cat cleanup.sh includes the line of text that you added.

    - Create a new HTML file named "index.html" by executing the following commands:

        touch index.html

    - Edit the index.html file with the nano editor using the following commands:

        nano index.html

    - Paste this HTML text into the text editor: 

        <html><head><title>Cat</title></head>
        <body>
        <h1>Cat</h1>
        <img src="REPLACE_WITH_CAT_URL">
        </body></html>

    - Replace the string "REPLACE_WITH_CAT_URL" with the URL of the cat image from an earlier task. The URL will look like this:

        Example: https://storage.googleapis.com/qwiklabs-gcp-1aeffbc5d0acb416/cat.jpg

    - Connect to first-vm using the secure shell: 

        gcloud compute ssh first-vm

    - Execute the following commands install the nginx Web server on the first-vm:

        sudo apt-get update
        sudo apt-get install nginx

    - Open a new terminal tab while leaving the initial one open and still connected to the first-vm

    - Copy the HTML file you edited using the nano Editor to your virtual machine:

        cd orchestrate-with-kubernetes
        gcloud compute scp index.html first-vm:index.nginx-debian.html --zone=us-central1-c

    - In the initial terminal where you were connected to the first-vm, copy the HTML file from your home directory to the document root of the nginx Web server:

        sudo cp index.nginx-debian.html /var/www/html

    - Execute the following commands to list virtual machine instances:

        gcloud compute instances list

    - Copy the external IP address of the first-vm and paste it in a web browser:

        A web page that contains the cat image will show up.