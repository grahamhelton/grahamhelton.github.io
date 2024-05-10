# Application credentials in configuration files
Accessing application credentials is not a Kubernetes specific issue, however, credentials used in a Kubernetes cluster may be visible through manifests. Most notably, gaining access to an Infrastructure as Code repository could lead to sensitive information being identified from manifests.

Additionally, Kubernetes ConfigMaps are frequently used to pass information to a pod. This can be in the form of configuration files, environment variables, etc.

In this example, information is passed via a ConfigMap to a Pod running postgres which sets the environment variables `POSTGRS_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `PGDATA`. While ConfigMaps are not *supposed*  to be used for sensitive information, they still can be used to pass in information such as passwords.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: ecommerce
    tier: postgres
data:
  POSTGRES_DB: prod 
  POSTGRES_USER: prod 
  POSTGRES_PASSWORD: 123graham_is_SO_cool123 
  PGDATA: /var/lib/postgresql/data/pgdata
```

After a config map is created, it can be referenced by a manifest by using `- configMapRef` which will link the config map to the Pod.

```yaml
apiVersion: v1
kind: Pod 
metadata:
  name: postgres 
spec:
  containers:
  - name: postgres
    image: postgres:latest
    envFrom:
      - configMapRef:
          name: postgres-config
```

Once inside the pod, environment variables passed in via ConfigMaps can be listed with `env`.

![](../images/Pasted%20image%2020240331145655.png)

Beyond ConfigMaps, searching for potentially sensitive strings such as `PASSWORD=`, is worthwhile. A tool like [Dredge](https://github.com/grahamhelton/dredge) can be used for this. 

![](../images/Pasted%20image%2020240331152414.png)

# Defending
Safeguarding Kubernetes secrets used in Infrastructure as Code (IaC) is crucial to maintaining the security and integrity of your infrastructure.  

**Fundamental Defense**

**Understanding the Importance of Secret Management**

In IaC, secrets such as API keys, passwords, and certificates need to be managed securely to prevent unauthorized access and breaches. Hardcoding these secrets in your codebase can expose them to risk, especially when the code is stored in version control systems. Never commit secrets to version control. Even with .gitignore, risks can arise from previous commits or configuration errors.

**Using Environment Variables**

Environment variables are a common method to separate secrets from code. They allow you to keep sensitive information out of your source code.

**Advantages**:
- **Separation of Code and Secrets**: By storing secrets in environment variables, the IaC scripts can be written without hardcoding sensitive information.
- **Flexibility**: Different environments (development, staging, production) can have their own set of values without changing the code.

**Implementation Tips**:
- Use environment variables to store secrets at runtime. For instance, in a CI/CD pipeline, configure these variables in the secure settings of your build server or CI/CD environment.
- Ensure that access to these environment variables is restricted based on roles within the team to minimize exposure.

**Using a Secret Management Tool**

Secret management tools such as HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, or Google Cloud Secret Manager provide more robust solutions for handling secrets.

**Advantages**:
- **Centralized Secrets Storage**: All secrets are stored in a centralized location, making management and auditing easier.
- **Access Controls and Auditing**: These tools offer detailed access controls and auditing capabilities to track who accesses what secret.
- **Dynamic Secrets**: Some tools can generate dynamic secrets that are valid for a single session or are short-lived, reducing the risk if they are exposed.

**Implementation Tips**:
- Integrate your IaC tooling with the secret manager. For example, Terraform can use provider configurations to pull secrets directly from HashiCorp Vault or AWS Secrets Manager.
- Use policies to automate the rotation of secrets and ensure that they are never hard-coded or logged.

**Integrating Environment Variables with Secret Managers**

Combine environment variables and secret managers by storing the access keys or tokens required to access the secret manager in environment variables. The actual secrets are fetched dynamically from the secret manager at runtime.

**Workflow**:
- Set up environment variables to hold minimal access configurations needed to authenticate to the secret manager.
- Configure your IaC scripts or applications to pull secrets from the secret manager dynamically during deployment or runtime.

**Advanced Defense**

Here’s a deeper dive into how engineers can effectively implement additional defenses:

**Use Encryption**

**Why It's Important**:
- **Security in Transit and at Rest**: Encryption ensures that secrets are secure both when they are stored (at rest) and when they are being transmitted (in transit) between systems.
- **Prevent Unauthorized Access**: If sensitive data were to be exposed or accessed inadvertently (e.g., via a breach of source control), encryption would prevent unauthorized users from reading the data.

**How to Implement**:
- Tools like **git-crypt** or **Blackbox** can be integrated into your version control system to automatically encrypt sensitive files when they are committed and decrypt them for authorized users.
- **Implementation steps**:
  - **git-crypt**: Initialize git-crypt in your repository and configure which files to encrypt using a `.gitattributes` file. Provide keys only to authorized team members.
  - **Blackbox**: Set up Blackbox to manage access to encrypted files, using GPG keys of authorized team members. Files are encrypted with these keys and can only be decrypted by users with the corresponding private key.

**Regularly Rotate Secrets**

**Why It's Important**:
- **Minimize Damage from Leaks**: If a secret is compromised, regular rotation limits the time window in which an attacker can use the secret.
- **Compliance**: Regular rotation of secrets may be required to comply with industry standards and regulations.

**How to Implement**:
- Use a secret management tool (e.g., HashiCorp Vault, AWS Secrets Manager) that supports automatic rotation of secrets.
- **Implementation steps**:
  - Set up policies within the secret manager to automate the lifecycle of secrets, including creation, rotation, and retirement.
  - Integrate these tools with your applications and IaC scripts to dynamically retrieve the latest versions of secrets.

**Audit and Monitor**

**Why It's Important**:
- **Detect Breaches**: Continuous monitoring can help detect unauthorized access to secrets or unusual access patterns that might indicate a breach.
- **Maintain Operational Integrity**: Regular audits ensure that only authorized access to secrets is happening and help maintain compliance with security policies.

**How to Implement**:
- Enable logging and monitoring features in your secret management tools.
- Use SIEM (Security Information and Event Management) systems to aggregate log data and generate alerts based on anomalies.
- **Implementation steps**:
  - Configure alerts for unusual access patterns (e.g., access at unusual times, frequent access).
  - Regularly review audit logs and adjust monitoring tools to improve detection capabilities.

**Least Privilege**

**Why It's Important**:
- **Reduce Attack Surface**: Limiting access to secrets to only those who need it minimizes the potential for internal or external attacks.
- **Prevent Accidental Misuse**: By restricting access, you also reduce the risk of accidental misuse of secrets by personnel.

**How to Implement**:
- Define roles and responsibilities clearly in your organization to determine who needs access to what secrets.
- Use role-based access control (RBAC) in your secret management systems to enforce these roles.
- **Implementation steps**:
  - Regularly review and update access permissions to ensure they are still appropriate.
  - Employ automated systems to manage and enforce access controls based on personnel changes or role modifications.


