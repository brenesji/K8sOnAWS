# K8sOnAWS
# Deploy a 3 nodes K8s Cluster on AWS

1. Create the VMs in AWS (**t2.medium**)

OS Image: Amazon Linux  6.12 64 bit
Instance Type: t2.medium
KeyPar: (create new one)
Nw Settings: Create New Security Group 
Storage: 8 gb gp3
Number of Instances: 3

    
2. Configure the sec groups (allow TCP and ICMP traffic), download the pem file
3. Change to root

`sudo su`

4. List Repos to verify K8s is not installed 

`yum repolist`

5. Install docker as a container runtime

`yum install docker -y`

6. Start Docker as a daemon

`systemctl start docker`

7. Check docker service

`docker ps`

8. Prepare for ***kubeadm*** installation

    
    ```bash
    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    ```
    
9. Install ***kubeadm*** on all 3 nodes

    
    ```bash
    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
    enabled=1
    gpgcheck=1
    gpgkey=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/repodata/repomd.xml.key
    exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
    EOF
    ```
    
10. List repos again and see K8s

`yum repolist`

11. Install ***kubectl*** and enable it (all 3 nodes)

`sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes`
`sudo systemctl enable --now kubelet`

12. Init ***kubeadm*** on **master** node 

`kubeadm init`

13. Export the ***kubeconfig*** file

`export KUBECONFIG=/etc/kubernetes/admin.conf`

14. Join worker nodes to the cluster

    
    ```bash
    kubeadm join 172.30.2.129:6443 --token 6h9hp1.6pfk4ig45iaiesf2 \
            --discovery-token-ca-cert-hash sha256:13754ad087441eb39c2e71754a3153942d9c4ac70f4d0cb4b2cbb843f000896f
           
    ```
    
15. I need to allow  TCP traffic to be able to add the nodes to the cluster (all traffic was allowed for this lab only, please adhere to your company policy)

    
    
16. Install ***calico*** on master node

    
    ```bash
    curl https://raw.githubusercontent.com/projectcalico/calico/v3.30.2/manifests/calico.yaml -O
    
    kubectl apply -f calico.yaml
    ```
    
17. Check NS in the cluster

`kubectl get ns`

18. Create a new NS

`kubectl create ns dev`

19. Create the deployment

`kubectl apply -f HWDep.yaml -n dev`

    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: hello-world-deployment
      labels:
        app: hello-world
    spec:
      replicas: 2 # Number of desired replicas of your application
      selector:
        matchLabels:
          app: hello-world
      template:
        metadata:
          labels:
            app: hello-world
        spec:
          containers:
          - name: hello-world-container
            image: nginxdemos/hello:plain-text # A simple Nginx image serving "Hello World!"
            ports:
            - containerPort: 80 # Port the application listens on inside the container
    ```
    
20. Create the service

`kubectl apply -f HWSev.yaml -n dev`

    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: hello-world-deployment
      labels:
        app: hello-world
    spec:
      replicas: 2 # Number of desired replicas of your application
      selector:
        matchLabels:
          app: hello-world
      template:
        metadata:
          labels:
            app: hello-world
        spec:
          containers:
          - name: hello-world-container
            image: nginxdemos/hello:plain-text # A simple Nginx image serving "Hello World!"
            ports:
            - containerPort: 80 # Port the application listens on inside the container
    ```
    
21. Create the service
    
    
    1. Check the nodes
    

    
22. Test the Application

[`http://13.218.239.9:31943/`](http://13.218.239.9:31943/)
