# BVR-Infra-Ansible

## Introduction:
This repository conatins the Ansible playbook which is used to create a Infrastructure with the help of Ansible in Azure.This Playbook create a resource group, virtual network, add subnet,public IP address, network security group,virtual network interface card,Create the Linux VM.

- name: Create an Azure Linux VM 

  hosts: localhost 

  connection: local 

  vars: 

    resource_group: myResourceGroup 

    vm_name: myLinuxVM 

    vm_size: Standard_DS1_v2 

    admin_username: azureuser 

    ssh_password_enabled: false 

    ssh_public_key: "<your_ssh_public_key>" 

    location: eastus 

    image: 

      publisher: Canonical 

      offer: UbuntuServer 

      sku: "20.04-LTS" 

      version: latest 

  

 tasks: 

  - name: Create a resource group 

    azure_rm_resourcegroup: 

      name: "{{ resource_group }}" 

      location: "{{ location }}" 

  

  - name: Create virtual network 

    azure_rm_virtualnetwork: 

      resource_group: "{{ resource_group }}" 

      name: myVnet 

      address_prefixes: "10.0.0.0/16" 

  

  - name: Add subnet 

    azure_rm_subnet: 

      resource_group: "{{ resource_group }}" 

      name: mySubnet 

      address_prefix: "10.0.1.0/24" 

      virtual_network: myVnet 

  

  - name: Create public IP 

    azure_rm_publicipaddress: 

      resource_group: "{{ resource_group }}" 

      name: myPublicIP 

      allocation_method: Static 

  

  - name: Create network security group 

    azure_rm_securitygroup: 

      resource_group: "{{ resource_group }}" 

      name: myNetworkSecurityGroup 

      rules: 

        - name: SSH 

          protocol: Tcp 

          destination_port_range: 22 

          access: Allow 

          priority: 1001 

          direction: Inbound 

  

  - name: Create virtual network interface card 

    azure_rm_networkinterface: 

      resource_group: "{{ resource_group }}" 

      name: myNIC 

      virtual_network: myVnet 

      subnet: mySubnet 

      public_ip_name: myPublicIP 

      security_group: myNetworkSecurityGroup 

  

  - name: Create the Linux VM 

    azure_rm_virtualmachine: 

      resource_group: "{{ resource_group }}" 

      name: "{{ vm_name }}" 

      vm_size: "{{ vm_size }}" 

      admin_username: "{{ admin_username }}" 

      ssh_password_enabled: "{{ ssh_password_enabled }}" 

      ssh_public_keys: 

        - path: /home/{{ admin_username }}/.ssh/authorized_keys 

          key_data: "{{ ssh_public_key }}" 

      image: 

        offer: "{{ image.offer }}" 

        publisher: "{{ image.publisher }}" 

        sku: "{{ image.sku }}" 

        version: "{{ image.version }}" 

        network_interfaces: myNIC 
