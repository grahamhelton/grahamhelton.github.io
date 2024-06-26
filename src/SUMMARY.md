[Home - Kubenomicon](./Kubenomicon.md)
- [Initial access](Initial_access.md)
    - [Using cloud credentials](Initial_access/Using_cloud_credentials.md)
    - [Compromised image In registry](Initial_access/Compromised_image_in_registry.md)
    - [Kubeconfig file](Initial_access/Kubeconfig_file.md)
    - [Application vulnerability](Initial_access/Application_vulnerability.md)
    - [Exposed sensitive interfaces](Initial_access/Exposed_sensitive_interfaces.md)
    - [SSH server running inside container](./Initial_access/SSH_server_running_inside_container.md)
- [Execution](Execution.md)
    - [Exec inside container](Execution/Exec_inside_container.md)
    - [New container](Execution/New_container.md)
    - [Application exploit (RCE) 🔗](./Execution/Application_exploit.md)
    - [Sidecar injection](Execution/Sidecar_injection.md)
- [Persistence](Persistence.md)
    - [Backdoor container](Persistence/Backdoor_container.md)
    - [Writable hostPath mount](Persistence/Writable_hostPath_mount.md)
    - [Kubernetes cronjob](Persistence/Kubernetes_cronjob.md)
    - [Malicious admission controller](Persistence/Malicious_admission_controller.md)
    - [Container service account 🔗](Persistence/Container_service_account.md)
    - [Static pods](Persistence/Static_pods.md)
- [Privilege escalation](Privilege_escalation.md)
    - [Privileged container](Privilege_escalation/Privileged_container.md)
    - [Cluster-admin binding](Privilege_escalation/Cluster-admin_binding.md)
    - [hostPath mount 🔗](Privilege_escalation/hostPath_mount.md)
    - [Access cloud resources 🔗](Privilege_escalation/Access_cloud_resources.md)
- [Defense evasion](Defense_evasion.md)
    - [Clear container logs](Defense_evasion/Clear_container_logs.md)
    - [Delete events](Defense_evasion/Delete_events.md)
    - [Pod name similarity](Defense_evasion/Pod_name_similarity.md)
    - [Connect from proxy server](Defense_evasion/Connect_from_proxy_server.md)
- [Credential access](Credential_access.md)
    - [List K8S secrets](Credential_access/List_K8S_secrets.md)
    - [Access node information](Credential_access/Access_node_information.md)
    - [Container service account](Credential_access/Container_service_account.md)
    - [Application credentials in configuration files](Credential_access/Application_credentials_in_configuration_files.md)
    - [Access managed identity credentials](Credential_access/Access_managed_identity_credentials.md)
    - [Malicious admission controller 🔗](Credential_access/Malicious_admission_controller.md)
- [Discovery](Discovery.md)
    - [Access Kubernetes API server](Discovery/Access_kubernetes_API_server.md)
    - [Access Kubelet API](Discovery/Access_kubelet_API.md)
    - [Network mapping](Discovery/Network_mapping.md)
    - [Exposed sensitive interfaces 🔗](Discovery/Exposed_sensitive_interfaces.md)
    - [Instance Metadata API 🔗](Discovery/Instance_metadata_API.md)
- [Lateral movement](Lateral_movement.md)
    - [Access cloud resources 🔗](Lateral_movement/Access_cloud_resources.md)
    - [Container service account 🔗](Lateral_movement/Container_service_account.md)
    - [Cluster internal networking](Lateral_movement/Cluster_internal_networking.md)
    - [Application credentials in configuration files 🔗](Lateral_movement/Application_credentials_in_configuration_files.md)
    - [Writable hostPath mount 🔗](Lateral_movement/Writable_hostPath_mount.md)
    - [CoreDNS poisoning](Lateral_movement/CoreDNS_poisoning.md)
    - [ARP poisoning and IP spoofing](Lateral_movement/ARP_poisoning_and_IP_spoofing.md)
- [Collection](Collection.md)
    - [Images from a private registry](Collection/Images_from_a_private_registry.md)
    - [Collecting data from pod](Collection/Collecting_data_from_pod.md)
- [Impact](Impact.md)
    - [Data destruction](Impact/Data_destruction.md)
    - [Resource hijacking](Impact/Resource_hijacking.md)
    - [Denial of service](Impact/Denial_of_service.md)
- [Fundamentals](./Fundamentals.md)
	- [Nodes](./Fundamentals/Nodes.md)
	- [Services](./Fundamentals/Services.md)
	- [etcd](./Fundamentals/etcd.md)
	- [RBAC](./Fundamentals/RBAC.md)
	- [Kubelet](./Fundamentals/Kubelet.md)
	- [Namespaces](./Fundamentals/Namespaces.md)
	- [Secrets](./Fundamentals/Secrets.md)
	- [Interesting Files](./Fundamentals/Interesting_files.md)
	
[Contributing](./Contributing.md)
[Pentesting Kubernetes](Pentesting%20Kubernetes.md)
