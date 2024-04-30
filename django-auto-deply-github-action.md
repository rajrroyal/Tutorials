
### Automate Django Deployment using Github Action
- On Your Local Machine, Open Your Project using VS Code or any Editor
- Create A Folder named .scripts inside your root project folder e.g. elitesync/.scripts
- Inside .scripts folder Create A file with .sh extension e.g. elitesync/.scripts/deploy.sh
- Write the below script inside the created .sh file
```sh
#!/bin/bash
set -e

echo "Deployment started ..."

# Pull the latest version of the app
git pull origin master
echo "New changes copied to server !"

# Activate Virtual Env
source ../venv/bin/activate
echo "Virtual env 'venv' Activated !"

echo "Installing Dependencies..."
pip install -r requirements.txt --no-input

echo "Serving Static Files..."
python manage.py collectstatic --noinput

echo "Running Database migration"
python manage.py makemigrations
python manage.py migrate

# Deactivate Virtual Env
deactivate
echo "Virtual env 'venv' Deactivated !"

# Reloading Application So New Changes could reflect on website
pushd elitesync
touch wsgi.py
popd

echo "Deployment Finished!"
```
- Go inside the .scripts Folder then Set File Permission for .sh File
```sh
git update-index --add --chmod=+x deploy.sh
```
- Create a Directory Path named .github/workflows inside your root project folder e.g. elitesync/.github/workflows
- Inside the workflows folder Create A file with a .yml extension e.g. elitesync/.github/workflows/deploy.yml
- Write the below script inside the created .yml file
```sh
name: Deploy

# Trigger the workflow on push and
# pull request events on the master branch
on:
  push:
    branches: ["master"]
  pull_request:
    branches: ["master"]

# Authenticate to the server via SSH
# and run our deployment script
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          port: ${{ secrets.PORT }}
          key: ${{ secrets.SSH_KEY }}
          script: "cd /var/www/html/elitesync/project && ./.scripts/deploy.sh"
```
- Go to Your Github Repo Click on Settings
- Click on Secrets and Variables from the Sidebar then choose Actions
- On the Secret Tab, Click on New Repository Secret
- Add Four Secrets HOST, PORT, USERNAME, and SSHKEY as below
```sh
Name: HOST
Secret: Your_Server_IP
```
```sh
Name: PORT
Secret: Your_Server_PORT
```
```sh
Name: USERNAME
Secret: Your_Server_User_Name
```
- You can get the Server User Name by logging into your server via ssh and then run the below command
```sh
whoami
```
- Generate SSH Key for Github Action by Login into the Remote Server then run the below Command OR You can use the old SSH Key But I am creating a New one for GitHub Action
```sh
Syntax:- ssh-keygen -f key_path -t ed25519 -C "your_email@example.com"
Example:- ssh-keygen -f /home/ubuntu/.ssh/gitaction_ed25519 -t ed25519 -C "gitactionautodep"
```
- Open Newly Created Public SSH Keys then copy the key
```sh
cat ~/.ssh/gitaction_ed25519.pub
```
- Open the authorized_keys File which is inside .ssh/authroized_keys then paste the copied key in a new line
```sh
cd .ssh
nano authorized_keys
```
- Open Newly Created Private SSH Keys then copy the key, we will use this key to add a New Repository Secret On the GitHub Repo
```sh
cat ~/.ssh/gitaction_ed25519
```
```sh
Name: SSH_KEY
Secret: Private_SSH_KEY_Generated_On_Server
```
- Commit and Push the change to Your Github Repo
- Get Access to the Remote Server via SSH
```sh
Syntax:- ssh -p PORT USERNAME@HOSTIP
```
- Go to Your Project Directory
```sh
Syntax:- cd project_path
Example:- cd /var/www/html/elitesync/project
```
- Pull the changes from GitHub just once this time.
```sh
git pull
```
- Your Deployment should become automated.
- On Local Machine make some changes in Your Project then Commit and Push to Github Repo It will automatically be deployed on Live Server
- You can track your action from GitHub Actions Tab
- If you get any File Permission error in the action then you have to change file permission accordingly.
- All Done
