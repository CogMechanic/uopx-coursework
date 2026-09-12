

Notes from a Windows Server AD lab simulating a small company (three departments, nine employees) needing structured user management and department-specific security controls.

---

## Organizational Units (OUs)

An OU is a **container within Active Directory** used to group objects (users, computers, groups) — typically mirroring a company's real structure, like separate OUs for Marketing, Operations, and Accounting.

The key thing an OU actually provides is **two related capabilities**:

1. **Policy targeting** — Group Policy Objects (GPOs) can be linked to a specific OU, so settings apply only to the accounts/computers inside it.
2. **Delegated administration** — control over an OU (e.g., "who can reset passwords for this group") can be handed to a specific admin or team without giving them rights over the entire domain.

**Important distinction I initially got a little fuzzy on:** the OU itself doesn't enforce security directly — it's the _container_ that makes targeted policy enforcement possible. The actual restrictions come from GPOs linked to the OU, not from the OU structure alone. Segregating departments into separate OUs is what _enables_ department-specific security policy, rather than being the security control itself.

## Group Policy Objects (GPOs)

A **GPO** is a defined set of configuration settings that can be linked to a domain, site, or (most granularly) a specific OU. Unlike domain-wide policy, a GPO linked to a single OU applies **only to the accounts and computers inside that OU** — this is what let the lab scope a stricter policy (`AccountingPolicy`) to just the Accounting department, without affecting Marketing or Operations at all.

**Two Accounting-specific restrictions configured in this lab, and why each matters:**

- **Prohibit access to Control Panel and PC settings** — reduces attack surface and limits what a compromised or careless account can do at the system level. If an Accounting workstation gets compromised (e.g., via phishing, given how frequently finance departments are targeted for exactly that reason), the blast radius is smaller if the user account itself never had system-configuration access to begin with. This is an application of **least privilege** — restrict by default, only grant what's actually needed for the job function.
- **Deny all access to removable storage classes** — directly addresses **data exfiltration risk**. Financial data is a common target for insider threats or malware trying to copy sensitive files off the network via USB; blocking removable storage at the policy level closes that specific channel without relying on user judgment.

## Domain-Wide Account Security Policy

Configured separately from the OU-specific GPO — these settings apply at the **domain level**, meaning every account across all departments:

|Setting|Value|What it defends against|
|---|---|---|
|Minimum password length|10 characters|**Brute-force / offline hash cracking** — longer minimum length increases the keyspace an attacker has to search exponentially, making both online guessing and offline cracking of a stolen password hash significantly slower|
|Account lockout threshold|5 failed attempts|**Online brute-force / password spraying** — locks the account before an attacker can grind through repeated guesses directly against the login prompt|
|Account lockout duration|20 minutes|A deliberate **balance**, not just "more is better." Too short and it barely slows a determined attacker; too long and the _lockout policy itself_ becomes a denial-of-service risk — an attacker could intentionally trigger lockouts to lock real employees out of their own accounts. 20 minutes is a practical middle ground: real inconvenience for an attacker, limited disruption for a legitimate user who mistypes a password.|

## Completion Note

Redid this lab fully from memory on a second pass — no walkthrough, no notes open — creating the OUs, adding users to each, building the Accounting-specific GPO (Control Panel restriction + removable storage denial), and configuring the domain-wide password/lockout policy. Confirms the concepts above weren't just copied down — the actual reasoning (why Accounting gets Control Panel and removable-storage restrictions specifically: locking out access to system settings and storage devices so non-technical staff can't accidentally or intentionally misconfigure machines or exfiltrate data) held up without needing to look anything up.

## Bigger-Picture Takeaway

The real concept tying this whole lab together is **least privilege and segmentation**, applied at two different layers:

- **Account/password policy** (domain-wide) protects _who can authenticate as a given user_ in the first place.
- **OU + GPO structure** protects _what an authenticated user is actually allowed to do_, scoped to their specific role/department.

Both layers matter independently — strong password policy doesn't help if a compromised low-privilege account still has full system access, and tight GPO restrictions don't help if the account itself is trivially crackable. Real AD hardening requires both working together, not either one in isolation.