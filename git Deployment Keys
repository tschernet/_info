* Deployment Key is a (usually read-only) access to a repo to deploy the software
* Attention: cant use the same key on multiple repos! -> need a new key for each repo

1) Create Key:
ssh-keygen -t ed25519 -C "deploy-mysuperapp" -f ~/.ssh/deploy_mysuperapp

this creates "deploy_mysuperapp" (private key) and "deploy_mysuperapp.pub" (public key)

2) add the a host alias to `~/.ssh/config`: this is required to use the correct key per repo
Host github-mysuperapp
    HostName github.com
    User git
    IdentityFile ~/.ssh/deploy_mysuperapp
    IdentitiesOnly yes
(this basically tells ssh: when we come asking for the host "github-mysuperapp", redirect us to "github.com", use user "git", and use the correct identityfile)

3) add deployment key:
go to github -> repository -> settings -> deploy keys
add public key string from .pub file

3) clone the repo:
git clone git@github-mysuperapp:tschernet/mysuperapp
(this uses the alias set up above 
