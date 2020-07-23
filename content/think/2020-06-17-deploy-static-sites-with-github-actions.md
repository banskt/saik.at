---
title: "Deploy static sites with Github Actions"
date: 2020-06-17T22:46:18+02:00
draft: false
---

This Hugo website is deployed using [Github Actions](https://github.com/features/actions).
I used a specific user with restricted permissions so that Github is not granted access to my full server.
The action is created as an YAML file, `.github/workflows/deploy.yml` inside the site repository.
```yml
name: Autodeploy
on: [push]
jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Git checkout
      uses: actions/checkout@v2

    - name: rsync git repository to vultr
      uses: AEnterprise/rsync-deploy@v1.0
      env:
        DEPLOY_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        ARGS: "-e -c -r --delete"
        SERVER_PORT: ${{ secrets.SSH_PORT }}
        FOLDER: "./"
        SERVER_IP: ${{ secrets.HOST }}
        USERNAME: ${{ secrets.USERNAME }}
        SERVER_DESTINATION: "./saik.at"

    - name: run Hugo
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        port: ${{ secrets.SSH_PORT }}
        script: source deploy_saik_at
```
The `deploy_saik_at` file at the server builds the hugo library, with something like this:
```bash
TEMP=$( mktemp -d )
${HUGO} --destination="${TEMP}" --logFile="${LOGFILE}"
if [ $? -eq 0 ]; then
    rsync -aq --delete "${TEMP}/" "${DEPLOY_DIR}"
fi
```

## SSH Key
The SSH key is generated as:
```bash
mkdir ~/.ssh
ssh-keygen -t rsa -b 4028 -C "bnrj.saikat@gmail.com"
cd .ssh/
cat id_rsa.pub >> authorized_keys
chmod 600 authorized_keys
```
and the private key in `.ssh/id_rsa` is used as the `SSH_PRIVATE_KEY` secret of the repository. 

## Resources
* If only one user is allowed in webserver, then [restrict SSH to rsync](http://gergap.de/restrict-ssh-to-rsync.html).
* Actions used:
    * [checkout git](https://github.com/actions/checkout)
    * [rsync to server](https://github.com/AEnterprise/rsync-deploy)
    * [bash create Hugo static site](https://github.com/appleboy/ssh-action)
* Long stories: [1](https://ruddra.com/hugo-deploy-static-page-using-github-actions/),
[2](https://zartman.xyz/blog/gh-static-deploy/), [3](https://www.schakko.de/2020/07/20/restrict-ssh-to-rsync-for-deploying-files-with-github-actions).
