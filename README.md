
# Apt Tool Package Creation
This walkthrough takes you step by step on how to create an apt package from your .NET Core 3.0 application code so that the same apt package can be easily installed on user's client machine using a simple apt-install command.
it is inspired by this <a href="https://medium.com/bluekiri/packaging-a-net-core-service-for-ubuntu-4f8e9202d1e5">article</a>

# Deployment Steps
<b>Step 1:</b> You need to <b>'Download Code'</b> for your .NET Core Application from <a href="https://github.com/komalashrafsyed/pingasync">here</a> </br>

use the <b>git clone codeurl</b> command to download the code to your VM </br>

<b>Step 2:</b> You need a Linux OS based virtual machine, you can use the following command to create one, or you can even use an existing VM </br>

<b> az vm create --resource-group myResourceGroup --name myVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys </b>

<b>Step 3:</b> After you ssh into the VM using the command "ssh azureuser@ipaddress", you can type the following to install the relevant required packages</br>
<b>
$ sudo wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb</br>
$ sudo dpkg -i packages-microsoft-prod.deb</br>
$ sudo apt-get install apt-transport-https</br>
$ sudo apt-get update</br>
$ sudo apt-get install dotnet-sdk-3.0</br>
$ sudo dotnet --version</br>


$ git clone https://airband@dev.azure.com/airband/Komal/_git/Komal</br></b>
#cd into the code folder at PingAsync level 
</br>
<b>
$ sudo dotnet build </br>
$ sudo dotnet publish -c Release --self-contained -r ubuntu.18.04-x64</br>
</b>

#run the following this 
</br>
<b>$ sudo dotnet tool install -g dotnet-symbol</br></b>
#it is ok if the following command doesnt run as it if for .NET SDK 2.1
</br>
<b>$ sudo dotnet-symbol --symbols bin/Release/netcoreapp3.0/ubuntu.18.04-x64/publish/</br></b>


#run the following after the above has run successfully or not
</br>
<b>$ sudo apt install gnupg pbuilder ubuntu-dev-tools apt-file dh-make bzr-builddeb</br>
$ export DEBEMAIL="******syed@microsoft.com"</br>
$ export DEBFULLNAME="PingAsync Tool"</br>
$ sudo dotnet clean</br>
$ sudo rm -rf bin obj</br>

$ cd ..</br>
$ sudo mv PingAsync pingasync-2.0</br>
$ sudo tar cvzf pingasync-2.0.tar.gz pingasync-2.0</br>
$ sudo cd pingasync-2.0</br>
$ sudo dh_make -f ../pingasync-2.0.tar.gz -s -c mit -n</br>

$ cd debian </br>
$ sudo rm *ex *EX </br>
$ sudo rm README README.Debian README.source pingasync-docs.docs </br>
</b>

<b>Step 4:</b> After the enviornment has been successfully setup, the following demonstrates Files Changes that need to be made in the following files located in the <b> debian </b> folder. You would use the sudo vim -filename command to open each of the following files and make changes as shown. So for instance your first file would be opened </br>
<b>$ sudo vim changelog </b> </br>

--------------------------------------
<b>changelog</b> file changed </br>
--------------------------------------

pingasync (5.0-0ubuntu1) bionic; urgency=medium

  * Initial Release.

 -- PingAsync Tool <*****syed@microsoft.com>  Wed, 27 Nov 2019 22:37:13 +0000

<img src="https://komalsandboxdiag.blob.core.windows.net/pingarmtemplatereadmefiles/changelog.png" >

--------------------------------------
<b>control</b> file changed </br>
--------------------------------------

Source: pingasync </br>
Section: unknown </br>
Priority: optional</br>
Maintainer: root <******syed@microsoft.com></br>
Build-Depends: debhelper (>= 10)</br>
Standards-Version: 4.1.2</br>
Homepage: <insert the upstream URL, if relevant></br>
#Vcs-Git: https://anonscm.debian.org/git/collab-maint/pingasync.git
</br>
#Vcs-Browser: https://anonscm.debian.org/cgit/collab-maint/pingasync.git
</br>

Package: pingasync</br>
Architecture: any</br>
Depends: ${shlibs:Depends}, ${misc:Depends}</br>
Description: <insert up to 60 chars description></br>
 <insert long description, indented with spaces></br>

<img src="https://komalsandboxdiag.blob.core.windows.net/pingarmtemplatereadmefiles/control.png" >

--------------------------------------
<b>rules</b> file changed </br>
---------------------------------------

#!/usr/bin/make -f
</br>
#export DH_VERBOSE = 1
</br>
%: </br>
	dh $@ --with=systemd </br>


override_dh_auto_build: </br>
	dotnet publish -c Release --self-contained -r ubuntu.18.04-x64 </br>
	# auto-build disabled </br>
override_dh_auto_install: </br>
	# install application 
	</br>
	mkdir -p debian/pingasync/opt/ksyed/pingtool </br>
	install -D -m 755 bin/Release/netcoreapp3.0/ubuntu.18.04-x64/publish/* debian/pingasync/opt/ksyed/pingtool </br>
	rm debian/pingasync/opt/ksyed/pingtool/*.pdb #delete pdb
	</br>

<img src="https://komalsandboxdiag.blob.core.windows.net/pingarmtemplatereadmefiles/rules.png" >

--------------------------------------
<b> pingasync.service </b> file changed </br>
--------------------------------------
You write the following in console to create a new pingasync.service file </br>
<b>$ sudo vim pingasync.service </b>
</br>


[Unit] </br>
Description=Ping Async </br>
After=network.target </br>

[Service] </br>
Type=simple </br>
ExecStart=/opt/ksyed/pingtool/pingtool </br>
ExecReload=/bin/kill -HUP $MAINPID </br>

[Install] </br>
WantedBy=multi-user.target </br>


<img src="https://komalsandboxdiag.blob.core.windows.net/pingarmtemplatereadmefiles/pingasyncservice.png" >

</br>

</b>
<b>Step 5:</b> After the files have been changed then go up a level in folder heirarchy as shown below and run the following commands </br>
<b>

$ cd ..</br>
$ sudo dpkg-buildpackage -b --no-sign</br>
</b>
------------</br>
<b>Note: </b>
#if the above exits with error code 2 and does not go through run the following 3 commands marked with a * and then retry Step 5 command <b> sudo dpkg-buildpackage -b --no-sign </b> to ensure it runs successfully
</br>
<b>
*
$ sudo apt-get install lttng-tools </br> 
*
$ sudo apt-get install lttng-modules-dkms </br>
*
$ apt-get install liblttng-ust-dev </br>
</b>
------------</br>
<b>Step 6:</b> After the above commands has run successfully, then run the following commands </br>

<b>
$ cd ..</br>
$ sudo dpkg -i pingasync_2.0-0ubuntu1_amd64.deb </br>
$ sudo apt-get install gnupg rng-tools </br>
$ gpg --gen-key </br>
</b>
#copy the key generated by the above command in a seperate file you will need this in the following steps, your key should be in the format as shown below (image below as shows it)
</br>
------
</br>
<b>
pub     rsa3072 2019-11-28 [SC] [expires: 2021-11-27] </br>
        DDDXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX315E </br>
uid                    PingAsync Tool <******syed@microsoft.com> </br>
sub     rsa3072 2019-11-28 [E] [expires: 2021-11-27] </br>
</br>
</b>
<img src="https://komalsandboxdiag.blob.core.windows.net/pingarmtemplatereadmefiles/gpg%20gen%20key.png" >

<b>Step 7:</b> After the above commands has run successfully, and you have a key with you in a seperate file, then run the following commands </br> navigate to position at directory '/Komal/Asyn-r-code/' 
</br>

<b>
$ sudo apt-get install reprepro </br>
$ mkdir repo && cd repo </br>
$ mkdir conf && cd conf </br>
$ touch distributions </br>
$ sudo vim distributions</br>
</b>

--------
<b>distributions </b> file changed </br>
--------

<b>#change your file name and put the key SignWith below obtained in the previous Step
</br>
#this is the pub key id given earlier step 
</b>
</br>

Origin: PingAsync Tool </br>
Label: pingasync </br>
Codename: bionic </br>
Architectures: amd64 </br>
Components: main </br>
Description: Personal repository </br>
SignWith: B501DE17DA19A16F  </br>

-------

<b>Step 8:</b> After the distributions file has been successfully changed, now we will upload the apt deb file to a git repository of our choice so that users wanting to use our apt distribution package will be able to download and install it. The following describes a way to upload the consolidated folder with the repo folder</br>
 #give path of repo and also of the deb file if you in the folder where both
 </br>
 #the repo folder and the deb file are on same level then dont need to give it a path
 </br>
 </br>
 <b>
$ sudo reprepro --basedir repo includedeb bionic pingasync_4.0-0ubuntu1_amd64.deb </br>
$ sudo reprepro --basedir repo includedeb bionic pingasync*.deb </br>
$ sudo reprepro --basedir repo list bionic </br>

$ sudo gpg --output PUBLIC.KEY --armor --export ******syed@microsoft.com </br>
$ sudo mv PUBLIC.KEY repo </br>

</b>

<b>Step 9:</b> Once everything has been consolidated into a single repo folder then you will clone an empty git repo into your repo folder so you can upload the distribution final code to it, make sure to move the PUBLIC.KEY file and all other folder to your folder which has been cloned from a git repository, make sure you have ownership rights to that folder in order to be able to do a push after commit, it will ask for credentials in order for you push the code to the online github repo</br>

<b>
$ cd repo </br>
$ git clone ---- distination url of an online empty repo let's suppose it's called destfolder for this example </br>

$ sudo mv PUBLIC.KEY destfolder</br>
$ sudo mv db destfolder </br>
$ sudo mv conf destfolder</br>
$ sudo mv pool destfolder</br>
$ cd destfolder </br>
$ git add --all </br>
$ git commit -m "Package repository" </br>
$ git push </br>


 
#enter username and password for your git repo  </br>
</b>
</br>
 At this stage you have successfully uploaded your apt package and it is ready to be downloaded by anyone looking to use your apt package on their system or vm, instructions to access or install the apt package are given <a href="https://github.com/komalashrafsyed/PingAppARMTemplate/tree/master/subdir">here</a> along with showing your how to prepare your VM for the particular apt PingAsync package we have been dealing with so far </br>
</br>










