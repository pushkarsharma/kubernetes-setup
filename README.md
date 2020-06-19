# kubernetes-setup

<h1>Local Installation</h1>

<h2>KVM/QEMU & libvirt Installation</h2>

[KVM Installation Reference](https://kubernetes.io/blog/2019/03/28/running-kubernetes-locally-on-linux-with-minikube-now-with-kubernetes-1.14-support/#qemu-kvm-and-libvirt-installation)

After installing libvirt, you may verify the host validity to run the virtual machines with virt-host-validate tool, which is a part of libvirt.<br>
```
sudo virt-host-validate
```
#
&#x1F534;[IOMMU Error Reference](https://www.reddit.com/r/linuxquestions/comments/bgbpim/how_to_enable_iommu_on_ubuntu_1804/):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If you see error <strong>`iommu appears to be disabled in kernel. add intel_iommu=on to kernel cmdline arguments`</strong> then follow the following steps:<br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Edit `/etc/default/grub` and add the option `"intel_iommu=on"` to `GRUB_CMDLINE_LINUX` or &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`GRUB_CMDLINE_LINUX_DEFAULT`:<br>
```
GRUB_CMDLINE_LINUX="intel_iommu=on"
```
#
These changes will take effect once you run:
```
sudo update-grub or grub-mkconfig -o /boot/grub/grub.cfg
```
Finally,
```
sudo reboot
```

---
<h1>Cluster Deployement</h1>

<h2>Steps for Deployement</h2>

Follow the steps below for cluster deployement:

1. `chmod +x start.sh kubernetes.sh docker.sh`
2. `sudo su`
3. `./start.sh`
4. Comment out the line which has mention of swap partition in the file that opens (`/etc/fstab`).
5. Add the hostnames of Master & Kubelets VMs in the next opened file (`/etc/hostname`).
6. Add `Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"` in the next opened file (`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`)

<h3>Master VM</h3>

7. Initiate the cluster: `kubeadm init --apiserver-advertise-address=<ip-address-of-kmaster-vm> --pod-network-cidr=192.168.0.0/16`

&#x1F534;**It will give the join statement for kubelet nodes (KEEP IT!)<br>
&#x1F534;The output will ask to execute the following commands (CROSS CHECK ALWAYS):**

8. `mkdir -p $HOME/.kube`<br>
9. `cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`<br>
10. `chown $(id -u):$(id -g) $HOME/.kube/config`<br>
11. Verify the cluster with `kubectl get pods -o wide --all-namespaces`

&#x1F534;**You will notice from the previous command, that all the pods are running except one: ‘kube-dns’. For resolving this we will install a pod network. To install the CALICO pod network, run the following command:**

12. `kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml`

<h3>Kubelet VM</h3>

13. Connect the Kubelet to the cluster using the statement saved from `7` output that looks like: `kubeadm join <IP>:<PORT> --token <TOKEN> --discovery-token-ca-cert-hash <SHA256-VALUE>`
14. Check nodes' health with `kubectl get nodes`.

<h3>Kubernetes cluster is deployed. Yay!</h3>
