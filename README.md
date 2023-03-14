# OCI ANSIBLE INSTANCE

Ansible script for provisioning a compute instance in OCI

## Requirements

- OCI CLI
- ansible with oracle oci collection
- ssh-keygen

## Versions

Testing with:

- OCI CLI 3.23.3
- ansible 2.13.3
- ansible oracle.oci collection 4.15.0

## Configuration

1. Follow the instructions to install and add the authentication to your tenant https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm

2. Clone this repository

```
# Stable diffusion 1.5
git clone --depth 1 --branch main https://github.com/jorgeceballosgarcia/oci-ansible-instance.git
```

3. Set variables in your path. 

- The comparment OCID where the instance will be created.

```
export compartment_ocid='<comparment-ocid>'
```


4. Personalize your oci-ansible-instance-vars.yml

```
#tenancy vars
instance_compartment: "{{ lookup('env', 'compartment_ocid') }}"
#Example: ocid1.compartment.oc1..aaaaaaaatw53h3inxmpseokpj7ld7atkusnsc6pu3qlw46drotkbhvgpattq
#You can provide the name of instance avaliability domains or ansible get the first one
instance_ad: ""
region: "eu-paris-1"

#instance vars
instance_name: "oci-instance-ansible"
instance_hostname: "{{ instance_name }}"
instance_shape: "VM.Standard.E4.Flex"
instance_memory_in_gbs: "16"
instance_ocpus: "1"
instance_os: "Oracle Linux"
# You can provide an "OL" image or ansible search last image version of SO 
# find OL image ocids per region here: https://docs.cloud.oracle.com/iaas/images/image/501c6e22-4dc6-4e99-b045-cae47aae343f/
instance_image_ocid: ""
#Example: ocid1.image.oc1.eu-paris-1.aaaaaaaamkinyu5l6foftfo6abgeajhtsgb22us3c2bbumlo273oeueqhona

#Network vars
quad_zero_route: "0.0.0.0/0"
TCP_protocol: "6"
SSH_port: "22"
vcn_name: "{{ instance_name }}-vcn"
vcn_cidr_block: "10.0.0.0/16"
vcn_dns_label: "{{ instance_name }}"
nat_name: "{{ instance_name }}-nat-gateway"
route_table_name: "{{ instance_name }}-route-table-private"
route_table_rules:
    - cidr_block: "{{ quad_zero_route }}"
      network_entity_id: "{{ ng_id }}"
subnet_cidr: "10.0.0.0/24"
subnet_name: "{{ instance_name }}-subnet-private"
subnet_dns_label: "{{ instance_name }}-subnet-private"
securitylist_name: "{{ instance_name }}-security-list-private"
```

5. Execute the script generate-keys.sh to generate private key to access the instance

```
sh generate-keys.sh
```

## Build
To build simply execute the next command. 

```
ansible-playbook oci-ansible-instance.yml
```

## Connect to compute instance

The output of the ansible script will give the ssh full command so you only need to copy and paste

```
ssh -i server.key -o ProxyCommand="ssh -i server.key -W %h:%p -p 22 ocid1.bastionsession.XXXX@host.bastion.XXXX.oci.oraclecloud.com" -p 22 opc@<instance_private_ip>
```

The instance created is located in a vcn with a private subnet, and "Bastion Service" is used to connect to it.
The life of the bastion service session is one hour, after that time it disappears. To be able to connect again, we will have to launch ```ansible-playbook oci-ansible-instance-bastion-session.yml``` and it will create a new bastion session.

## Clean
To delete the instance execute.
```
ansible-playbook oci-ansible-instance-destroy.yml
```

## Start&Stop Instance
To start or stop the instance just execute the script start-stop-instance.sh

If the instance is RUNNING the script STOP it and viceversa

```
/bin/bash start-stop-instance.sh
```
