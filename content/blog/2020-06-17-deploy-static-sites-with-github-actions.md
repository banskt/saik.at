---
title: "Deploy static sites with Github Actions"
date: 2020-06-17T22:46:18+02:00
draft: false
---

I used [Hugo](https://gohugo.io/) to generate a static site 
and copy it to my web server [using rsync](https://gohugo.io/hosting-and-deployment/deployment-with-rsync/) over SSH. 
Recently, I started using [Github Actions](https://github.com/features/actions),
which is a free continuous integration (CI) platform from Github.

On my web server, I created a user with restricted permissions so that Github is not granted access to my full server.
Setting up Github Actions is as simple as creating the YAML file `.github/workflows/deploy.yml` inside the repository.
The [documentation](https://docs.github.com/en/actions) is a 
[good starting point](https://docs.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow).
My workflow file contains instructions to run a `job` on every `push` event on this repository.
```yml
name: Autodeploy
on: [push]
jobs:

  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Git checkout
      uses: actions/checkout@v2
      with:
        token: ${{ secrets.MY_GITHUB_PUBLIC_PAT }}
        submodules: recursive

    - name: rsync git repository to vultr
      uses: AEnterprise/rsync-deploy@v1.0
      env:
        DEPLOY_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        SERVER_PORT: ${{ secrets.SSH_PORT }}
        SERVER_IP: ${{ secrets.HOST }}
        USERNAME: ${{ secrets.USERNAME }}
        ARGS: "-e -c -r --delete"
        FOLDER: "./"
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
The `job` contains three steps using existing GitHub Actions:
1. [__checkout__](https://github.com/actions/checkout) the repository and submodules on the "mysterious" server,
where the GitHub Actions are running.
2. [__rsync__](https://github.com/AEnterprise/rsync-deploy) the content to my web server.
3. [__SSH__](https://github.com/appleboy/ssh-action) to my web server and run the `deploy_saik_at` script.  

These actions use several vulnerable personal information, 
which I trusted Github to keep secure by keeping them as Secrets in the Settings of the repository.
The `HOST`, `USERNAME` and `SSH_PORT` are respectively the IP address of my web server,
username of the restricted account, and the port that listens to incoming SSH requests.
The `SSH_PRIVATE_KEY` is the RSA private key generated on my web server.
The corresponding public key was added to `.ssh/authorized_keys` on the web server.
```bash
ssh-keygen -t rsa -b 4028 -C "username@client.com" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
I had to use my Github registered email for generating the key.
The `MY_GITHUB_PUBLIC_PAT` is my [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)
for Github and is required for recursive submodule checkout. 
It can be generated [here](https://github.com/settings/tokens).
The `deploy_saik_at` is a `bash` script on the web server which generates the static site
and, if successful, makes it public.
Here is a trimmed down version:
```bash
TEMP=$( mktemp -d )
${HUGO} --destination="${TEMP}" --logFile="${LOGFILE}"
if [ $? -eq 0 ]; then
    rsync -aq --delete "${TEMP}/" "${DEPLOY_DIR}"
fi
```

## Resources
1. If restricted user cannot be created in the webserver, then [restrict SSH to rsync](http://gergap.de/restrict-ssh-to-rsync.html).
2. Some other relevant blogs: 
[\[1\]](https://ruddra.com/hugo-deploy-static-page-using-github-actions/)
[\[2\]](https://zartman.xyz/blog/gh-static-deploy/)
[\[3\]](https://www.schakko.de/2020/07/20/restrict-ssh-to-rsync-for-deploying-files-with-github-actions).
