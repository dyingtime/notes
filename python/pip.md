## PIP

---

`pip`是`Python`的软件包安装程序. 您可以使用`pip`从`Python`软件包索引和其他索引安装软件包.

#### 1.快速开始

```shell
pip install SomePackage              #最新帮本
pip install SomePackage==1.0.0       #指定版本
pip install SomePackage>=1.0.4       #最小版本
pip install --upgrede SomePackage    #升级包
pip uninstall SomePackage            #卸载包
pip search SomePackage               #搜索包
pip show                             #显示安装包信息
pip show -files SomePackage          #显示指定包信息
pip list                             #列出已安装的包
pip list --outdated                  #列出可升级的包
```

#### 2.下载

  ```shell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py #Python 2.7.9+/3.4+ 自带 pip 工具
  
sudo python get-pip.py
sudo python get-pip.py pip==9.0.2 wheel==0.30.0 setuptools==28.8.0 #指定版本

pip install -U pip              #on linux or mac
python -m pip install -U pip    #on window
  ```

`pip`默认会同时安装`setuptools`和`wheel`

- `options`:
    - `--no-setuptools` 不安装`setuptools`

    - `--no-wheel` 不安装 `wheel`

### 3.用户手册

- 1.临时更改 - 使用`-i` 参数

  ```shell
  pip install django -i https://pypi.tuna.tsinghua.edu.cn/simple
  ```

- 2.永久更改 - 修改`~/.pip/pip.conf`配置文件

  ```shell 
  [global]
  index-url = https://pypi.tuna.tsinghua.edu.cn/simple
  ```

- 3.常见国内源

  ```shell
  https://pypi.mirrors.ustc.edu.cn/simple/ #中国科技大学
  https://pypi.tuna.tsinghua.edu.cn/simple/ #清华大学
  ```

## how-to-change-python-version-on-ubuntu

Follow the simple steps to install and configure Python 3.9.0


Step 1: Add the repository and update
Latest Python 3.9 not available in Ubuntu’s default repositories. So, we have to add an additional repository. On launchpad repository
named [deadsnakes](https://launchpad.net/~deadsnakes/+archive/ubuntu/ppa) is available for Python Packages.

Add the deadsnakes repository using the below commands.
```shell
sudo add-apt-repository ppa:deadsnakes/ppa
```

Update the package list using the below command.

```shell
sudo apt-get update
```

Verify the updated Python packages list using this command.

```shell
apt list | grep python3.10

#  WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
#  
#  idle-python3.10/jammy-updates,jammy-updates,jammy-security,jammy-security 3.10.6-1~22.04.2 all
#  libpython3.10-dbg/jammy-updates,jammy-security 3.10.6-1~22.04.2 amd64
#  libpython3.10-dbg/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  libpython3.10-dev/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  libpython3.10-dev/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  libpython3.10-minimal/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  libpython3.10-minimal/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  libpython3.10-stdlib/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  libpython3.10-stdlib/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  libpython3.10-testsuite/jammy-updates,jammy-updates,jammy-security,jammy-security 3.10.6-1~22.04.2 all
#  libpython3.10/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  libpython3.10/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-dbg/jammy-updates,jammy-security 3.10.6-1~22.04.2 amd64
#  python3.10-dbg/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-dev/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  python3.10-dev/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-doc/jammy-updates,jammy-updates,jammy-security,jammy-security 3.10.6-1~22.04.2 all
#  python3.10-examples/jammy-updates,jammy-updates,jammy-security,jammy-security 3.10.6-1~22.04.2 all
#  python3.10-full/jammy-updates,jammy-security 3.10.6-1~22.04.2 amd64
#  python3.10-full/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-minimal/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  python3.10-minimal/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-nopie/jammy-updates,jammy-security 3.10.6-1~22.04.2 amd64
#  python3.10-nopie/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10-venv/jammy-updates,jammy-security 3.10.6-1~22.04.2 amd64
#  python3.10-venv/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
#  python3.10/jammy-updates,jammy-security,now 3.10.6-1~22.04.2 amd64 [installed,automatic]
#  python3.10/jammy-updates,jammy-security 3.10.6-1~22.04.2 i386
```

As seen in the output above, Now we have Python 3.9.0 available for installation


Step 2: Install the Python 3.9.0 package using apt-get
install Python 3.9.0 by using the below command :

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
By default, Python 3 is pointed to Python 3.6. That means when we run python3 it will execute as python3.6 but we want to execute this as python3.9.

Type this command to configure python3:

```shell
sudo update-alternatives --config python3

#  There are 2 choices for the alternative python3 (providing /usr/bin/python3).
#  
#    Selection    Path                 Priority   Status
#  ------------------------------------------------------------
#  * 0            /usr/bin/python3.10   2         auto mode
#    1            /usr/bin/python3.10   2         manual mode
#    2            /usr/bin/python3.7    1         manual mode
```

