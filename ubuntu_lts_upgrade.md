## Just a quick note to my self 

We mostly want to run LTS-versions only, of Ubuntu. 

In case you absolutely need to update, in between LTS-versions, this is how it's done: 


## Ubuntu 18.04 upgrade to Ubuntu 19.04 (not LTS)


`nano /etc/update-manager/release-upgrades`

-> Prompt=normal 

`sudo apt update && sudo apt dist-upgrade -y`

`sudo do-release-upgrade`
