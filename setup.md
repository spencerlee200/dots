# Setting up a Ubuntu 16.04 server to deploy multiple static sites

If you are looking for an easy way to deploy your static sites live look no further than this guide. Deploying multiple static sites to the same server on Digital Ocean is a very easy, although admittedly tedious process. In this guide I will be explaining how to set up your initial server and how to set up an initial subdomain. You will be able to create and deploy to as many subdomains as you want by the end of this tutorial. 

## Initial Server Setup
### Setting up a virtual server with Digital Ocean
Once you've logged in or created a new account with [Digital Ocean]('https://www.digitalocean.com/') spinning up a virtual server (otherwise known as a droplet) is super easy. Navigate to the droplets page in the top navigation and hit the create droplet button. For this set up we are going to be using the **Ubuntu 16.04.2 x64** image. Select it from the dropdown menu under Ubuntu.

![Digital Ocean Images](http://i.imgur.com/vha5P8m.png)

Now you are going to have to select a payment plan. For this project the 5 dollar a month option should be plenty. You can always go in and change this as your application scales up.

![Digital Ocean Plans](http://i.imgur.com/lXhXhlp.png)

Next you are going to be prompted to select a datacenter. You should go with the one that is closest to you and your target audience. You may also notice that each datacenter has a number associated with it. The higher the number the newer the datacenter is.

![Digital Ocean Datacenters](http://i.imgur.com/l4QD4ks.png)

That's really all there is to it. Digital Ocean provides you with a couple more options such as private networking and adding SSH keys but we will handle all of that stuff as we come to it. Just give it a hostname, hit that big green create button and move on to the next step.

![Digital Ocean Options](http://i.imgur.com/ADtkDBx.png)

### Configuring the new server
Now its time for the fun part. Start by opening up a new terminal window. First things first we need to SSH into our server. To do this copy the command below and replace `<your server IP address>` with the IP address from your droplet on Digital Ocean.

```shell
ssh root@<your server IP address>
```

Since this is your first time connecting you will probably get a warning about the authenticity of the host. Just type in yes and hit enter one more time. Here you should be prompted for a password. Digital Ocean should of provided you with this password in an email. Copy in your password and hit enter once again to finally connect to your server. You should see a screen similiar to this one: 

![Root ssh](http://i.imgur.com/OITGzLK.png)

By default Ubuntu requires you to immediately change the password for your root user for security reasons. To change it simply enter your old password again and enter your new password following the onscreen prompts. Once you've done that you should see something line `root@wordpress:~#` on a new line. Now we can start editing things.

#### Adding a new user
Because of the root users extremely broad privileges, working in root opens up the potential for a lot of destructive changes to occur. To mend this we are going to go ahead and set up a new user. Copy this line into terminal replacing `<name>` with the desired name for your new user and hit enter. Then follow the onscreen prompts to set that users password and fill in the prompts with any additional information you need your user to have.

```shell
adduser <name>
```

Next, we need to give your new user regular account privileges. Without this step your new user wouldn't be very useful. To do this simply enter the line below into terminal, once again replacing `<name>` with your new user.This will give your user the power to use sudo commands by adding it to the sudo group.

```shell 
usermod -aG sudo <name>
```

Finally to switch to your newly created user enter this line into your terminal subbing in your own user name.

```shell 
su - <name>
```

#### Disabling root access (optional)
Now that we are on our new users account we can go ahead and create an extra security feature by configuring our SSH daemon to not allow remote SSH access to our server. Luckily this is pretty easy to do and can be done in just a few commands. First access the config file:

```shell
sudo nano /etc/ssh/sshd_config
``` 

Next look for a line that says `PermitRootLogin yes` and modify it to say `PermitRootLogin no` then exit and save the file (`CTRL+X`, then `Y`, then `ENTER`). Finally reload SSH with the command below to finalize these changes.

```shell
sudo service ssh restart
```

#### Adding public key authentication (optional)
This step is another optional security step but itll make it quicker and easier for you to connect to your sever. If you were to disable passwords then only users with your key could get in. For now though we are just going to make it easier for our current user to connect. Open a new terminal window and enter this line: 

```shell
ssh-keygen
```

You can set a passphrase to secure the key with or just leave it blank. Hit enter through the prompts until the keys are generated. Remember that if you do set a passphrase you will need both the passphrase and password to SSH into your server. Next you need to install the public key to your user. To do this enter this line in the same terminal window you generated the keys in replacing `<name>` with your users name and `<your server IP address>` with your servers IP just like you would when connecting. You will have to enter your users password again at the end of the prompt.

```shell
ssh-copy-id <name>@<your server IP address> 
```

### Installing Nginx
Just in case we have any packages in need of update we should run an update before installing nginx. Make sure you are logged into your server as your user and enter this line into terminal. You'll have to confirm that you want to allocate disk space for the install. 

```shell
sudo apt-get update && sudo apt-get install nginx
```

One this command finishes running you can check if it worked by navigating to your servers public IP in the browser. You should see the basic nginx install screen.

![nginx install screen](http://i.imgur.com/gniYuwZ.png)

## Adding Projects

For each project you want to deploy you are going to have to set up an additional subdomain on your server.

### Creating a subdomain

First we need to move into our working directory and create a directory that our subdomain will look to find our code. You can do this by entering the lines below replacing `<project name>` with whatever project you are trying to deploy to the site.

```shell
cd /var/www/html
sudo mkdir <project name>
```

Now we have to configure nginx to send certain URLS to that subomain. First we need to access the default sites-enabled page created by nginx so that we can edit the server blocks.

```shell
sudo nano /etc/nginx/sites-enabled/default
```

Assuming that you want to leave your default IP as a landing page all you have to do is add a server block below the one that already exists following the template below. 


```shell
server {
        listen 80;
        listen [::]:80;
        root /var/www/html/<your project name/;
        index index.html index.htm index.nginx-debian.html;
        server_name <your project name>.<your server ip>.xip.io;
        location / {
                try_files $uri $uri/ =404;
        }
}
```

Next just refresh nginx with the command below:

```shell
sudo systemctl reload nginx
```

And thats all you have to do to add a subdomain. If you wanted to edit the landing page all you need to do it edit the default nginx page with `sudo nano /var/www/html/index.nginx-debian.html` or you could just create your own with `sudo nano /var/www/html/index.html` and delete the old one with `sudo rm /var/www/index.nginx-debian.html`.

## Deploying your project
Now that you have your server set up I am sure you are excited to push your website live. To do that follow my deployment guide [here](https://github.com/spencerlee200/static-sites-pipeline). 



