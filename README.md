# Automating-Virtual-Machine-Backups-Replication-To-An-Amazon-S3-Bucket
 
## Task
*  Create a bash script that runs on a schedule to compress and backup critical data on a virtual machine (EC2 Instance) in AWS and  Automate the backup replication to an Amazon S3 Bucket
    ![image](https://github.com/emar137/Automate-Virtual-Machine-Data-Backup-Replication-To-An-Amazon-S3-Bucket/assets/84228720/313cca10-73f2-42bf-bef8-f49ad1a81ea4)
## Key Features:
* Backup Scheduling: Allow users to schedule backups at specific times or intervals (e.g., daily, weekly, or monthly)
* Backup Compression: Compress the backup files to save storage space using the tar command with gzip (tar -czf) or other compression methods.
* Logging: Create log files to record backup operations, including start time, end time, and any errors encountered during the backup.
* AWS Integration: Seamlessly integrate with the AWS Command Line  Interface (CLI) for secure and efficient uploads to Amazon S3.
 ## prerequisites
 * Bash scripting
 * Aws cli  
 * Ec2 instance(ubuntu)
 * S3 bucket
## Step 1
* Create an IAM role (s3-bash-role) with the Permission policy "AmazonS3FullAccess"
* Attach the IAM role to the EC2 instance
## Step 2
* connect on Ec2 instance(ubuntu) through ssh and install aws cli . https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zi
    ```
    ```
    sudo apt install unzip
    ```
    ```
    unzip awscliv2.zip
    ```
    ```
     sudo ./aws/install
    ```
* Use the following command to show where the package was installed
    ```
    which aws
    ```
* Run the following command to find out the AWS CLI version installed
    ```
    aws --version
    ```
## Step 3 
* Create an S3 bucket in AWS through cli
  ```
  aws s3api create-bucket --bucket  s3-backup-456
  ```
## Step 4
* Create the script 
     ```
        vim backup.sh  
     ```
     ```
    #!/bin/bash

    #Directory path that you want to backup
    Backup_file=/home/ubuntu/bash

    #Directory path that we store backup files on it
    Dest=/home/ubuntu/backup

    #Backup file name
    time=$(date +%m-%d-%y_%H_%M_%S)
    filename=file-backup-$time.tar.gz

    #Log file
    log_file=/home/ubuntu/backup/logfile.log

    #S3 bucket name
    S3_BUCKET="s3-backup-456"

    # checks if the AWS Command Line Interface (CLI) is installed on the system

    if ! command -v aws &> /dev/null; then
      echo "AWS CLI is not installed. Please install it first."
      exit 2
    fi
    if [ $? -ne 2 ]
            then
                    if [ -f $Dest/$filename ]
                            then
                                    echo " This file is already exit: error "| tee -a "$log_file "
                                    exit 2


                            else
                                    tar -czvf "$Dest/$filename"  "$Backup_file"
                                    echo
                                    echo "Backup completed successfully. Backup file: $filename " | tee -a "$log_file"
                                     echo
                                    # upload backup file on s3 bucket
                                    aws s3 cp "$Dest/$filename"  "s3://$S3_BUCKET/"
                    fi
    fi

    if [ $? -eq 0 ]
         then
              echo
              echo "File uploaded successfully to the S3 bucket: $S3_BUCKET"
         else
              echo "File upload to S3 failed."
    fi   
   ```
 * Add executable permission for the file
    ```
    chmod +x backup.sh  
    ```
  * Run the script (Note:I run the script manually but in the next step I will run it automatically)
    ``` 
    ./backup.sh
    ```
## Step 5
* I need to run a schedule by using root permissions so I will use crontab file.
  ```
  sudo vim /etc/crontab
  ```
* Add This entry to it.
  ```
  0 0 * * *  root  /home/ubuntu/backup-script.sh
  ```
* That entry will run the backup script automatically every day at midnight
## Step 6
* How to delete the schedule by removing this entry from crontab file
  ```
  0 0 * * *  root  /home/ubuntu/backup-script.sh
  ```
* To delete s3bucket on aws
   ```
    aws s3 rb s3://s3-backup-456 --force
