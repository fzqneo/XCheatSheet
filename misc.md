## vncserver
```bash
vncserver :1
vncserver -geometry 1280x1024 :1
vncserver -kill :1
```

## ssh agent
```bash
eval "$(ssh-agent -s)"
ssh-add -K ~/.ssh/id_rsa
ssh-add -L
```

## git
```bash
git rebase -i origin/dev
git push origin +dev  # force push
git commit --amend  # amend message of last commit
```

## apt, dpkg
```bash
dpkg --list PACKAGE
dpkg --status PACKAGE

apt-cache search KEYWORD
apt-cache rdepends PACKAGE  # show reverse dependency

apt-get autoremove --purge
```

## netstat
```bash
netstat -plnt # show programs listening on TCP
```

## ansible
```bash
ansible GROUP -i hosts/inventory -m ping -a "some arg" --check
ansible-playbook diamond-sites.yml --tags "restart" --limit "diamond[0]" -i "somehost," --key-file tom.id_rsa
```

## docker
```bash
docker ps --filter 'ancestor=registry.xxx.yyy:zzz' --filter 'status=running' --filter 'name=diamond-resource'
docker rmi $(docker images -q --filter dangling=true)
```

## curl, httpie
```bash
curl -X POST -F image=@dog.png http://localhost:8000/predict
http --form post http://localhost:8000/predict picture@apple.jpg confidence=0.95
```
