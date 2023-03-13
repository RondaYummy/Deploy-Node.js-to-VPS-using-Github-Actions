## Deploy Node.js to VPS using Github Actions
![](https://raw.githubusercontent.com/danielwetan/node1/master/nodejs-github-actions.png)


> Steps to deploy Node.js to VPS using PM2 and Github Actions


#### 1. Clone repo to VPS folder

```bash
git clone https://github.com/you-user/tou-repo.git
```

#### 2. Setup CI workflows

location: `/.github/workflows/ci.yml`

example: /.github/workflows/ci.yml

```yaml
# This workflow will do a clean install of node dependencies, 
# build the source code and run tests across different versions of node
# For more information see: 
# https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

name: Node.js CI

on:
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js 16
      uses: actions/setup-node@v2
      with:
        node-version: 16.x
    - run: npm i
    - run: npm run build --if-present
    - run: npm test
```

#### 3. Setup CD workflows

location: `/.github/workflows/cd.yml`

example: /.github/workflows/cd.yml

```yaml
# This is a basic workflow to help you get started with Actions

name: Node.js CD

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ main ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    - name: Deploy using ssh
      uses: appleboy/ssh-action@v0.1.8
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.PRIVATE_KEY }}
        port: 22
        script: |
          cd ~/you-project
          git pull origin master
          git status
          npm install --only=prod
          pm2 restart node1
```

#### 4. Setup Private Key and Public Key

```bash
ssh-keygen -t rsa -b 4096 -m PEM -C "github-actions-node1"

# view the key value
cat id_rsa-github-node1.pub # Public Key
cat id_rsa-github-node1 # Private Key
```

#### 5. Copy Public Key to `authorized key`

```bash
cat id_rsa-github-node1.pub

# Copy the output to
vi ~/.ssh/authorized_keys
```

#### 6. Copy Private Key
```bash
# Type and copy the output
cat ~/.ssh/id_rsa
```

#### 7. Setup Github Secret keys

```bash
# Github Secret location
Settings -> Secrets -> Actions -> New repository secret

PRIVATE_KEY = "Copy generated private key from vps to github secret"
HOST = "YOUR SERVER ADDRESS, example: 172.41.91.123" 
USERNAME = "YOUR SERVER USERNAME, example: daniel"
```
