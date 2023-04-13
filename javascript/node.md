## nvm

### Install

下载并执行安装脚本

```shell
# downloads a script and runs it
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
# add the source lines from the snippet below to the correct profile file 
# s(~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc)
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

### Usage

```shell
nvm install node    # "node" is an alias for the latest version
nvm install 14.7.0  # or 16.3.0, 12.22.1, etc
nvm ls-remote       # list available versionsss
```

## npm

### config registry

- set registry by command 

```shell
npm config set registry http://registry.npmjs.org # config registry global
```

- `.npmrc` file in current project directory 

```ini
registry = https://registry.npm.taobao.org
```

### Usage

```shell
# install git package
npm install git+https://git@git.hub/<username>/<repository>.git[#<branch>]
```

