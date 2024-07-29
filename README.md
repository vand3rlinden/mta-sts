# MTA-STS
MTA-STS (Mail Transfer Agent Strict Transport Security) is a security protocol designed to improve the security of email communication by enforcing the use of TLS (Transport Layer Security) to encrypt email traffic between mail servers. It helps prevent man-in-the-middle attacks and downgrade attacks, where an attacker could intercept or tamper with email messages in transit.

##  Implement an MTA-STS policy for a domain
### Adding the DNS TXT Record for the MTA STS Policy
This `TXT` record is placed at `_mta-sts.example.com` and signals the presence of an MTA-STS policy.
```
_mta-sts.example.com IN TXT "v=STSv1; id=20240728"
```

- `v=STSv1:` Indicates the version of the MTA-STS policy.
- `id=2024072801`: A unique identifier for the policy. This can be a date-based identifier or any other unique string. It must be updated whenever the policy file is changed.

### Host the MTA-STS Policy on Github Pages
1. Create a new [public repository](https://docs.github.com/en/pages/quickstart#creating-your-website) called `mta-sts`
2. Disabling Jekyll, GitHub Pages is powered by Jekyll, a static website generator. However, Jekyll does not serve folders that begin with a dot. In order to do so, we'll add a single empty file called `.nojekyll` to the repo.
3. Creating the MTA-STS Policy in `.well-known/mta-sts.txt` with the folliwing content:
```
version: STSv1
mode: testing
mx: your-mx.com
mx: your-mx.com
max_age: 604800
```

4. Activating [GitHub Pages for the subdomain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-a-subdomain): `mta-sts`

   - Go to your DNS provider and add a `CNAME` record with the following details:
   ```
   Host: mta-sts.example.com
   Value: github-username.github.io.
   ```
   - After adding the record, return to your GitHub repository, click _Settings_ in the repository navigation, and scroll down to _Pages_.
     - Under _Build and deployment_, change the Branch to be the `main` and select `root` as the publishing folder. Click Save, this will activate Github pages.
     - Under _Custom domain_ add `mta-sts.example.com`, this will create a commit that adds a `CNAME` file directly to the root of your repository.

5. Once your custom domain is set up, GitHub will begin the process of issuing an SSL certificate using Let's Encrypt. This process typically takes around 15 minutes but can take up to 24 hours. Once the certificate is issued, the _Enforce HTTPS_ checkbox will be clickable. Please enable this setting if you are allowed to do so. After enabling this setting, the policy is deployed. You can verify this by navigating to:
`https://mta-sts.example.com/.well-known/mta-sts.txt` - you should see the policy, and it should have loaded over `HTTPS`.

## Detailed Explanation of the Policy File
- `version: STSv1`: Indicates the version of the MTA-STS policy.
- `mode: enforce`: Specifies that the policy should be strictly enforced. Other modes are `testing` (send report only) and `none` (do nothing).
- `mx`: Lists each of the MX servers that handle email for your domain. If needed, wildcards are allowd, such as `*.j-v1.mx.microsoft`
- `max_age: 604800`: Specifies the duration (in seconds) that the policy is valid. After this time, the sending MTA should refresh the policy.

In the `mta-sts.txt` file, you should list all MX servers that are used for receiving emails for your domain and that support TLS. This ensures that sending MTAs know which servers to establish secure connections with when delivering emails to your domain.

## Recommendations
- **Start with Testing Mode**: It’s advisable to start with the `testing` mode. This allows you to monitor and log the results without affecting email delivery. It helps you ensure that your MX servers are correctly configured to support TLS before enforcing the policy.
- **Transition to Enforce Mode**: Once you are confident that your MX servers support TLS and that email can be securely delivered, you can switch to `enforce` mode to enhance your email security.
- **Activate TLS Reporting (TLSRPT)**: TLSRPT records are DNS `TXT` records that specify how to report issues with TLS encryption for SMTP. When an email server experiences issues delivering emails securely to another server, it can refer to the TLSRPT record to know where to send the report of the problem.

## Implementation of TLSRPT
1. Log in to your DNS hosting provider's management console.
2. Add a new TXT record with the following details:

| Host                        | Type | Value                                         |
| ----                        | ---  | ---                                           |
| `_smtp._tls.example.com` | `TXT`| `v=TLSRPTv1; rua=mailto:tlsrpt@example.com`      |

## In summary
MTA-STS is a powerful tool for enhancing email security by ensuring the use of encryption for email in transit, thereby protecting against various types of attacks.
