
# AWS IAM — Identity and Access Management

## 1. IAM Overview

**IAM (Identity and Access Management)** is a global AWS service used to manage authentication and authorization.

IAM allows administrators to control who can access AWS resources and what actions they can perform.

### Key Concepts

- IAM is a **global service**, not Region-specific.
- The AWS root user is created when an AWS account is created.
- The root user has unrestricted access to the AWS account.
- Avoid using the root user for everyday tasks.
- Never share root account credentials.
- Create individual identities with appropriate permissions.

---

## 2. IAM Users and Groups

### IAM Users

An IAM user represents an identity within an AWS account.

- Users can have individual permissions.
- Users do not have to belong to a group.
- A user can belong to multiple groups.
- Users can receive permissions directly or inherit them through groups.

### IAM Groups

IAM groups are collections of IAM users.

- Groups contain users, not other groups.
- Groups cannot be nested.
- Permissions assigned to a group are inherited by its members.
- Adding users to groups makes permission management simpler.

### Example

**Developers Group**
- Alice
- Bob
- Charles

**Audit Team Group**
- Charles
- David

**Operations Group**
- David
- Edward

**No Group**
- Fred

Charles belongs to both Developers and Audit Team.

David belongs to both Audit Team and Operations.

Fred does not belong to any group.

### Why Use Groups?

Instead of assigning the same permissions individually to every user, administrators can assign permissions to a group.

**Example:**

A company has 10 developers who need access to Amazon EC2.

Without groups:
- Assign EC2 permissions to each developer individually.
- Update permissions for each developer when requirements change.

With groups:
1. Create a `Developers` group.
2. Assign the required EC2 permissions to the group.
3. Add developers to the group.
4. Manage their shared permissions through the group.

**Key Takeaway:**
Groups simplify permission management, reduce administrative overhead, and help maintain consistent access controls.

---

## 3. IAM Permissions

IAM permissions determine which actions users and other identities can perform on AWS resources.

### IAM Policies

- Policies are JSON documents that define permissions.
- Policies can be attached to IAM users, groups, and roles.
- Policies specify which actions are allowed or denied.
- A user can receive permissions from multiple policies.

### Principle of Least Privilege

The principle of least privilege means:

> Give users only the permissions they need to perform their tasks.

**Example:**

A developer needs to view EC2 instances but should not be able to terminate them.

Appropriate permissions:
- `ec2:DescribeInstances`

Unnecessary permissions:
- `ec2:TerminateInstances`
- `ec2:*`

### Why Least Privilege Matters

- Reduces security risks.
- Prevents accidental modifications.
- Limits damage if credentials are compromised.
- Improves access control.

**Key Takeaway:**
IAM policies control access to AWS resources. Always grant the minimum permissions necessary.

---

## 4. IAM Policy Structure

IAM policies are JSON documents composed of specific elements.

### Top-Level Elements

| Element | Description | Required |
|---------|-------------|----------|
| Version | Policy language version, normally `2012-10-17` | Recommended |
| Id | Optional identifier for the policy | No |
| Statement | One or more permission statements | Yes |

### Statement Elements

| Element | Description |
|---------|-------------|
| Sid | Optional statement identifier |
| Effect | Specifies `Allow` or `Deny` |
| Principal | Specifies the identity receiving access; used in resource-based policies |
| Action | AWS API actions allowed or denied |
| Resource | AWS resources affected by the policy |
| Condition | Optional conditions controlling when permissions apply |

### Example IAM Policy

The following policy allows specific read operations on EC2, Elastic Load Balancing, and CloudWatch.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "elasticloadbalancing:Describe*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudwatch:ListMetrics",
        "cloudwatch:GetMetricStatistics",
        "cloudwatch:Describe*"
      ],
      "Resource": "*"
    }
  ]
}
```

### Understanding the Example

**Version**
- `2012-10-17` identifies the IAM policy language version.
- It is not the policy creation date.

**Statement**
- Contains individual permission rules.
- A policy can contain multiple statements.

**Effect**
- `Allow`: Grants permission.
- `Deny`: Explicitly denies permission.

**Action**
- Defines which AWS operations are permitted.
- `ec2:Describe*` matches EC2 actions beginning with `Describe`.

**Resource**
- Defines the resources affected.
- `"*"` means all applicable resources.
- Some AWS actions require `"*"` because they do not support resource-level restrictions.

### Important Permission Rules

- Access is denied by default unless explicitly allowed.
- An explicit `Deny` overrides an `Allow`.
- Permissions can come from multiple policies.
- Resource-based policies may specify a `Principal`.
- Identity-based policies generally do not include a `Principal`.

**Key Takeaway:**
IAM policies use JSON to define who can perform specific actions on particular AWS resources.

---

## 5. AWS Account Alias

An AWS account alias is a custom name that replaces the AWS account ID in the IAM user sign-in URL.

### Default Sign-In URL

`https://123456789012.signin.aws.amazon.com/console`

### With an Account Alias

`https://my-company.signin.aws.amazon.com/console`

### Benefits

- Easier to remember than a 12-digit account ID.
- Provides a recognizable sign-in URL.
- Simplifies access for IAM users.

### Important Notes

- Account aliases must be unique across AWS accounts.
- An AWS account can have only one account alias.
- Creating an alias does not change the AWS account ID.
- An alias does not provide additional security permissions.

**Key Takeaway:**
Account aliases simplify the AWS IAM user sign-in experience without changing account identities or permissions.

---

## 6. AWS Multi-Session Support

AWS Management Console multi-session support allows users to work with multiple AWS console sessions in the same browser.

### Key Features

- Supports multiple active console sessions.
- Allows switching between different AWS accounts or identities.
- Reduces the need to sign out and sign back in.
- Useful for managing development, staging, and production environments.

### Example

A cloud engineer manages three AWS accounts:

- Development
- Staging
- Production

With multi-session support, the engineer can access and switch between these environments without repeatedly signing out.

### Benefits

- Faster account switching.
- Easier administration of multiple AWS accounts.
- Improved productivity when working across environments.

### Important Notes

- Multi-session support must be enabled in the AWS console.
- It supports up to five concurrent sessions per browser.
- Each session retains its own account identity and permissions.
- Multi-session support does not grant additional permissions.

**Key Takeaway:**
Multi-session support simplifies working across multiple AWS accounts while maintaining separate access permissions.

---

## 7. AWS Certification Review

### Essential Concepts

1. IAM is a global AWS service.
2. Avoid using the root user for everyday activities.
3. IAM users can belong to multiple groups.
4. IAM groups cannot contain other groups.
5. Users can exist without belonging to a group.
6. Group policies simplify permission management.
7. IAM policies are written in JSON.
8. Apply the principle of least privilege.
9. Explicit `Deny` takes precedence over `Allow`.
10. Account aliases simplify IAM user sign-in URLs.
11. Multi-session support allows switching between AWS console sessions.

### Practice Questions

**Q1: Is IAM a global or regional service?**

Answer: Global.

**Q2: Can an IAM group contain another IAM group?**

Answer: No. Groups can contain only IAM users.

**Q3: Can an IAM user belong to multiple groups?**

Answer: Yes.

**Q4: Why is assigning permissions through groups recommended?**

Answer: It simplifies administration and maintains consistent permissions across users.

**Q5: What is the principle of least privilege?**

Answer: Grant only the permissions required to perform a specific task.

**Q6: What happens when one policy allows an action and another explicitly denies it?**

Answer: The explicit `Deny` overrides the `Allow`.

**Q7: What does `"Resource": "*"` mean?**

Answer: The statement applies to all resources supported by the specified actions.

**Q8: What does `2012-10-17` represent in an IAM policy?**

Answer: The policy language version.

**Q9: What is the purpose of an AWS account alias?**

Answer: To provide an easier-to-remember IAM user sign-in URL.

**Q10: Does multi-session support allow users to bypass IAM permissions?**

Answer: No. Each session retains its own identity and permissions.

**Q11: What is the difference between authentication and authorization?**

Answer:
- Authentication verifies who you are.
- Authorization determines what you are allowed to do.

**Q12: Can policies be attached directly to IAM users?**

Answer: Yes. However, using groups or roles is often easier to maintain.

---


# AWS IAM: Password Policies and Multi-Factor Authentication (MFA)

## 1. IAM Password Policies

AWS Identity and Access Management (IAM) allows administrators to configure **password policies** to improve account security and reduce unauthorized access.

A password policy defines the requirements IAM users must follow when creating or changing their AWS Management Console passwords.

### Password Policy Settings

AWS supports the following configurations:

- **Minimum password length:** Define the minimum number of characters required.
- **Character requirements:** Require passwords to contain:
  - Uppercase letters (A–Z).
  - Lowercase letters (a–z).
  - Numbers (0–9).
  - Special characters (e.g., `!`, `@`, `#`, `$`).
- **Allow users to change passwords:** Enable IAM users to update their own passwords.
- **Password expiration:** Require password changes after a specified number of days.
- **Password reuse prevention:** Prevent users from reusing a configured number of previous passwords.

### Important Considerations

- Password policies apply to **IAM users**, not the AWS root user.
- Password policies affect AWS Management Console passwords, not programmatic access keys.
- Strong, unique passwords reduce the risk of unauthorized access.
- Password expiration is configurable, but modern security recommendations generally favor changing passwords when compromise is suspected rather than enforcing frequent rotation without a specific reason.

---

## 2. Multi-Factor Authentication (MFA)

**Multi-Factor Authentication (MFA)** is an additional security mechanism that requires users to verify their identities using more than one authentication factor.

IAM users may have permissions to modify configurations, access sensitive information, or delete AWS resources. MFA provides additional protection against unauthorized access.

### How MFA Works

Traditional password-based authentication uses:

- **Something you know:** A password.

MFA introduces an additional factor:

- **Something you have:** A physical security key or an authenticator device.

For example:

```text
IAM User
   |
   v
Enter Password
   |
   v
Complete MFA Verification
   |
   v
Successful Login
```

### Benefits of MFA

The primary benefit of MFA is protecting accounts when passwords become compromised.

If an attacker steals a user's password, the attacker generally cannot authenticate without completing the additional MFA verification.

**Important:** MFA significantly reduces the risk of account compromise, but it does not completely eliminate threats such as phishing, session hijacking, or stolen authentication tokens.

### AWS MFA Recommendations

- Enable MFA for the **AWS root user**.
- Enable MFA for IAM users, particularly those with administrative privileges.
- Prefer phishing-resistant authentication methods when available.
- Maintain secure account-recovery procedures.

---

## 3. MFA Device Options in AWS

AWS supports different MFA technologies, including software authenticators, physical security keys, and hardware OTP tokens.

### 3.1 Virtual MFA Devices

A **virtual MFA device** is an application that generates temporary authentication codes.

Examples:

- Google Authenticator.
- Authy.

These applications commonly use **Time-Based One-Time Passwords (TOTP)**.

#### How TOTP Works

1. AWS and the authenticator application share a secret during device registration.
2. The application generates a temporary numeric code using the secret and the current time.
3. The user enters the code during authentication.
4. AWS validates the code.

**Characteristics:**

- Codes typically change every 30 seconds.
- Multiple accounts can be configured in one authenticator application.
- Codes can be generated without an internet connection after setup.
- Virtual MFA devices eliminate the need for a dedicated hardware token.

### 3.2 Hardware Security Keys (FIDO2/U2F)

A **hardware security key** is a physical device used to authenticate users through cryptographic verification.

Example:

- YubiKey by Yubico.

The screenshots refer to **Universal 2nd Factor (U2F)**, an older authentication standard. Modern AWS authentication also supports **FIDO2/WebAuthn** technologies.

#### How Security Keys Work

1. The user enters their credentials or initiates a supported authentication flow.
2. AWS requests verification through the registered security key.
3. The user interacts with the device, such as touching its button.
4. The device performs a cryptographic authentication operation.
5. AWS validates the response.

**Advantages:**

- Strong protection against phishing.
- No need to manually enter temporary authentication codes.
- Physical possession of the device is required.
- Cryptographic credentials are associated with the legitimate authentication service.

### 3.3 Hardware Key Fob MFA Devices

A **hardware key fob** is a dedicated physical device that generates temporary authentication codes.

Example from the screenshots:

- Gemalto hardware token (now associated with Thales).

Unlike a hardware security key, a traditional hardware OTP token displays a numeric code that the user manually enters.

**Characteristics:**

- Does not require a smartphone.
- Generates one-time authentication codes.
- Requires physical possession of the token.
- Supported models must be compatible with AWS.

### 3.4 Hardware MFA Devices for AWS GovCloud (US)

**AWS GovCloud (US)** consists of isolated AWS Regions designed to help eligible customers address specific U.S. government and regulated workload requirements.

The screenshots identify:

- SurePassID hardware MFA tokens.

These devices provide hardware-generated one-time passwords.

Supported MFA devices and authentication methods can vary, so compatibility should be verified against current AWS GovCloud documentation.

---

## 4. Comparison of MFA Devices

| MFA Type | Example | Authentication Method | Main Advantage |
|---|---|---|---|
| Virtual MFA | Google Authenticator, Authy | Time-based one-time passwords | Convenient and supports multiple accounts |
| Hardware security key | YubiKey | FIDO2/WebAuthn cryptographic authentication | Phishing-resistant authentication |
| Hardware OTP token | Gemalto/Thales | One-time numeric codes | No smartphone required |
| GovCloud hardware token | SurePassID | One-time numeric codes | Hardware authentication option for supported GovCloud environments |

**Key distinction:** Hardware security keys using FIDO2/WebAuthn provide stronger phishing resistance than traditional one-time password methods.

---

## 5. AWS IAM Security Best Practices

### Password Security

- Use long, unique passwords.
- Avoid reusing passwords across accounts.
- Consider using a password manager.
- Configure IAM password policies when IAM users require console passwords.

### MFA Security

- Enable MFA for the root user and human identities.
- Prefer phishing-resistant MFA methods when supported.
- Protect physical MFA devices against loss or theft.
- Follow AWS recovery and backup MFA recommendations.

### Access Management

MFA protects the authentication process, but it does not determine what users can access.

AWS IAM policies control authorization through permissions.

The two mechanisms complement each other:

- **Authentication:** Verifies who the user is.
- **Authorization:** Determines which AWS resources and actions the user can access.

Example:

```text
User
  |
  v
Authentication
(Password + MFA)
  |
  v
Authorization
(IAM Policies)
  |
  v
Access to Permitted AWS Resources
```

An authenticated user can perform only the actions allowed by their applicable AWS permissions.

---

## 6. Key Concepts to Remember

- **IAM password policies** define password requirements for IAM users.
- **MFA** adds additional authentication factors beyond a password.
- **Virtual MFA devices** generate temporary codes using applications such as Google Authenticator.
- **TOTP** generates time-based authentication codes using a shared secret.
- **FIDO2/WebAuthn security keys** use cryptographic authentication and provide phishing resistance.
- **Hardware OTP tokens** generate authentication codes without requiring a smartphone.
- **AWS root users and IAM users** should be protected with MFA.
- **Authentication and authorization are different:** MFA verifies identity, while IAM policies determine permissions.

