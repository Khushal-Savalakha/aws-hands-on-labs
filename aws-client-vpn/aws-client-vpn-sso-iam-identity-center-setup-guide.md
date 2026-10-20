# AWS Client VPN with IAM Identity Center (SAML SSO)

[![AWS](https://img.shields.io/badge/AWS-Client%20VPN-FF9900?logo=amazonaws&logoColor=white)](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/what-is.html)
[![SSO](https://img.shields.io/badge/IAM%20Identity%20Center-SAML%202.0-232F3E?logo=amazonaws&logoColor=white)](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html)
[![Auth](https://img.shields.io/badge/Auth-Federated%20%2B%20MFA-2E7D32)](#security-checklist)

Set up AWS Client VPN so users sign in with **IAM Identity Center** (SSO + MFA) instead of shared client certificates. Access is controlled by group membership, so onboarding and offboarding is one change.

This lab covers the parts most guides skip: why you need **two SAML apps**, **two IAM identity providers**, and the **same group assigned to both**.

## Table of Contents

- [What you'll build](#what-youll-build)
- [Prerequisites](#prerequisites)
- [How it works](#how-it-works)
- [Step 1: Enable IAM Identity Center](#step-1-enable-iam-identity-center)
- [Step 2: Create users and groups](#step-2-create-users-and-groups)
- [Step 3: Create the server certificate](#step-3-create-the-server-certificate)
- [Step 4: Create the two SAML apps](#step-4-create-the-two-saml-apps)
- [Step 5: Assign the group to both apps](#step-5-assign-the-group-to-both-apps)
- [Step 6: Create the two IAM identity providers](#step-6-create-the-two-iam-identity-providers)
- [Step 7: Create the Client VPN endpoint](#step-7-create-the-client-vpn-endpoint)
- [Step 8: Link the self-service app to the endpoint](#step-8-link-the-self-service-app-to-the-endpoint)
- [Step 9: Connect and verify](#step-9-connect-and-verify)
- [Troubleshooting](#troubleshooting)
- [Security checklist](#security-checklist)
- [References](#references)

## What you'll build

| Component | Name used in this lab | Purpose |
|---|---|---|
| Identity Center group | `dev-test-access` | Who can download the config and connect |
| SAML app 1 | `dev-test-vpn-client-app` | Login when you click **Connect** |
| SAML app 2 | `dev-test-vpn-client-self-service` | Login for the `.ovpn` download portal |
| IAM SAML provider 1 | `dev-test-vpn-client` | Trust record for app 1 |
| IAM SAML provider 2 | `dev-test-vpn-client-self-service` | Trust record for app 2 |
| Server certificate | ACM (imported) | Proves the endpoint is genuine |
| Client VPN endpoint | Full tunnel, federated auth | Routes user traffic through the VPC |

## Prerequisites

| Requirement | Notes |
|---|---|
| AWS account with admin access | IAM, Identity Center, VPC, ACM |
| VPC with a private subnet | Add a NAT gateway if users need internet through the VPN |
| [AWS VPN Client](https://aws.amazon.com/vpn/client-vpn-download/) | Installed on the user's laptop |
| AWS CLI v2 (optional) | For the CLI steps |

## How it works

Clicking **Connect** looks like one step. It's seven, and three of them never touch your browser.

![AWS Client VPN login flow: 7 steps between the VPN Client, browser, IAM Identity Center and Client VPN endpoint](https://github.com/user-attachments/assets/2258c53b-4b8b-49bc-b6b2-5bf5eb0c4158)


| # | Who talks to whom | Where you see it |
|:-:|---|---|
| 1 | **VPN Client → endpoint:** "I want to connect." Client checks the server certificate. | Not in the browser |
| 2 | **Endpoint → VPN Client:** "Here is a login request, open this URL." | Not in the browser |
| 3 | **Browser →** `portal.sso.<region>.amazonaws.com/saml/assertion/...?SAMLRequest=...` | 1st URL |
| 4 | **Browser →** `<region>.signin.aws/platform/d-xxxxxxxxxx/login` (password + MFA) | 2nd URL |
| 5 | **Browser →** `your-org.awsapps.com/start` (portal session set, login finished) | 3rd URL |
| 6 | **Browser →** `http://127.0.0.1:35001` (signed SAML response posted to the VPN Client) | 4th URL |
| 7 | **VPN Client → endpoint:** "Here is the signed response." Endpoint verifies → **Connected** | Not in the browser |

### Why two of everything?

| Question | Answer |
|---|---|
| **Why two SAML apps?** | A SAML app sends its signed result to exactly one address (the **ACS URL**). Client VPN needs two: your laptop (`127.0.0.1:35001`) for Connect, and AWS's self-service portal for the download. |
| **Why two IAM identity providers?** | The endpoint only accepts the ARN of an IAM SAML provider. Each provider holds one app's sign-in URL (hop 2) and signing certificate (hop 7). Two apps → two providers. |
| **Why two portals?** | The **access portal** (`your-org.awsapps.com/start`) is Identity Center's launcher, one tile per assigned app. The **self-service portal** (`self-service.clientvpn.amazonaws.com/endpoints/cvpn-endpoint-...`) belongs to one endpoint and serves its `.ovpn` file. |

## Step 1: Enable IAM Identity Center

1. In the AWS console, open **IAM Identity Center** and click **Enable**.
2. Go to **Settings** and check your **Identity source**:

   | What you see | What it means |
   |---|---|
   | Identity Center directory | Users are created directly in AWS (used in this lab) |
   | Active Directory | AWS Managed Microsoft AD or AD Connector |
   | External identity provider | Entra ID, Okta, Google Workspace or another SAML 2.0 IdP |

3. In **Settings summary**, customize the **AWS access portal URL**, e.g. `https://your-org.awsapps.com/start`.
4. Under **Authentication**, require **MFA at every sign-in**. Client VPN enforces MFA only if Identity Center does.

> [!NOTE]
> Identity Center runs in the Region where you enable it. You'll see that Region in the sign-in URLs.

![IAM Identity Center dashboard](https://github.com/user-attachments/assets/d8e065f1-147a-43cf-b9b2-39b25cbaff7b)

![Settings: identity source and access portal URL](https://github.com/user-attachments/assets/7479641e-fbf1-4e94-9d62-637d0bcf41f4)

## Step 2: Create users and groups

1. **Users → Add user:** fill in username, email, first and last name. The **email becomes the VPN username**.
2. **Groups → Create group:** e.g. `dev-test-access`, then add your users.
3. Open the group and copy its **Group ID**. You need it in [Step 7](#step-7-create-the-client-vpn-endpoint).

> [!TIP]
> Always grant VPN access through groups, never individual users.

![Users in IAM Identity Center](https://github.com/user-attachments/assets/a4571240-4b9f-4b9b-8f70-12c4a3bbd5b0)

![Groups list](https://github.com/user-attachments/assets/92f7d040-a82b-4eab-b40e-a002b5fa3922)

![The dev-test-access group with its member](https://github.com/user-attachments/assets/2f8fb0df-04bf-4e8f-8806-ef8b2c56d1ef)

## Step 3: Create the server certificate

Even with SSO, the endpoint needs a **server certificate** so the VPN Client can verify it in hop 1. Federated auth needs no client certificates.

```bash
git clone https://github.com/OpenVPN/easy-rsa.git && cd easy-rsa/easyrsa3
./easyrsa init-pki                                           # 1. create the key store (pki/)
./easyrsa build-ca nopass                                    # 2. create the CA (ca.crt + ca.key)
./easyrsa --san=DNS:server build-server-full server nopass   # 3. server cert + key, signed by the CA
```

Import it into ACM **in the same Region as the endpoint**:

```bash
aws acm import-certificate \
  --certificate fileb://pki/issued/server.crt \
  --private-key fileb://pki/private/server.key \
  --certificate-chain fileb://pki/ca.crt \
  --region us-west-2
```

> [!WARNING]
> Move `ca.key` and `server.key` off your laptop into a secrets manager or offline storage. Anyone with `ca.key` can mint certificates your VPN Client will trust. Never commit them to this repo.

## Step 4: Create the two SAML apps

In Identity Center: **Applications → Add application → I have an application I want to set up → SAML 2.0**.

| Field | VPN client app | Self-service app |
|---|---|---|
| **Display name** | `dev-test-vpn-client-app` | `dev-test-vpn-client-self-service` |
| **Application start URL** | Leave empty | Endpoint's self-service portal URL (add in [Step 8](#step-8-link-the-self-service-app-to-the-endpoint)) |
| **Relay state, session duration** | Default | Default |
| **Application ACS URL** | `http://127.0.0.1:35001` | `https://self-service.clientvpn.amazonaws.com/api/auth/sso/saml` |
| **Application SAML audience** | `urn:amazon:webservices:clientvpn` | `urn:amazon:webservices:clientvpn` |
| **Metadata file (rename to)** | `vpn-client-metadata.xml` | `vpn-self-service-metadata.xml` |
| **Goes to IAM provider** | `dev-test-vpn-client` | `dev-test-vpn-client-self-service` |

Before saving each app, click **Download** next to **IAM Identity Center SAML metadata file**.

> [!IMPORTANT]
> Rename both metadata files immediately. They look almost identical. If you swap them, Connect sends the login result to the portal website instead of your laptop and the VPN never connects.

### Attribute mappings (both apps)

**Actions → Edit attribute mappings:**

| User attribute in the application | Maps to this string value or user attribute in AWS IAM Identity Center | Format | Used for |
|---|---|---|---|
| `Subject` | `${user:email}` | `emailAddress` | VPN username (required) |
| `Name` | `${user:email}` | `unspecified` | Display name in connection logs |
| `FirstName` | `${user:givenName}` | `unspecified` | User's first name |
| `LastName` | `${user:familyName}` | `unspecified` | User's last name |
| `memberOf` | `${user:groups}` | `unspecified` | Authorization rules (required, case-sensitive) |

![Both SAML apps in the Applications list](https://github.com/user-attachments/assets/1e5787bc-237e-4528-9b17-c904106e8fdf)

![VPN client app: ACS URL, audience and the metadata download](https://github.com/user-attachments/assets/b588797a-f11e-4380-a72f-5cc0391fc9c1)

![Attribute mappings: Subject and memberOf](https://github.com/user-attachments/assets/907b8b28-569f-4f28-ab08-f8401ff519e7)

## Step 5: Assign the group to both apps

Open each app → **Assigned users and groups → Assign users and groups** → add `dev-test-access`.

| Group assigned to | Result |
|---|---|
| VPN client app only | Can connect, but can't download the `.ovpn` file |
| Self-service app only | Downloads the file, then **login is rejected** at Connect |
| **Both apps** | ✅ Works |

In the access portal users see two tiles:

| Tile | What clicking it does |
|---|---|
| `dev-test-vpn-client-self-service` | Opens the endpoint's portal to download the `.ovpn` file |
| `dev-test-vpn-client-app` | Nothing useful from the browser. Used by the AWS VPN Client when you click Connect |

![Assigning the dev-test-access group to the VPN client app](https://github.com/user-attachments/assets/8780f0fb-710d-4a87-a0eb-05dea5b74bc3)

![The group assigned to both apps](https://github.com/user-attachments/assets/f67e8fb7-74e4-4fbd-b61b-d7d03b7e99f6)

## Step 6: Create the two IAM identity providers

**IAM → Access management → Identity providers → Add provider**

| Setting | Provider 1 | Provider 2 |
|---|---|---|
| Provider type | SAML | SAML |
| Provider name | `dev-test-vpn-client` | `dev-test-vpn-client-self-service` |
| Metadata document | `vpn-client-metadata.xml` | `vpn-self-service-metadata.xml` |
| SAML encryption | Leave empty | Leave empty |

> [!NOTE]
> The second file picker under **SAML encryption** is for an optional private key, not your second metadata file.

Or with the AWS CLI:

```bash
aws iam create-saml-provider --name dev-test-vpn-client \
  --saml-metadata-document file://vpn-client-metadata.xml

aws iam create-saml-provider --name dev-test-vpn-client-self-service \
  --saml-metadata-document file://vpn-self-service-metadata.xml
```

Each provider stores the **SSO service location** (the hop 3 URL) and the **signing certificate** the endpoint uses to verify hop 7.

![Both identity providers in IAM](https://github.com/user-attachments/assets/5c804492-ed6c-40ef-9123-1d850f49e0b2)

![Provider details: sign-in URL, issuer and SSO service location](https://github.com/user-attachments/assets/c6f54997-baab-4778-84e3-20343dcbd8a7)

## Step 7: Create the Client VPN endpoint

**VPC → Client VPN endpoints → Create Client VPN endpoint**

| Setting | Value | Why |
|---|---|---|
| Client IPv4 CIDR | e.g. `10.100.0.0/22` | /22 or larger; must not overlap the VPC or users' home networks |
| Server certificate ARN | ACM cert from [Step 3](#step-3-create-the-server-certificate) | Verified by the VPN Client in hop 1 |
| Authentication | **User-based → Federated authentication** | Uses Identity Center |
| SAML provider ARN | `dev-test-vpn-client` | Used when a user clicks **Connect** |
| Self-service SAML provider ARN | `dev-test-vpn-client-self-service` | Used for the `.ovpn` download portal |
| Self-service portal | **Enabled** | Otherwise the portal tile returns an error |
| DNS servers | VPC resolver (CIDR base + 2, e.g. `10.0.0.2`) | Resolves private names; `8.8.8.8` / `1.1.1.1` only for public-DNS labs |
| Split tunnel | **Off** (full tunnel) | All traffic, including internet, goes through the VPC |
| Connection logging | Enabled → CloudWatch Logs | Who connected, when and from where |

> [!CAUTION]
> Never put the same IAM provider in both ARN fields.

![Client VPN endpoints in the VPC console](https://github.com/user-attachments/assets/6281b4fe-061c-420d-91f9-d14475979c69)

![Federated authentication with both provider ARNs](https://github.com/user-attachments/assets/f8660269-0be2-459f-8b1a-2bf591a9e20a)

### Associate, authorize and route

The endpoint starts in **Pending associate**. Configure three tabs:

| Tab | Setting | Value |
|---|---|---|
| Target network associations | Associate target network | Your VPC + a **private subnet** (two AZs for production) |
| Authorization rules | Add authorization rule | Destination `0.0.0.0/0` (full tunnel) or VPC CIDR → **Allow access to users in a specific access group** → paste the **Group ID**, not the name |
| Route table | Create route (full tunnel only) | Destination `0.0.0.0/0` → target the associated subnet |

> [!IMPORTANT]
> For internet over a full tunnel, the associated subnet's route table also needs `0.0.0.0/0` → **NAT gateway**. Traffic then exits from the NAT gateway's public IP.

## Step 8: Link the self-service app to the endpoint

1. Endpoint → **Details** tab → copy the **Self-service portal URL**:
   `https://self-service.clientvpn.amazonaws.com/endpoints/cvpn-endpoint-xxxxxxxxxxxxxxxxx`
2. Identity Center → **self-service app → Edit configuration** → paste it as **Application start URL**.

The access portal now knows which endpoint's `.ovpn` file to give out. Each endpoint has its own portal page because each `.ovpn` file contains that endpoint's hostname.

## Step 9: Connect and verify

1. Sign in to the access portal (`your-org.awsapps.com/start`). You should see both tiles.
2. Click the **self-service** tile and download the client configuration.
3. **AWS VPN Client → File → Manage profiles → Add profile** → select the `.ovpn` file.
4. Click **Connect** and sign in with password + MFA.
5. The browser lands on `127.0.0.1:35001` and the VPN Client shows **Connected**.
6. Endpoint → **Connections** tab shows an active session with your email as username.

With full tunnel, your public IP should change to the NAT gateway's:

| Check | Before connecting | After connecting |
|---|---|---|
| WhatIsMyIP.com | Your ISP's IP and country | NAT gateway IP in the endpoint's Region |
| `curl https://checkip.amazonaws.com` | Your ISP's IP | NAT gateway Elastic IP |

![Before connecting: public IP and location in India](https://github.com/user-attachments/assets/08bb04c2-e121-40d3-9771-bac68a830b5f)

![After connecting: traffic exits from the AWS Region in the US](https://github.com/user-attachments/assets/16b084ba-ded6-4dba-b72c-09374c526cb8)

## Troubleshooting

| Symptom | Likely cause | Fix in |
|---|---|---|
| Login rejected in the browser | Group not assigned to the VPN client app | [Step 5](#step-5-assign-the-group-to-both-apps) |
| Self-service tile shows an error | Self-service portal disabled, or providers swapped | [Step 7](#step-7-create-the-client-vpn-endpoint) |
| Connect posts to a website, not your laptop | Metadata files swapped between apps | [Step 4](#step-4-create-the-two-saml-apps) |
| Connected, nothing reachable | Authorization rule missing or uses group name instead of Group ID | [Step 7](#step-7-create-the-client-vpn-endpoint) |
| Connected, VPC works, no internet | No `0.0.0.0/0` route on endpoint, or no NAT gateway | [Step 7](#step-7-create-the-client-vpn-endpoint) |
| Private DNS names don't resolve | Public DNS server set; use the VPC resolver | [Step 7](#step-7-create-the-client-vpn-endpoint) |
| A service times out | Target security group blocks the endpoint's security group | Security groups |

> [!TIP]
> Hold **Shift** while clicking the VPN client tile in the access portal. A new tab shows the SAML assertion with every group ID your login carries. Compare them with your authorization rules.

## Security checklist

| # | Control | Why |
|:-:|---|---|
| 1 | Require **MFA at every sign-in** in Identity Center | Client VPN inherits MFA from the IdP |
| 2 | Narrow authorization rules per group | `0.0.0.0/0` is fine for a lab; production groups get only the CIDRs they need |
| 3 | Alert on `CreateSAMLProvider`, `UpdateSAMLProvider`, `DeleteSAMLProvider` in CloudTrail | A tampered provider can redirect users to a fake sign-in page |
| 4 | Encrypted CloudWatch log group with retention for connection logs | Audit record of who connected, when and from where |
| 5 | Terminate live sessions on offboarding | No SAML single logout; disabling a user doesn't end an active tunnel |

```bash
aws ec2 terminate-client-vpn-connections \
  --client-vpn-endpoint-id cvpn-endpoint-xxxxxxxxxxxxxxxxx \
  --username user@example.com
```

## References

- [Authenticate AWS Client VPN users with AWS IAM Identity Center](https://aws.amazon.com/blogs/security/authenticate-aws-client-vpn-users-with-aws-single-sign-on/) (AWS Security Blog)
- [SAML 2.0-based federated authentication](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/federated-authentication.html) (AWS Client VPN Administrator Guide)
- [Self-service portal](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-self-service-portal.html) (AWS Client VPN Administrator Guide)
