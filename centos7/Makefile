virtualmachine_name 		= centos7-cloudinit
virtualmachine_image 		= https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud.qcow2c
virtualmachine_network 	= default
virtualmachine_os				= rhel7.5
libvirt_storage					= /var/lib/libvirt/images
MKISOFS 								= genisoimage
VIRTSINSTALL 						= virt-install

.PHONY : all clean fetch_image copy_image nocloud create_vm
all : copy_image nocloud create_vm

clean:
	rm -rf /var/lib/libvirt/images/$(virtualmachine_name)_vm.qcow2
	rm -rf /var/lib/libvirt/images/$(virtualmachine_name)_nocloud.iso
	virsh shutdown $(virtualmachine_name)
	sleep 2
	virsh undefine $(virtualmachine_name)

fetch_image:
	wget -O $(libvirt_storage)/$(virtualmachine_name).qcow2 $(virtualmachine_image)

# --selinux-relabel needed for selinux images only
update_image:
	virt-customize -a $(libvirt_storage)/$(virtualmachine_name).qcow2 --update --selinux-relabel

copy_image: $(libvirt_storage)/$(virtualmachine_name).qcow2
	cp $(libvirt_storage)/$(virtualmachine_name).qcow2 $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2

nocloud: meta-data user-data network-config
	$(MKISOFS) \
		-joliet -rock \
		-volid "cidata" \
		-output $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso meta-data user-data network-config

create_vm: $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2
	$(VIRTSINSTALL) \
		-n $(virtualmachine_name) -r 1024 -w network=$(virtualmachine_network) \
		--disk $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2,device=disk,bus=virtio \
		--disk path=$(libvirt_storage)/$(virtualmachine_name)_nocloud.iso,device=cdrom \
		--import \
		--cpu mode=host-passthrough \
		--vcpus 2 \
		--os-variant $(virtualmachine_os) \
		--import \
		--graphics none \
		--console pty,target_type=virtio
		# --cpu mode=host-passthrough
