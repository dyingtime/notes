## how-to-change-python-version-on-ubuntu

Follow the simple steps to install and configure Python 3.9.0

Step 1: Add the repository and update
The Latest Python 3.9 not available in Ubuntu’s default repositories. So, we have to add a repository. On launchpad repository
named [deadsnakes](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa) is available for Python Packages.

Step 1: Add the `deadsnakes` repository and update the package list using the below commands.

```shell
sudo add-apt-repository ppa:deadsnakes/ppa && apt-get update
```

Step 2: Install the Python 3.9.0 package using apt-get

```shell
sudo apt-get install python3.9
```

Step 3: Add Python 3.6 & Python 3.9 to update-alternatives
Add both old and new versions of Python to Update Alternatives.

```shell
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 2
```
Step 4: Update Python 3 for point to Python 3.9
By default, Python 3 is pointed to Python 3.6. That means when we run python3 it will execute as python3.6, but we want to execute this as python3.9.

Type this command to configure python3:

```shell
sudo update-alternatives --config python3
```
