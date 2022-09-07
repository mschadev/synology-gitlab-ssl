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
ID="l3lwrh"
SYNOLOGY_CERT_DIRECTORY="/usr/syno/etc/certificate/_archive/${ID}"
GITLAB_CERT_DIRECTORY="/volume1/docker/gitlab/certs"

action() {
    echo "Action Required.";
    cd "$SYNOLOGY_CERT_DIRECTORY"
    sudo \cp -f privkey.pem "${GITLAB_CERT_DIRECTORY}/gitlab.key";
    sudo \cp -f fullchain.pem "${GITLAB_CERT_DIRECTORY}/gitlab.crt";
    chmod 400 "${GITLAB_CERT_DIRECTORY}/gitlab.key"
    echo "Reboot Required."
}
#TODO: Add file exist condition.(fullchain.pem, privkey.pem)
if [ ! -d "$SYNOLOGY_CERT_DIRECTORY" ] 
then
    echo "${SYNOLOGY_CERT_DIRECTORY} not found." 
    exit -1
fi

if [ ! -d "$GITLAB_CERT_DIRECTORY" ]
then
    echo "${GITLAB_CERT_DIRECTORY} not found."
    echo "${GITLAB_CERT_DIRECTORY} creating..."
    mkdir "$GITLAB_CERT_DIRECTORY"
    action
    exit 0
fi

cd "$GITLAB_CERT_DIRECTORY"
SYNOLOGY_CERT=$(sudo openssl x509 -checkend 0 -in "${SYNOLOGY_CERT_DIRECTORY}/fullchain.pem")
GITLAB_CERT=$(openssl x509 -checkend 0 -in "${GITLAB_CERT_DIRECTORY}/gitlab.crt")
echo "synology cert status: ${SYNOLOGY_CERT}"
echo "gitLab cert status: ${GITLAB_CERT}"

if [ "${SYNOLOGY_CERT}" != "${GITLAB_CERT}" ]
then
    action
else
    echo "No Action Required."
fi

echo "Done."
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
