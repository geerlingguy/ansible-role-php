test_ubuntu-14.04:
	VM_BOX='ubuntu/trusty64' vagrant up
	VM_BOX='ubuntu/trusty64' vagrant provision
	VM_BOX='ubuntu/trusty64' vagrant destroy -f

test_centos-6.5:
	VM_BOX='chef/centos-6.5' vagrant up
	VM_BOX='chef/centos-6.5' vagrant provision
	VM_BOX='chef/centos-6.5' vagrant destroy -f

test_ubuntu-12.04:
	VM_BOX='hashicorp/precise64' vagrant up
	VM_BOX='hashicorp/precise64' vagrant provision
	VM_BOX='hashicorp/precise64' vagrant destroy -f
