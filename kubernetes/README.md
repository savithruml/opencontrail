## Deploying Kubernetes with OpenContrail CNI on AWS

### LAUNCH INSTANCES (Manual)

NOTE: If you are interested in an automated instatiation of AWS instances, please follow this [play](https://github.com/savithruml/ansible-labs/tree/master/aws)

    * Ansible Instance  (x1)
    
      This is the instance from which, we will run our playbooks
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.micro
        DISK:       20 GB
        SEC GRP:    Allow all traffic from everywhere
    
    * K8s-Master Instance   (x1)
    
      This is the instance which will host Kubernetes & OpenContrail control-plane services (OpenContrail Controller)
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.xlarge
        DISK:       200 GB
        SEC GRP:    Allow all traffic from everywhere
    
    * K8s-Node Instance    (x1)
    
      This is the instance which will host Kubernetes & OpenContrail data-plane services (OpenContrail vRouter)
    
        IMAGE:      Ubuntu 16.04 LTS
        FLAVOR:     t2.xlarge
        DISK:       200 GB
        SEC GRP:    Allow all traffic from everywhere

    NOTE: Make sure to launch the instances in the same subnet & remember to select the auto-assign public IP option


### PREPARE NODES FOR DEPLOYMENT

    * Run these commands on all nodes. This will enable root access with password
    
            (all-nodes)# sudo su
            (all-nodes)# passwd
            (all-nodes)# sed -i -e 's/PermitRootLogin prohibit-password/PermitRootLogin yes/g' -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
            (all-nodes)# service sshd restart
          
          
    * Install dependencies
    
            (all-nodes)# apt-get update -y && apt-get install python -y
    
    
    * Logout & login as root user. Enable passwordless SSH for k8s master & node
   
            (ansible-node)# ssh-keygen -t rsa
            
          Make sure you utilize the internal hostname of the AWS instances while copying the SSH keys
    
          Eg. ssh-copy-id root@ip-10-10-10-1
    
            (ansible-node)# ssh-copy-id root@<k8s-master>
            (ansible-node)# ssh-copy-id root@<k8s-node>
      
            (ansible-node)# cd /root
            (ansible-node)# git clone https://github.com/savithruml/opencontrail
      
      
    * Populate hosts file with k8s-master & k8s-node info. Remember to use the private IP address of the instance
    
            (ansible-node)# cat /root/opencontrail/kubernetes/hosts
       
                  [masters]
                  10.10.10.1

                  [nodes]
                  10.10.10.2
                  
                  
     * Install ansible
     
            (ansible-node)# apt-get install python-pip -y
            (ansible-node)# pip install ansible
            
        
 ### RUN THE PLAY
 
            (ansible-node)# cd /root/opencontrail/kubernetes
            (ansible-node)# ansible-playbook -i hosts site.yml
            
            
 ### VERIFY
 
     * Verify that OpenContrail pods are running
 
            (k8s-master)# kubectl get pods -n kube-system -o wide | grep contrail
               
               contrail-agent-nmtvz                      1/1       Running             0          14m       10.10.10.2    ip-10-10-10-2
               contrail-analytics-44gpk                  1/1       Running             0          15m       10.10.10.1    ip-10-10-10-1
               contrail-analyticsdb-brrk6                1/1       Running             0          15m       10.10.10.1    ip-10-10-10-1
               contrail-controller-x6nxv                 1/1       Running             0          15m       10.10.10.1    ip-10-10-10-1
               contrail-kube-manager-xxv2h               1/1       Running             0          15m       10.10.10.1    ip-10-10-10-1

            
    * Verify that OpenContrail services are all up
    
            (k8s-master)# kubectl exec -it contrail-controller-x6nxv contrail-status
            
         Similarly run for other OpenContrail pods
      
      
    * Verify that you can login to the Web-UI of both Kubernetes & OpenContrail SDN
    
            OpenContrail dashboard: https://<k8s-master-ip>:8143
            Kubernetes dashboard: http://<k8s-node-ip>:9090
