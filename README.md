# BVR-Infra-Ansible

## Introduction:
This repository conatins the Ansible playbook which is used to create a Infrastructure with the help of Ansible in Azure.This Playbook create a resource group, virtual network, add subnet,public IP address, network security group,virtual network interface card,Create the Linux VM.

- name: Create Azure VM

  hosts: localhost
  
  connection: local
  
  tasks:
  
  - name: Create resource group
  
    azure_rm_resourcegroup:
    
      name: myResourceGroup
      
      location: eastus
      
  - name: Create virtual network
  
    azure_rm_virtualnetwork:
    
      resource_group: myResourceGroup
      
      name: myVnet
      
      address_prefixes: "10.0.0.0/16"
      
  - name: Add subnet
  
    azure_rm_subnet:
    
      resource_group: myResourceGroup
      
      name: mySubnet
      
      address_prefix: "10.0.1.0/24"
      
      virtual_network: myVnet
      
  - name: Create public IP address
  
    azure_rm_publicipaddress:
    
      resource_group: myResourceGroup
      
      allocation_method: Static
      
      name: myPublicIP
      
    register: output_ip_address
    
  - name: Public IP of VM
  
    debug:
    
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."
      
  - name: Create Network Security Group that allows SSH
  
    azure_rm_securitygroup:
    
      resource_group: myResourceGroup
      
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
    
      resource_group: myResourceGroup
      
      name: myNIC
      
      virtual_network: myVnet
      
      subnet: mySubnet
      
      public_ip_name: myPublicIP
      
      security_group: myNetworkSecurityGroup

- name: Create VM

    azure_rm_virtualmachine:
    
      resource_group: myResourceGroup
      
      name: myVM
      
      vm_size: Standard_DS1_v2
      
      admin_username: azureuser
      
      ssh_password_enabled: false
      
      ssh_public_keys:
      
        - path: /home/azureuser/.ssh/authorized_keys
        
          key_data: "<key_data>"
          
      network_interfaces: myNIC
      
      image:
      
        offer: CentOS
        
        publisher: OpenLogic
        
        sku: '7.5'
        
        version: latest
