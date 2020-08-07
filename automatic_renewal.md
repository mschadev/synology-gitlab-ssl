# Automatic renewal
When Let's encrypt renews its certificate, it is not copied automatically, so you need to write a script.
# 1. Add scheduler
1. Open control panel.
2. Go to System > task scheduler
3. Hover create button
4. Scheduled Task > User-defined script
5. Task Settings click
6. Input script code
``` bash
#VARIABLES
ID=""

#/usr/syno/bin/synopkg stop Docker-GitLab
cd /usr/syno/etc/certificate/_archive/${ID}
SYNOLOGY_CERT=$(sudo openssl x509 -checkend 0 -in fullchain.pem)
GITLAB_CERT=$(openssl x509 -checkend 0 -in /volume1/docker/gitlab/gitlab/certs/gitlab.crt)
echo "synology cert status: ${SYNOLOGY_CERT}"
echo "gitLab cert status: ${GITLAB_CERT}"

if [ "${SYNOLOGY_CERT}" != "${GITLAB_CERT}" ]
then
echo "Action Required"
sudo \cp -f privkey.pem /volume1/docker/gitlab/gitlab/certs/gitlab.key;
sudo \cp -f fullchain.pem /volume1/docker/gitlab/gitlab/certs/gitlab.crt;
echo "GitLab restarting.."
/usr/syno/bin/synopkg restart Docker-GitLab
else
echo "no action required."
fi

echo "done."
```
## What is ID?
> There is a certificate in a directory of six digits directory.
`ID` is directory name.
### example
``` bash
ID=Q2EACD
...
```
# Done.