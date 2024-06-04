# kubernetes-upgrade
kubernetes upgrade from v1.28 to v1.29

![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/ab9b1f2f-4eba-4e42-97e7-6455088ce7dc)

**Upgrade preparation**
Note: I will upgrade Kubernetes from v1.28.10 to 1.29.X in this example. 

Please replace the version numbers with those of your existing version and upgrade requirements.

**We need to run the upgrades in the following order.**

  1) Control plane node

  2) Worker Nodes

**Also, we need to upgrade the following components on both the control plane and the worker nodes.**

  Kubeadm
  Kubelet
  Kubectl
  
**Before we get started you should know the cluster node names.**

  Kubectl get nodes	//displays how many cluster nodes and notes it.

**The current version of the components**

  Kubeadm version –o json

  Kubectl describe nodes | grep –i “kubelet version”

  Kubectl version –o json

Upgrade Control Plane Node
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/a6ec9be6-74a7-4580-ad60-4db0b796810e)

**Step 1: unhold kubeadm and Install the latest version**

During the Kubeadm cluster installation process, you would have hold the kubeadm installation to prevent upgrades as the standard instructions. So now you have to unhold for further upgrade.

**Drain the Node to evict all workloads.**

  kubectl drain kmaster --ignore-daemonsets

  Kubectl get nodes	// master node in “Ready, SchedulingDisabled” state

**Now we need to unhold kubeadm.**

  sudo apt-mark showhold kubeadm	// check the hold status of kubeadm, no output means it’s in unhold mode

  sudo apt-mark unhold kubeadm

Check the available latest Kubeadm version in the server using the following command.

  sudo apt-cache madison kubeadm | tac	//will display the current available versions in the server repo

**we can see here, we have 1.28.x version available so, we need 1.29.x version. for that, run the below in all the cluster nodes to update the server repo with the desired upgraded version.**

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt-get update

sudo apt-get install -y kubeadm

**Hold kubeadm package.**

  sudo apt-mark hold kubeadm

**Verify that the download works and has the expected version:**

  kubeadm version –o json

**Verify the upgrade plan:**

  sudo kubeadm upgrade plan

**This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.**

   kubeadm upgrade also automatically renews the certificates that it manages on this node.
 
**Choose a version to upgrade to, and run the appropriate command. For example:**

  sudo kubeadm upgrade apply v1.29.5	//# replace x with the desired patch version.

**Now if you check the kubeadm version, you can see the upgraded version.**

  kubeadm version -o json

**Upgrade Kubelet and Kubectl.**

**Unhold and upgrade kubectl and kubelet using the following commands.**

  sudo apt-mark unhold kubelet kubectl &&  sudo apt-get update && sudo apt-get install -y kubelet kubectl
  
  sudo apt-mark hold kubelet kubectl

**Restart the services.**

  sudo systemctl daemon-reload 
  
  sudo systemctl restart kubelet![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/2e6501b1-e5b9-4c34-9d23-8a3a4149e182)

**uncordon the Node and Verify the Node Status**

Use the following command to uncordon the control plane node

  kubectl uncordon controller

  kubectl get nodes		// now master node should be in Ready state.

You can see the control plane upgraded to the new version and nodes running on the old version as shown below.


Upgrade Worker Nodes
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/af245ccb-949d-45f2-a7c5-4cbbf37ed68c)

Step 1: unhold kubeadm and Install the latest version

During the Kubeadm cluster installation process, you would have hold the kubeadm installation to prevent upgrades as the standard instructions. So now you have to unhold for further upgrade.

Drain the Node to evict all workloads.

  kubectl drain kmaster --ignore-daemonsets
  
  Kubectl get nodes	// master node in “Ready, SchedulingDisabled” state

Now we need to unhold kubeadm.

  sudo apt-mark showhold kubeadm	// check the hold status of kubeadm, no output means it’s in unhold mode
  
  sudo apt-mark unhold kubeadm

Check the available latest Kubeadm version in the server using the following command.
  
  sudo apt-cache madison kubeadm | tac	//will display the current available versions in the server repo

we can see here, we have 1.28.x version available so, we need 1.29.x version. for that, run the below in all the cluster nodes to update the server repo with desired upgraded version.

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
  
  apt-get update
  
  sudo apt-get install -y kubeadm=1.29.5-1.1
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/b1a43ba2-045b-4c7e-beae-46753f5dd70b)


Hold kubeadm package.
  
  sudo apt-mark hold kubeadm

Verify that the download works and has the expected version:

  kubeadm version –o json

Verify the upgrade plan:
  
  sudo kubeadm upgrade plan

This command checks that your cluster can be upgraded, and fetches the versions you can upgrade to. It also shows a table with the component config version states.

kubeadm upgrade also automatically renews the certificates that it manages on this node.

Choose a version to upgrade to, and run the appropriate command. For example:

  sudo kubeadm upgrade apply v1.29.5	//# replace x with the desired patch version.

Now if you check the kubeadm version, you can see the upgraded version.

  kubeadm version -o json

Upgrade Kubelet and Kubectl.

Unhold and upgrade kubectl and kubelet using the following commands.

  sudo apt-mark unhold kubelet kubectl &&  sudo apt-get update && sudo apt-get install -y kubelet kubectl
  
  sudo apt-mark hold kubelet kubectl

Restart the services.
  
  sudo systemctl daemon-reload 
  
  sudo systemctl restart kubelet![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/b673d390-c2e7-4acf-8b82-6947cf2c357a)

  uncordon the Node and Verify the Node Status
  
Use the following command to uncordon the control plane node

kubectl uncordon controller

kubectl get nodes		// now master node should be in Ready state.

You can see the control plane upgraded to the new version and nodes running on the old version as shown below.

![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/0b582403-82ac-42fa-baf1-178495b933ca)

Upgrade Worker Nodes
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/7a9cf562-1e6c-48b2-ad53-760503440d48)

Goto Your master node drain the worker node , run the following command in your master node

kubectl drain kworker1.demo.com --ignore-daemonsets

If pods are using local storage, you might get the error and solution will be in the error message itself.

Worker node upgrade steps are similar to control plane upgrade. However the internal upgrade process differ from control plane.

All the following steps should be executed on the Worker node. If you have multiple worker nodes, upgrade one at a time.

sudo apt-mark unhold kubeadm

sudo apt-get update && sudo apt-get install -y kubeadm=1.29.5

sudo apt-mark hold kubeadm

As you are running the following command on a worker node, it skips all the steps required for the control plane and update only kubelet configuration required for a worker node.

sudo kubeadm upgrade node
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/5e72ae45-0d04-44ab-8d98-ade2ffbc5fc2)


Upgrade Kubelet & Kubectl

sudo apt-mark unhold kubelet kubectl

sudo apt-get update && sudo apt-get install -y kubelet kubectl

sudo apt-mark hold kubelet kubectl

Restart kubelet

sudo systemctl daemon-reload 

sudo systemctl restart kubelet

Uncordon worker node

kubectl uncordon worker1

Now that we have upgraded the control plane node and the worker nodes, lets verify if the cluster is upgraded and working as expected. Get the node information

kubectl get nodes

From the control plane node, you can use any one of the following kubectl commands to check the status of all the cluster components.

kubectl get --raw='/readyz?verbose'
![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/986dc9bb-70eb-4fcf-aa1b-332054a06ba7)

![image](https://github.com/aicloudpost/kubernetes-upgrade/assets/166476986/33ea33ba-548a-4703-a756-a6adfd421ff2)
