virtualmachine_name 		= redhat
virtualmachine_image 		= ../rhel-server-7.5-x86_64-kvm.qcow2
virtualmachine_network 	= default
virtualmachine_os				= rhel7.5
virtualmachine_resize		= 10G
libvirt_storage					= /var/lib/libvirt/images
MKISOFS 								= genisoimage
VIRTSINSTALL 						= virt-install

.PHONY : all clean fetch_update_image copy_image nocloud create_vm
all : copy_image nocloud create_vm

clean:
	rm -rf $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2
	rm -rf $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso
	virsh destroy $(virtualmachine_name) || true
	virsh undefine $(virtualmachine_name) || true

clean_image:
	rm -rf $(libvirt_storage)/$(virtualmachine_name).qcow2

fetch_update_image: $(libvirt_storage)/$(virtualmachine_name).qcow2
$(libvirt_storage)/$(virtualmachine_name).qcow2:
	cp $(virtualmachine_image) $(libvirt_storage)/$(virtualmachine_name).qcow2c
	qemu-img resize $(libvirt_storage)/$(virtualmachine_name).qcow2c $(virtualmachine_resize)
	mv $(libvirt_storage)/$(virtualmachine_name).qcow2c $(libvirt_storage)/$(virtualmachine_name).qcow2

copy_image: $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2
$(libvirt_storage)/$(virtualmachine_name)_vm.qcow2: $(libvirt_storage)/$(virtualmachine_name).qcow2
	cp $(libvirt_storage)/$(virtualmachine_name).qcow2 $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2

nocloud: $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso
$(libvirt_storage)/$(virtualmachine_name)_nocloud.iso: meta-data user-data network-config
	$(MKISOFS) \
		-joliet -rock \
		-volid "cidata" \
		-output $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso meta-data user-data network-config

vm_domain := $(shell virsh list --all --name | grep "$(virtualmachine_name)" )
ifndef vm_domain
create_vm: $(libvirt_storage)/$(virtualmachine_name)_nocloud.iso $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2
	$(VIRTSINSTALL) \
		-n $(virtualmachine_name) -r 1024 -w network=$(virtualmachine_network) \
		--disk $(libvirt_storage)/$(virtualmachine_name)_vm.qcow2,device=disk,bus=virtio \
		--disk path=$(libvirt_storage)/$(virtualmachine_name)_nocloud.iso,device=cdrom \
		--import \
		--vcpus 2 \
		--os-variant $(virtualmachine_os) \
		--graphics none \
		--console pty,target_type=virtio
endif