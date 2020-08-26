### PIP

 `pip`是`Python`的软件包安装程序. 您可以使用`pip`从`Python`软件包索引和其他索引安装软件包.

##### 1.快速开始

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

##### 2.下载

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

#### 3.用户手册

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
    http://mirrors.aliyun.com/pypi/simple/ #阿里云 
    https://pypi.mirrors.ustc.edu.cn/simple/ #中国科技大学
    http://pypi.douban.com/simple/ #豆瓣
    https://pypi.tuna.tsinghua.edu.cn/simple/ #清华大学
    http://pypi.mirrors.ustc.edu.cn/simple/ #中国科学技术大学
    ```
