# Passwordless SSH Login to a Remote Server with Visual Studio Code

Following article describes how to enable You to login to a Remote Server with the industry standard SSH network protocol by using a key and Visual Studio Code, so you do not have to enter your user/password every time you want to connect. It was written for/on MacOS, but the procedure should be the same on Linux and similar for Windows.

## Preparation

At first <a href="<https://code.visualstudio.com/>" target="_blank">download</a>, install and open VS Code.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/8e99b3ac-b3f7-4c06-adfc-eb7e28808ce3/02_grafik.png " =1509x393")


\
Then install the „Remote – SSH" Extension by changing to the Extensions tab, searching for it and click on „Install".

## Fun with Keys

At next open a terminal session by option „New Terminal".

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/31e011f2-ecf7-4f18-854b-b361f3b41ba1/03_grafik-1.png " =937x423")


At the terminal prompt (normally shown at the bottom of your window) connect to your server by entering following line and pressing enter key.

```shell
ssh -p PORT USERNAME@SERVER
```

So for example your line could be like „ssh -p 22 [heinz@mydomain.com](mailto:heinz@mydomain.com)". Then accept/confirm any dialogues you are prompted to for confirmation as well es entering the password.

Now, just leave the ssh session by entering following command to the terminal:

```shell
exit 
```

At next you create a key for ssh (in case you do not already have one) on your local PC. Be sure you really exited/disconnected from the server before:

```shell
ssh-keygen
```

Once done, you can copy the public key part to the server by utilizing following command:

```shell
ssh-copy-id -p PORT USERNAME@SERVER
```

Next time if you login to the server with ssh you will not be asked for the password anymore but the key will be automatically verified in the background.

## Setting up the environment

So let's try that by login at the server (ssh -p PORT USERNAME@SERVER) in the terminal again and by creating a workspace directory for the files you want to edit/control with VS code later on.

```shell
mkdir DIRECTORYNAME
```

The directory name can be anything you prefer, so e.g. „mkdir data" could be suitable. Jump into this directory by sending command:

```shell
cd DIRECTORYNAME
```

e.g. „cd data". Then print the full directory path to the terminal screen by:

```shell
pwd
```

The result will be something like „/home/heinz/data". Copy this path to your clipboard or note/remember it. You are more then 50% done, now.

## Configure VS Code

Open the Remote SSH tab in VS Code and click on „+" at SSH in the Remote Explorer.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/51818c99-f744-4bc9-bb6d-3a95cd5cd58d/04_grafik-2.png " =449x327")


\
Enter the same command you used before to connect to your server at the new prompt shown on top of the window:

```shell

ssh -p PORT USERNAME@SERVER
```

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/2ccdd14f-c804-47a1-86ce-ffcd00ea6659/05_grafik-3.png " =818x128")


Choose the local ssh config file to update, normally the first proposal is fine.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/6c4636e5-ef12-4a6f-ba30-8c006e29ce28/06_grafik-5.png " =620x166")


Now, in the Remote Explorer you should see the new option for connection, like:

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/d8d6568a-3dcb-4f20-b3b8-2864512d0647/07_grafik-6.png " =401x96")


\
If it's not directly shown, just click the refresh button as shown on top right of the screenshot. Then click on the „->" arrow to connect.

Finally, switch to the File Explorer Tab and click on „Open Folder". At the prompt you enter or select the directory you remembered or copied before and click on „OK":

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/e06cd317-7c99-4416-a51a-432f455c1433/08_grafik-7.png " =990x332")


In case you are asked if you trust the authors of the directory it makes sense to choose yes, since you are the author 🙂

We are done. You most likely see an empty folder structure since we just created the folder and you can create new files of any kind, like the „[README.MD](http://README.MD)" on the next screenshot, and folders as you wish.


It's also possible to copy files from your PC to the server by drag-and-drop as well as downloading files/folders by a right-click on it at the VS Code File Explorer.

## Result

Next time you open VS Code the connection will be established automatically. In case you switch between different environments you can always go back to the Remote Explorer tab of VS Code and connect to the server. Be sure to select the folder for connection and not the server itself, otherwise you will be prompted to select a folder at the File Explorer Tab again.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/92cf7d6d-c995-495c-a57d-6983635d4095/10_grafik-8.png " =466x179")


I hope that short tutorial helped you out a bit 🙂