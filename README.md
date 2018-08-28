# cloudinit-libvirt
Work in progress

Fedora make sure `export LIBVIRT_DEFAULT_URI="qemu:///system"` in `~/.bashrc`  
Ubuntu this is default

## CentOS7

**Create**   
`sudo make fetch_image update_image`  
`sudo make all`  


login with user `cloud` & password `cloud`  
deatach console: CTRL + ] key  
reattach: `virsh console centos7-cloudinit`  

**Cleanup**  
`sudo make clean`  
After cleanup `fetch_image` and `update_image` not needed to create VM again.
