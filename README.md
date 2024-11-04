# python-server-with-db-pipeline

Here is the project link "https://github.com/Abubakar-Meigag/Contact-Manager"

# CI/CD Pipeline for Contact-Manager on EC2

This repository includes a GitHub Actions pipeline that automates the deployment of the Contact-Manager application to an EC2 instance. Upon a push to the `main` branch, the workflow will:

1. Check out the latest code.
2. Set up a Python environment.
3. Securely connect to an EC2 instance via SSH.
4. Transfer files to the EC2 instance.
5. Set up the environment variable for an external PostgreSQL database.
6. Restart the application services to reflect changes.

## Prerequisites

Before setting up the pipeline, make sure you have:

- An EC2 instance with the required software installed (`Python`, `pip`, `virtualenv`, `Gunicorn`, `Nginx`).
- SSH access to the EC2 instance.
- An external PostgreSQL database, such as ElephantSQL, with the connection string.

## Setting up GitHub Secrets

To keep sensitive information secure, store necessary credentials in GitHub Secrets. You’ll need to add the following secrets:

- `EC2_SSH_KEY`: Your SSH private key for connecting to the EC2 instance.
- `EC2_PUBLIC_IP`: The public IP address of your EC2 instance.
- `EXTERNAL_DATA_LINK`: The PostgreSQL connection string for your external database, in the format:
  ```
  postgres://username:password@hostname:port/dbname
  ```

## Workflow File Explanation

The pipeline is defined in the `.github/workflows/deploy.yml` file with the following steps:

### Workflow Overview

![Screenshot 2024-11-04 at 7 04 11 PM](https://github.com/user-attachments/assets/b7eae1ce-d412-4dc5-a1aa-ccbe4a4ad6da)

```yaml
name: Deploy to EC2
```

This sets the name of the workflow. It will be triggered on pushes to the `main` branch.

### Trigger

```yaml
on:
  push:
    branches:
      - main
```

The workflow is triggered whenever there is a push to the `main` branch.

### Jobs and Steps

The `deploy` job runs on an `ubuntu-latest` runner with the following steps:

1. **Checkout Code**:
   - Uses the `actions/checkout@v2` action to pull the latest code from the repository.

   ```yaml
   - name: Checkout code
     uses: actions/checkout@v2
   ```

2. **Set up Python Environment**:
   - Sets up Python 3.9 using the `actions/setup-python@v3` action.

   ```yaml
   - name: Set up Python
     uses: actions/setup-python@v3
     with:
       python-version: '3.9'
   ```

3. **Set up SSH**:
   - Configures the SSH agent to use the `EC2_SSH_KEY` secret for connecting to the EC2 instance.

   ```yaml
   - name: Setup SSH
     uses: webfactory/ssh-agent@v0.5.3
     with:
       ssh-private-key: ${{ secrets.EC2_SSH_KEY }}
   ```
   
![Screenshot 2024-11-04 at 6 48 39 PM](https://github.com/user-attachments/assets/c5ee102c-d5ac-47f6-b6a2-3065cfc6e50e)

4. **Copy Files to EC2**:
   - Uses `rsync` to securely transfer files from the GitHub repository to the EC2 instance, using the `EC2_PUBLIC_IP` secret.

   ```yaml
   - name: Copy files to EC2
     run: |
       rsync -avz -e "ssh -o StrictHostKeyChecking=no" ./ ec2-user@${{ secrets.EC2_PUBLIC_IP }}:/home/ec2-user/Contact-Manager
   ```

5. **SSH into EC2 and Deploy**:
   - Connects to the EC2 instance over SSH and performs the following actions:
     - Navigates to the app directory on the EC2 instance.
     - Activates the virtual environment.
     - Installs any new dependencies from `requirements.txt`.
     - Creates or updates the `.env` file with the `EXTERNAL_DATA_LINK` environment variable.
     - Restarts `Gunicorn` and `Nginx` to apply the latest code and configuration changes.
    
       
![Screenshot 2024-11-04 at 6 47 22 PM](https://github.com/user-attachments/assets/99f7382e-0058-492b-8735-884e792787d5)
![Screenshot 2024-11-04 at 6 48 12 PM](https://github.com/user-attachments/assets/9180fed5-9f33-4a50-b698-4d215bf68e8f)

   ```yaml
   - name: SSH into EC2 and deploy
     env:
       EXTERNAL_DATA_LINK: ${{ secrets.EXTERNAL_DATA_LINK }} 
     run: |
       ssh -o StrictHostKeyChecking=no ec2-user@${{ secrets.EC2_PUBLIC_IP }} << 'EOF'
         # Navigate to the app directory
         cd /home/ec2-user/Contact-Manager/server

         # Activate the virtual environment
         source venv/bin/activate

         # Install dependencies if there are any changes
         pip install -r requirements.txt

         # Create or update the .env file with the external data link
         echo "EXTERNAL_DATA_LINK=$EXTERNAL_DATA_LINK" > .env

         # Restart Gunicorn to apply the latest code changes
         sudo systemctl restart gunicorn
         sudo systemctl restart nginx
       EOF
   ```
![Screenshot 2024-11-04 at 6 45 49 PM](https://github.com/user-attachments/assets/111df3cd-b428-4ceb-8f64-2c49ba28d886)

   

## Environment Variable Integration

To connect the application to an external database (e.g., ElephantSQL), this pipeline:

- Uses the `EXTERNAL_DATA_LINK` secret to securely pass the PostgreSQL connection string.
- Updates the `.env` file on the EC2 instance with the connection string:
  ```bash
  echo "EXTERNAL_DATA_LINK=$EXTERNAL_DATA_LINK" > .env
  ```

![Screenshot 2024-11-04 at 6 56 15 PM](https://github.com/user-attachments/assets/b44e0556-0cfc-41ad-a4ac-86671d375e08)


  
- This setup allows the application to access the database URL via the environment variable `EXTERNAL_DATA_LINK`, which is used in the Flask application to connect to the database.

## Verifying the Deployment

1. Once the workflow runs successfully, visit your application’s endpoint on your EC2 instance’s public IP or domain.
2. Check if the application can access and display data from the external database.

![Screenshot 2024-11-04 at 7 02 04 PM](https://github.com/user-attachments/assets/2c280050-fe32-47ad-85f6-f50ba321c462)
![Screenshot 2024-11-04 at 6 49 41 PM](https://github.com/user-attachments/assets/5dd6180b-ad12-459b-86f0-cac3ca64c6fa)


This pipeline provides an automated deployment process for a Flask application on EC2, with secure handling of environment variables and seamless integration with an external PostgreSQL database.
Here is the project link "https://github.com/Abubakar-Meigag/Contact-Manager"
