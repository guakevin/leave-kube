## K8s pentest tools and internal/external intelligence guides

- Docker image for pentest with all tools: `docker.io/tsl0922/ttyd:latest` (create pod from worker-node or : https://github.com/r0binak/MTKPI/blob/main/deploy/mtkpi.yaml)

- Threat Matrix for Kubernetes: https://microsoft.github.io/Threat-Matrix-for-Kubernetes/

### Tools 
* [kube-hunter](https://github.com/aquasecurity/kube-hunter) – a powerful tool for pentesters from Aqua Security
* [kubescape](https://github.com/armosec/kubescape) – identifying cluster misconfigs, RBAC, image scan
* [kdigger](https://github.com/quarkslab/kdigger) – tool for reconnaissance of the environment
* [kubelectl](https://github.com/quarkslab/kdigger) – a custom client for communicating with kubelet
* [peirates](https://github.com/inguardians/peirates) – multi-tool for pentesting a cluster, including from inside the Pod
* [BOtB](https://github.com/brompwnie/botb) – pentest tool for escaping from a container
* Krew plugins (in `$PROJECT_DIR/pentest-tools/krew`) – [access-matrix](https://github.com/corneliusweig/rakkess), [fuzzy](https://github.com/d-kuro/kubectl-fuzzy), [kubesec-scan](https://github.com/controlplaneio/kubectl-kubesec), [node-shell](https://github.com/kvaps/kubectl-node-shell), [sudo](https://github.com/postfinance/kubectl-sudo), [who-can](https://github.com/aquasecurity/kubectl-who-can)

### Privilege escalation 
* [linPEAS](https://github.com/carlospolop/PEASS-ng) – (in `./PEASS-ng/linPEAS`) script for raising privileges in Unix environments
* [LinEnum](https://github.com/rebootuser/LinEnum)
* [LinuxSmartEnumeration](https://github.com/diego-treitos/linux-smart-enumeration)
* [Traitor](https://github.com/liamg/traitor) – a tool for identifying potential vulnerabilities and misconfigurations, which may lead to privilege escalation


##### Links: 
- [Kubernetes_Pentest_All_in_one_The_Ultimate_Toolkit.pdf](https://github.com/r0binak/MTKPI/blob/main/Kubernetes_Pentest_All_in_one_The_Ultimate_Toolkit.pdf)
- [VolgaCTF_2022_Kanibor.pdf](http://archive.volgactf.ru/volgactf_2022/slides/VolgaCTF_2022_Kanibor.pdf)
- [01_zn2021_container_escapes_kubernetes_edition_.pdf](https://luntry.ru/wp-content/uploads/2023/10/01_zn2021_container_escapes_kubernetes_edition_.pdf)
- [Standoff 365. Самое красивое недопустимое событие в деталях](https://habr.com/ru/companies/jetinfosystems/articles/795247/)
- [Как хакеры ломают банки за 48 часов и что нужно для защиты](https://habr.com/ru/companies/pt/articles/801213/)
- Kubernetes Pentest Methodology: [part1](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-1), [part2](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-2), [part3](https://www.cyberark.com/resources/threat-research-blog/kubernetes-pentest-methodology-part-3)
