# pve_wp_test
on PVE host:
### Prep. ansible_user w/all nessesary rights, ssh-keys, api-token, etc. 
    vi /etc/network/interfaces  
    #------ add br to #/etc/network/interfaces on your PVE ------  
    auto vmbr1  
    iface vmbr1 inet static  
       address 10.0.10.68/24  
        bridge-ports enp2s0.100  
        bridge-stp off  
        bridge-fd 0  
    #-----  
  ifup vmbr1  
    
### Prep. image
  sed -i -e 's/pve-enterprise/pve-no-subscription/g' /etc/apt/sources.list.d/pve-enterprise.list  
  sed -i -e 's/enterprise/download/g' /etc/apt/sources.list.d/pve-enterprise.list  
  sed -i -e 's/https/http/g' /etc/apt/sources.list.d/pve-enterprise.list  
  wget -O /tmp/ubuntu-amd64.img https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img  
  apt update -y && apt install libguestfs-tools -y  
  virt-customize -a /tmp/ubuntu-amd64.img --install qemu-guest-agent  
  qemu-img resize /tmp/ubuntu-amd64.img +6G  
  qm create 9000 --memory 2048 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci  --name 9000  
  qm set 9000 --scsi0 local-lvm:0,import-from=/tmp/ubuntu-amd64.img   #change storage  
  qm set 9000 --ide2 local-lvm:cloudinit                              #change storage  
  qm set 9000 --boot order=scsi0  
  qm set 9000 --serial0 socket --vga serial0  
  qm set 9000 --agent enabled=1  
  qm set 9000 --sshkey ~/.ssh/id_rsa.pub  
  rm /tmp/ubuntu-amd64.img  

# Prep. ansible   
set some ansible vars   
run#  
ansible-playbook --vault-password-file .vault_pass infra.yml   

