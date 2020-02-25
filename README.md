## Basic Data Collection Using AWS
### Why?
Several participants in the UCLA-ITS Data Camp and/or monthly hack nights have been curious about using cloud services to repeatedly gather data over longer intervals of time. For example, one could query LA Metro's real-time bus location API to get bus locations every couple minutes for an entire week. 

Running such an application on a cloud computing platform offers the advantage of not requiring your personal computer to be running and online for the entire collection period, in addition to providing a platform for potential ongoing analysis. 

It's not especially hard to do so, and familiar environments such as Jupyter Notebooks and Anaconda work just fine. There are however several steps that aren't necessarily intuitive to a beginner. This guide is designed to outline those steps on the Amazon Web Services EC2 platform. 

### Making an AWS Account

First, make an AWS account through Amazon [here](https://aws.amazon.com/). 

UCLA students and faculty should also sign up for an AWS Educate account [here](https://idre.ucla.edu/signing-up-for-a-ucla-aws-educate-account) in order to recieve free AWS credits as well as other resources. 

Simple applications without large compute or storage requirements can be run using the AWS Free Tier, so depending on your needs it's quite possible you can use these services without paying anything, at least for a while. Details [here](https://aws.amazon.com/free/).

### Creating an EC2 Instance with Anaconda and Jupyter

#### Subscribing to Anaconda on AWS

AWS Marketplace offers subscriptions to a wide variety of free and paid software that can then be used in EC2 instances. 

Access the marketplace [here](https://aws.amazon.com/marketplace) and search for Anaconda. Choose "Anaconda with Python 3" and follow the steps to subscribe. 

#### Configuring your EC2 Instance

After subscribing, you should see an option to "Continue to Configuration" on the AWS Marketplace page. It should default to the latest version of Anaconda, select an appropriate AWS region and select "Continue to Launch" 

1. Under "EC2 Instance Type", select t2.micro to stay within the free tier. Other instance types are more powerful but will incur charges. 
1. Under "Key Pair Settings", follow the link to create a key pair in the EC2 console, then select that key. Be sure to keep the resulting .pem file somewhere safe, it cannot be redownloaded and losing it will lock you out of your instance. It's possible to regain access via a lengthy process, more details via AWS User Guide [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html). 
1. As described in the user guide link, Mac/Linux users must change the permissions of the key file using this command: ``` chmod 400 key_file_name.pem ```

1.    Using Terminal (or whatever shell you've got on Linux), first navigate to the directory where you've stored the .pem file then run the command. Alternatively, run the command specifing the filepath, for example ```chmod 400 ~/.ssh/my_key.pem```. 
    
   * *If you're unfamiliar with the bash command line, consider a basic tutorial like [this one](https://programminghistorian.org/en/lessons/intro-to-bash). Some level of familiarity will be necessary to configure our instance after we get it started.* 
1. Press Launch! 

#### Connecting to your EC2 Instance

Now that your instance is running, let's connect to it using SSH. Mac/Linux users should have an ssh client already installed and accessible via the Terminal ```ssh``` command, Windows users should install an appropriate ssh client. 

1. After launching, you should see a link to view the instance on the EC2 Console. You can also access the console through your AWS account.
2. You should see something like this: 
![alt text](https://raw.githubusercontent.com/ucla-its/basic-aws/master/aws-screenshot.png "AWS Console Screenshot")
3. Right-click on your instance and select "connect". The resulting window includes connection instructions as well as the option to use a browser-based client if you prefer.
 
    * Copy and run the example command, making sure to put the path to your .pem file inside the quotes and changing "root" to "ec2-user"
    * for example, ```ssh -i "~/.ssh/my_key.pem" ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com```
    * A prompt will appear saying the authenticity of the host cannot be established, type yes to continue connecting
1. You should be connected, with something like this appearing in Terminal or your ssh client:
	
	
```
(ox) Erics-MacBook-Air:.ssh edasmalchi$ ssh -i "~/.ssh/my_key.pem" ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|
https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
10 package(s) needed for security, out of 17 available
Run "sudo yum update" to apply all updates.
(base) [ec2-user@ip-172-31-8-129 ~]$
``` 

1. Go ahead and run ```sudo yum update``` to update software on your EC2 instance. Remember that with the ssh connection open, any command you run in this window will run on the EC2 instance, not your local computer. 

#### Configuring Anaconda

Now that we're connected to our EC2 instance, we can configure our Anaconda environment just as we would on our local computer. 

* For example, if you are interested in spatial analysis you may be interested in osmnx. To install, follow Geoff Boeing's [instructions](https://github.com/gboeing/osmnx) for installing osmnx and its dependencies (geopandas, etc) in a new environment called "ox". 

	1. Run these commands to install osmnx:

		```
		conda config --prepend channels conda-forge
		conda create -n ox --strict-channel-priority osmnx
		```
	2. 	Run ```conda activate ox``` to activate the environment.
	3. 	Run ```conda install jupyter``` to install Jupyter Notebooks

* Note that larger packages such as osmnx can take up a lot of disk space on the free t2.micro instances, so if you intend to collect and save a lot of files and don't need such advanced tools it's best to only install the packages you need.

#### SSH Port Forwarding

Since any notebook we start on our EC2 instance will be running there and not on our local computer, it won't automatically open in our web browser. We need to redirect one of our local TCP ports to point to the EC2 instance instead, allowing us to open remotely running notebooks in our browser. We'll use Amazon's [instructions](https://docs.aws.amazon.com/dlami/latest/devguide/setup-jupyter-configure-client.html) as a guide, slightly modifying the port number.

Since local Jupyter Notebooks default to port 8888 and you may have a reason to access both local notebooks and notebooks running on EC2 at the same time, let's forward port 8889 instead to avoid conflicts. 

1. Open a new, **local** terminal window (don't run this command in your EC2 instance)
1. Follow Amazon's instructions to complete the command, but replace 8888 with 8889. Also change the username to "ec2-user" and add the correct region to the domain name at the end. Refer back to the command you used to establish the ssh session. For example:
	```
	ssh -i ~/.ssh/my_key.pem -N -f -L 8889:localhost:8889 ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com
	```

#### Changing Jupyter Default Port

Since we just forwarded port 8889 instead of 8888, we'll need to tell the version of Jupyter running on our EC2 instance to use that port instead. Let's use these [instructions](https://stackoverflow.com/questions/40373330/change-jupyter-notebooks-localhost8888-default-server-with-other/40436673) on stackoverflow as a guide. 

1. On your EC2 instance now, run this command to generate the Jupyter configuration file:
	```
	jupyter notebook --generate-config
	```

	##### A Quick Aside About Console Text Editors
	
	We'll soon have to edit the configuration file to change the Jupyter default port. Since our only way to interact with our EC2 instance is  through our ssh console, we need a text editor that runs within the console. Vim is a powerful, if not immediately user-friendly, editor that comes pre-installed on most Mac/Linux machines as well as our EC2 instance.
	
	Take a few minutes to learn the basics using a guide like [this one](https://www.howtoforge.com/vim-basics), or better yet just run ```vimtutor```,  preferably in a local terminal (though it's also available through your EC2 instance). 

1. On your EC2 instance, open the configuration file for editing by running:
	```
	vim /home/ec2-user/.jupyter/jupyter_notebook_config.py
	```
2. Find the line:

	```
	# c.NotebookApp.port = 8888
	```
	(Vim's search function is covered in vimtutor, or look it up) Removing the comment marker and changing the port number, change it to:
	
		
		c.NotebookApp.port = 8889
3. Save the file and exit Vim using ```:wq``` (make sure to hit esc first so you're out of insert mode)

#### Optional: Change Time Zone

By default, EC2 instances use UTC time. If you plan on using Python modules like ```datetime``` in your code, you may prefer to have the EC2 instance use your local time.

1. Now that we know how to edit text files, follow Amazon's [instructions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/set-time.html#change_time_zone) to set the time by editing ```/etc/sysconfig/clock```

	```
	sudo vim /etc/sysconfig/clock
	```
Simply edit the first line to read ```ZONE="America/Los_Angeles"```, or another time zone of your choosing. Again save and exit Vim with ```:wq```.

2. Create a symbolic link per Amazon instructions:
	```
	sudo ln -sf /usr/share/zoneinfo/America/Los_Angeles /etc/localtime
	```
3. Reboot the EC2 instance: ```sudo reboot``` (you'll then have to reconnect via ssh)

#### Starting and Accessing a Remote Jupyter Notebook

We should now be ready to run and interact with a notebook on our EC2 instance! 

1. Making sure your preferred environment such as "ox" is active, run the ```jupyter notebook``` on your EC2 instance.
2. In that window, Jupyter will output a message that a web browser cannot be found. Don't panic and simply copy the given address starting with ```http://localhost``` to your web browser of choice, as long as the port forwarding and default port change were successful it should open right up!
3. Pat yourself on the back, secure in the knowledge that your notebook is running on a huge Amazon server somewhere. 
4. By default, Jupyter's file browser opens to your home directory, which may be empty. You may want to make a new folder for your notebook and other files. Either do this through Jupyter or by running a command like ```mkdir data_projects```

#### Copying Files with scp

You will probably want to copy files to and from your EC2 instance, for example to run a notebook you've already written locally on the cloud or to download data collected via EC2. You can do this from the terminal using scp. Here are Amazon's [instructions](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html#AccessingInstancesLinuxSCP). 

1. Just like we did with the ssh commands, edit the example command with the path to your .pem file as well as the IP and domain for your EC2 instance. For example:
	```
	scp -i ~/.ssh/my_key.pem UploadThis.txt ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com:~
	```
	This command will upload ```UploadThis.txt``` from your current working directory to the home directory of your EC2 instance. 
	
2. You can specify another file path instead, for example ```~/some_directory/UploadThis.txt``` if the source file was located in ```~/some_direcctory``` and/or ```...compute.amazonaws.com:~/data_projects``` to copy to the ```~/data_projects``` folder on your EC2 instance. 
2. You can use scp to copy folders by adding the ```-r``` flag, for example: 
	```
	scp -r -i ~/.ssh/my_key.pem ~/folder_to_upload ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com:~/
	```
3. To download files from your EC2 instance, simply switch the "from" and "to" filepaths:
	```
	scp -i ~/.ssh/my_key.pem ec2-user@ec2-13-57-201-20.us-west-1.compute.amazonaws.com:~/download_this.txt ~/Downloads
	```
	
#### Using Screen to Keep Notebooks (or whatever else) Running

A good reason to run notebooks in the cloud is so they can continue to run when you're away from your computer, or your computer isn't connected. Right now, even though our Jupyter Notebook server is running on the EC2 instance, the process will end when our ssh session ends. 

One way to get around this involves using GNU Screen. Screen is a "terminal multiplexer"– a sort of virtual terminal window that we can create, detach from, and reattach to without disturbing whatever processes are happening within the Screen session– even after closing and reopening an ssh connection. 

This [guide](https://opensource.com/article/17/3/introduction-gnu-screen) covers the basics. I'll give a brief example showing how to start a Jupyter Notebook server using Screen.

1. In your EC2 instance, run ```screen``` to start a Screen window
2. From the screen window, run ```jupyter notebook```

     *	Don't forget to double-check the conda environment first
     * 	Save the notebook url somewhere, you'll need it later to reconnect
3. After starting any notebooks you wish to keep running, type ctrl-a, then d to detach from the Screen window. You'll notice you return to your initial shell.
4. Once detached, you can safely close your ssh connection and any running notebooks will continue to run. Close the ssh connection by running ```exit```. 
5. At this point, you should still have access to the notebook. This is because even though you've ended the ssh terminal connection, ssh port forwarding remains active. Go ahead and end that too by running ```killall ssh``` in a local terminal window.
6. Reconnect to your EC2 instance and restart port forwarding just like we did before. You should be able to reconnect to the notebook using the url saved in step 2. If it was running before, it should still be running.
4. To reattach to the Screen window, run ```screen -r``` in the EC2 instance. You'll once again see the Jupyter Notebook console messages.

For more advanced Screen capabilities (multiple windows, etc), see the linked guide or even the Screen [manual](https://www.gnu.org/software/screen/manual/screen.html). 

#### A Quick Exercise with the LA Metro API

The included notebook, based off of one of the ITS Data Camp exercises, uses Python's ```datetime``` module and a simple loop to collect a day's worth of vehicle location data. Although a functional proof of concept, it isn't reliable enough to be used as a final product– especially if you plan to collect multiple days of data without checking to see if it's crashed. Some suggestions for improvement are included in the notebook.

#### Final Notes on AWS

After you've finished gathering data, you may want to stop your instance so that it stops counting towards your usage limits. This can be done via the EC2 Console. Be sure to select "Stop" instead of "Terminate"– stopped instances can be restarted with data perserved, but terminated instances are gone forever. More info [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html).

AWS has various tools to track your usage against the Free Tier limits or more generally, this page provides a basic [introduction](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/tracking-free-tier-usage.html). 

We've really only scratched the surface of what AWS can do– using just two of a vast number of available services: EC2 (our compute platform), and EBS (Elastic Block Store, the default storage type for our EC2 instance). If you found this interesting, definitely learn more through the AWS documentation, AWS Educate, or whatever other resources you can find. Hope you found this intro helpful!
