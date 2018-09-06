---
published: true
title: Using SSH on Windows
---
To perform admin tasks on our Linux EC2 instances we'll need to SSH onto them, allowing us to execute remote commands through our terminal. On Linux / Macs this is a very easy process but for Windows users it's a bit more complicated. One way that is often recommended is to use [Putty](https://www.putty.org/) but this requires converting the .pem files that AWS supplies into a different format and is not as simple as using the terminal. Another option is using Git Bash, which works well and uses the same syntax as regular Bash.

For Windows 10 users there's a new (and better IMO) option, which is to use [Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/install-win10). This lets you install an actual Linux distro inside Windows and run (some) Linux software. WSL is not running in a VM and is fully integrated into your Windows system. This lets us do some cool things, like having the Docker daemon running in Windows but using the Linux client tools to connect to it.

Working with files that need special permissions (like the .pem files that AWS uses for SSH keys)  
requires a bit of care with how we work with them. Windows sets aside an area on your file system as special Linux compatible area where we can set the necessary permissions but any files outside this area won't work.
Let's assume you've downloaded key.pem from AWS and it's saved in D:\Downloads and you want to use this file in Linux. From Bash this file would be located at `/mnt/d/Downloads/key.pem`. We can copy this file to our home directory inside the Linux filesystem using `cp /mnt/d/Downloads/key.pem ~` Once it's there then we can set the required permissions using `chmod 400 key.pem` from home.

Basic Linux skills are an essential requirement for any cloud architect and WSL represents a chance to learns these from the comfort of a familiar environment. I recommend using it over the alternatives.
