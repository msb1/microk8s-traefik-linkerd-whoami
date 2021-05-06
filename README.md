<h2>microk8s-traefik-linkerd-whoami</h2>
<h4>A MicroK8s implementation of Traefik Ingress Controller with Linkerd Service Mesh in VirtualBox Guest Ubuntu VM</h4>
<p>
There is considerable confusion and lack of good documentation for the implementation of Traefik Ingress Controller with Linkerd Service Mesh within MicroK8s on a bare metal
install on VirtualBox Guest Ubuntu OS. This demonstration includes metallb BGP load balancer and revisions to netplan yaml file to enable connections to the Traefik Ingress Controller from Host and connected environments (through Host-Only Network Adapter)
</p>
<h4>Notes for installation and configuration:</h4>
<ol>
<li>Install MicroK8s with Snap in Ubuntu: sudo snap install microk8s --classic </li>
<li>May need to change firewall settings: see https://microk8s.io/tutorials </li>
<li><p>Edit ~/.bashrc file to include:</p>
<ul>
<li>export KUBECONFIG="/var/snap/microk8s/current/credentials/client.config"</li>
<li>alias kubectl='microk8s kubectl' so that kubectl can be used with microk8s (as in minikube and k8s)</li>
</ul>
<p>After editing run "source .bashrc" or reboot to enable changes 
</li>
<li>Set up MicroK8s with Add Ons: microk8s enable dashboard dns linkerd metallb registry storage traefik </li>
<li>When metallb is being installed, it will be necessary to add IP addresses for load balancer. These MUST addresses that are unused by VirtualBox Host-Only Adapters but are allocated for the Host-Only Adapter (in the VirtualBox Manager, see Networks --> Properties)</li>
<li><h5>Install Kubernetes Services (Traefik and Whoami), then Deply and add Ingress Routes with Traefik Custom Resource Definitions</h5>
<ul>
<li><h6>Traefik Custom Resource Definitions with RBAC:</h6> kubectl apply -f 01-traefik-crd.yaml</li>
<li><h6>Services with Traefik Ingress as LoadBalancer:</h6> kubectl apply -f 02-service.yaml</li>
<li><h6>Deployments:</h6> kubectl apply -f 03-deploy.yaml</li>
<li><h6>Ingress Routes for Traefik Ingress Controller, Traefik Dashboard and Whoami:</h6> kubectl apply -f 04-ingress-routes.yaml</li>
</ul>
</li>
<li>Since metallb load balancer is enable, an external IP will be created for Traefik Ingress Controller - use 'kubectl get services' to show IP Address for Traefik Ingress Controller </li>
<li>Check that pods are running and deployments are successful: 'kubectl get pods' and 'kubectl get deployments'</li>
<li>Inject linkerd service mesh into all deployments in current (default) namespace: 'kubectl get deploy -o yaml | microk8s linkerd inject - | kubectl apply -f -'</li>
<li>From a separate linux terminal, run Linkerd Dashboard: 'microk8s linkerd dashboard'</li>
<li><h5>Setup second IP address for Host-Only Network adapter enp0s3</h5></li>
<ul>

<li>For Virtualbox VM's need IP address assigned by metallb to be in VM Host-Only Virtual adapter IP range...
Leave some spare IP's between highest last number and metallb IP's...</li>

<li>metallb will assign IP addresses to any LoadBalancer service - in this case Traefik Ingress Controller...
edit netplan yaml file in '/etc/netplan' directory - file in this Guest VM is '01-network-manager-all.yaml'</li>

<li><p>use the following to add 2nd static ip for enp0s3 - this 2nd IP is Traefik Ingress Controller - loadbalancer - IP</p>
<pre><code>
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: false
      optional: true
      addresses: [192.168.xxx.6/24, 192.168.xxx.12/24]
    enp0s8:
      dhcp4: true
</code></pre>
<li>The first enp0s3 address should be the existing Host-Only adapter address (from ifconfig or ip addr) and the second address should be the address assigned by metallb to the Traefik Ingress Controller...</li>
<li>If enp0s8, a second Host-Only adapter is present, then it should be configured to allow dhcp (and NetworkManager) to determine its setup.</li>
<li>After changes to this file "sudo netplan try" or "sudo netplan apply"<li>
</ul>

<li>From VirtualBox Host (can be Windows or Linux), ping both addresses for enp0s3 adapter to ensure the config is working properly</li>
<li>From any web browser in VirtualBox Host OS or Guest Ubuntu OS, enter "192.168.xxx.12:8080/dashboard" to see Traefik Dashboard</li>
<li>From any web browser in VirtualBox Host OS or Guest Ubuntu OS, enter "192.168.xxx.12/whoami" to test Whoami App</li>
</ol>
<h5>This is a basic implementation that demonstrates Traefik Ingress Controller with Linkerd Service Mesh on MicroK8s running on an Ubuntu Virtual Box Guest VM. This demonstration should enable implementation of more complex applications within MicroK8s</h5>