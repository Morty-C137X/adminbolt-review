# AdminBolt v1.9.2 Reviewed: Reliable File Recovery, Uneven Diagnostics

**English** · [中文](README.zh.md) · [Read on the website](https://c137morty.cc.cd/reviews/adminbolt/)

> By Morty, an IoT Engineering undergraduate. Tested on a RackNerd 4 GB KVM VPS in Los Angeles, October 16, 2026.

This is my first VPS and my first hosting control panel. That made the most useful test a practical one: could I edit a website file, back it up and recover it without losing either the original or the newer version?

**Disclosure:** The VPS's first year was provided free through the RackNerd–AdminBolt giveaway. AdminBolt was running on an activated 30-day trial; a one-year panel license had not been confirmed at the time of testing. AI assisted with testing and evidence collection. The findings below are based on actual use and independent checks.

## 1. What stood out

**File recovery was the strongest part of the experience.** The recovered test file matched the original exactly, and the two existing website files were unchanged. Better still, restoration automatically preserved the newer version in a separate archive, leaving a fallback if I had chosen the wrong backup.

**Readiness indicators were less dependable as a guide.** DNS Health showed Healthy for a domain that lacked public address records, while the SSL page correctly identified that missing prerequisite. The AI Hub showed Connected, but the assistant could not answer either of two questions. Invalid domain input also produced an internal-error message rather than a useful explanation.

My main reservation is therefore about how the panel communicates readiness and failure. The tested file workflow worked; understanding some of the other status screens required cross-checking.

### Positives beyond recovery

- **Straightforward installation:** RackNerd ctrl's one-click image installed the panel without requiring an SSH session.
- **Readable everyday tools:** File Manager was easy to navigate, Filesystem Backups stated its scope, and the certificate list clearly separated active and disabled SSL.
- **Useful security visibility:** Fail2Ban rules and counters could be inspected within the panel, alongside the other management tools.

| Evaluation | Result | Finding |
| --- | --- | --- |
| File creation and editing | Passed | Panel-created UTF-8 content was saved and served by its virtual host |
| Filesystem backup and restore | Passed | Restored probe and existing files matched expected hashes |
| Backup portability | Passed | Both archives downloaded through File Manager and could be read independently |
| Recovery safeguard | Verified | Automatic before-restore archive preserved revision 2 |
| Account backup readiness | Not configured | Client page explicitly said no policy was assigned and no backups were being made |
| Custom-domain free SSL | Issued in the follow-up | Initial prerequisite checks blocked the request; issuance succeeded after DNS and default-address metadata were corrected |
| DNS Health interpretation | Ambiguous | Healthy did not mean the domain had usable web-address records |
| Invalid domain input | Failed operation; weak feedback | Generic internal error, with no new domain shown |
| Built-in AI | Unavailable in this session | Two distinct explanation prompts failed despite Hub registration |
| Local response samples | Measured locally | Seven login-page requests and seven static-file requests; not a browser speed benchmark |

## 2. How I tested

I tested **v1.9.2, build v1 · f590965**, on AlmaLinux 9.8. The RackNerd VPS has 3 vCPU, advertised 4 GB RAM and 60 GB SSD. It already contained two domain entries and a small default website. Installation used RackNerd ctrl's one-click image; I did not record how long it took.

![Installed panel version](images/07-version-install.png)

I carried out the panel actions in a browser, switching from the administrator session into the existing hosting account. I checked the results separately through SSH file reads, local HTTP requests, public DNS queries and downloaded archives. This covered the client workflow, but not permission isolation between independently signed-in customers.

During the initial controlled tests, the probe was a uniquely named text file, not a replacement homepage. Existing file hashes were recorded before and after recovery. No passwords, API keys, firewall rules or DNS records were changed. Free SSL was attempted once with Force issue off. The probe was moved out of the web root afterwards; the small backups were retained privately. The original interface language was restored after screenshot collection.

I completed the edit-and-restore cycle twice. Both runs produced the same modified and recovered file hashes. The dataset was deliberately small, making this a check of recovery accuracy rather than capacity or throughput.

![Resource dashboard, captured separately from the controlled file test](images/01-dashboard.png)

The VPS used roughly 1.3–1.4 GB of RAM during testing, with about 2.5 GB available. These figures include the operating system and other installed services; they do not isolate AdminBolt's memory use.

## 3. File editing: did the saved content reach the website?

File Manager offered several editor choices. I used TextArea to create a test file containing English and multi-byte UTF-8 characters. After saving, an independent server read found a **129-byte** UTF-8 file, and a request to the site's virtual host returned HTTP 200 with that content. Because the domain had no public address records, I directed this request locally to the configured site. It confirmed delivery from the server, while public reachability remained unverified.

![UTF-8 probe in the panel editor](images/12-file-editor-utf8.png)

I then changed the probe to revision 2. Its saved size became **115 bytes**, its hash changed, and the virtual host served the changed content. The editor, the saved file and the website response agreed in both cases.

![Different revision saved before recovery](images/14-file-mutated-before-restore.png)

For this small UTF-8 file, the editing workflow behaved as expected. Large uploads, concurrent editing, line-ending preservation, permission changes and other editor modes remain untested.

## 4. Backups: accurate recovery and a useful safety copy

### Account backup was not ready

The client Backups page made an important distinction: backups were included in the plan, but no policy had been assigned, so none were being made. That warning was useful. Without it, an included feature could easily be mistaken for active protection.

![Account backup policy prerequisite](images/11-backup-policy-prerequisite.png)

### File backup worked without that account policy

The separate Filesystem Backups page explicitly limits its scope to domain-root files, excluding databases and server configuration. It created a manual archive containing the 129-byte probe plus the existing `index.php` and `.user.ini` files.

![Created filesystem archive and its stated scope](images/13-filesystem-backup-created.png)

I restored this archive after saving revision 2. The confirmation dialog was generic: “Are you sure you would like to do this?” It did not identify the selected backup or explain what would be replaced. Showing the domain, backup time and overwrite behavior at this point would make the decision easier to check.

![Actual restore confirmation](images/15-backup-restore-confirmation.png)

After restoration, the file contained revision 1 again and its SHA-256 matched the original. Both pre-existing website files also retained their original hashes. I confirmed the recovered content by reopening it in the editor and downloading a copy.

![Recovered revision reopened in the editor](images/16-file-restored-verified.png)

![Verification derived from actual file reads and archive inspection](images/28-recovery-content-verification.png)

### A fallback for the restore itself

The account's backups directory contained a second archive labelled `before-restore`. This was not a second backup I manually requested. Downloading and inspecting it showed revision 2—the state immediately before restoration. The manual archive was 4,553 bytes; the safety archive was 4,508 bytes. Both were readable gzip/tar archives with the expected three files.

![Manual and automatic before-restore archives](images/22-automatic-before-restore-snapshot.png)

This was the most thoughtful detail in the workflow: restoring an older version did not discard the current one without a fallback. The protection is still limited to website files on the same VPS. It excludes databases and server configuration; off-site protection, larger restores, retention and deletion handling need separate evaluation.

The page description mentioned downloading backups, but the visible table exposed Restore and Delete actions. I downloaded the archives through File Manager. A Download button beside Restore and Delete would make this useful capability much easier to find.

## 5. DNS and SSL: useful errors, ambiguous health status

### Clear certificate status

The temporary site had a ZeroSSL certificate, while the custom domain showed SSL Disabled. The list made it easy to see which site already had a certificate and which still needed one.

![Existing certificate versus the custom domain without SSL](images/18-custom-domain-ssl-disabled.png)

### A certificate failure that explained the next step

I attempted Enable Free SSL once, leaving Force issue off. The operation returned an explicit error: the domain had no public A or AAAA record. It suggested an A record and included a correlation ID. An independent Google Public DNS-over-HTTPS query returned NOERROR but **zero A answers and zero AAAA answers**, consistent with the missing-address diagnosis.

![Normal issuance, with Force issue left off](images/19-ssl-preflight-force-off.png)

![Actionable certificate failure, with private identifiers masked](images/20-ssl-failure-actionable-message.png)

The refusal was appropriate: the domain was not ready for issuance. The panel identified the missing prerequisite rather than returning an unexplained failure. Successful issuance was verified later in the publication follow-up below; automatic renewal remains untested.

After the VPS's IP change, the suggested A-record address and client Shared IP field still showed the previous address. The server was already being managed at its replacement address. I left these settings unchanged during the test, but the mismatch meant the suggestion needed checking before use. This is a finding about this post-replacement setup, not a conclusion about fresh installations.

### DNS Health needed a clearer meaning

DNS Health displayed Healthy for both domains. For the custom domain, that appeared alongside the independent no-address result and the SSL workflow's missing-address error.

![DNS Health displayed Healthy](images/17-dns-health-healthy.png)

**Healthy was not sufficient to establish website readiness.** The page did not explain its checks clearly enough to reconcile that status with the certificate error. Listing the checks performed and the results for each record type would give users a more useful basis for deciding what to do next.

## 6. Error handling: a deliberately invalid domain

I submitted `not a domain` as a Domain-type entry. The operation failed, the domain list remained at two entries, and a server-side check found no corresponding top-level account directory. The displayed error was “Domain not created: The operation could not be completed because of an unexpected internal error.”

![Domain list remained at two entries after rejection; the transient error is transcribed above](images/23-invalid-domain-generic-error.png)

The operation was rejected, but the message made a simple formatting mistake look like a server fault. A field-level explanation would be more useful and would save users from troubleshooting the wrong problem. This was a single invalid-input case; duplicate domains, internationalized domains and path boundaries were not tested.

## 7. AI and integrations: connection status versus a working assistant

I submitted two distinct read-only prompts in the client assistant: explain filesystem versus account backup, and give a checklist for diagnosing website certificate failure. Both returned “AI Assistant is currently unavailable. Please try again later.” No configuration changes were requested.

![Two different read-only questions, both unavailable](images/21-ai-two-readonly-prompts.png)

In the administrator Agents area, the AI Agent tab displayed Connected and said the panel was registered with the external AI Agent Hub. The page identified the vendor's external Hub and reported registration as complete. It did not explain why the assistant could not answer. The cause remained unresolved; this test does not establish whether licensing, the Hub or another part of the request path was responsible.

![AI Hub registration status](images/25-ai-hub-registered.png)

The separate Bolt Agent tab reported a connected local management agent. After its Test Connection action completed, the page still showed Connected and valid configuration. This was a separate check of the local management agent. Its connected status did not resolve the AI assistant's failure.

![Local Bolt Agent status after the connection test](images/26-bolt-agent-connection-tested.png)

The installation also exposed API-key pages, and the official Integrations page listed WHMCS, Blesta and Upmind, including setup links and API-based account-management descriptions. I did not issue a new token, execute an API write, validate permission scopes or confirm an official MCP server.

![Official billing and automation integrations](images/24-official-integrations.png)

For the assistant, a check that actually returns a response would be more useful than registration status alone. A specific error category or support reference would also make failures easier to investigate.

## 8. Response times and access checks

To measure the server without long-distance network transit, I made seven sequential loopback requests to the HTTPS login page, using a new curl connection each time. All returned HTTP 200 with certificate verification succeeding. Median total time was **378.00 ms**, range **339.59–418.33 ms**.

Seven local requests to the 129-byte static probe all returned HTTP 200, with a **1.09 ms** median and **0.94–2.27 ms** range. The two sets describe different protocols and workloads, so they should be read separately: one checks static delivery, the other measures the login page.

![Actual local response samples, shown on separate scales](images/27-panel-local-response-samples.png)

These local samples exclude browser rendering, signed-in pages and long-distance network transit; they cannot predict the user's experience from distant regions or under load. A separate unauthenticated request to the administrator dashboard returned HTTP 302. That is only a basic access-gate observation, not a penetration test or proof of account isolation.

![Earlier login-protection rules and counters](images/05-firewall.png)

The panel also displayed Fail2Ban rules and counters. Some hardening had already been performed outside AdminBolt, so those settings are context rather than evidence of the panel's default protection. They were left unchanged throughout testing.

## 9. Verdict

**The file-management and small-site file-recovery path earned the strongest evidence.** Writes reached the server, recovery restored the expected bytes, downloadable archives were usable independently, and the automatic before-restore copy preserved a fallback version. That combination gave me a concrete reason to trust this workflow for the small set of files tested.

The main shortcomings were fragmented readiness signals and uneven failure messages. Certificate preflight was specific and useful; DNS Health and AI registration could appear positive without proving the relevant task worked. Invalid domain input was rejected with an unhelpfully generic error.

My priorities for improvement would be:

1. Define health indicators precisely and connect DNS, network-address metadata and certificate prerequisites.
2. Give restores a specific impact summary while explaining the automatic safety copy, backup scope and off-site limitations.
3. Replace generic input failures with field-level diagnostics.
4. Add an end-to-end AI health check distinct from Hub registration.
5. Make archive download and API-permission documentation easier to discover.

I intend to keep using this VPS for IoT and student project websites, with AI agent and MCP experiments alongside them. For that use, the file tools provide a useful starting point; clearer diagnostics would make the panel easier to rely on as those projects grow.

This assessment covers one version on one VPS. Database recovery, the WordPress lifecycle, mail delivery, API authorization boundaries, larger datasets and certificate renewal still need testing. Thank you to Dustin, RackNerd and AdminBolt for making this hands-on evaluation possible.

## 10. Publication follow-up: the review now runs on this VPS

After the controlled tests, I published this review as a bilingual static website on the same VPS, using the custom domain already created in AdminBolt. The pages were generated locally and uploaded over SSH; the domain configuration and certificate issuance were handled through the panel.

This added a useful real-world check. Once public A records pointed to the current VPS address, normal certificate issuance still failed: AdminBolt's default IP record retained the previous address and the validation message asked me to use it. I registered the current address under Network IPs and made it the default, leaving the old record in place. The client dashboard then showed the current shared address. A second normal request succeeded, with Force issue left off throughout.

![The custom website now has a ZeroSSL certificate; private identifiers are masked](images/29-website-ssl-issued.png)

The panel listed the new certificate as ZeroSSL, expiring January 14, 2027, with automatic renewal enabled. Independent HTTPS requests to the root and www addresses passed certificate verification and returned HTTP 200 before redirect configuration. The site was then set to redirect ordinary HTTP and www requests to the main HTTPS address, while leaving the certificate-validation path available.

This extends the earlier SSL finding: successful issuance is now verified after both public DNS and the panel's default-address metadata are correct. The missing-record error and the later stale-address rejection were useful safeguards, although an explicit reminder to update the panel after a provider-side IP replacement would have saved troubleshooting. Automatic renewal remains untested.

The published site includes English and Chinese versions, expandable screenshots, verification records and a downloadable review pack. It also turns this review into an ongoing use of the hosting panel, rather than ending with the test session.

## Evidence

[Raw verification records](evidence/) include file hashes, both archive inventories, the downloaded-file comparison, public DNS answers and every local timing sample. Archive contents and credentials are not included. The illustrations comprise 20 English-language interface captures with private information masked, plus two charts derived from the verification data.
