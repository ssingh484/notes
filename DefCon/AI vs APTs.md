Aikido research

Co founder of Conpago
@advocatemack

# Understanding teh supply chain

70-90% of your code comes form open-source
Transitive dependencies
Flaw in SCA

Vuln Found -> CVE Created -> SCA Tool -> Application

Undisclosed, Undiscovered vulns have no feed into this
### ~~Silent~~ patching:
- Fear of exploitation
- Reputation concerns
- Lack of resources
- Belief issue is minor

### Vulnerability bottleneck a la NIST



# Discovering Undisclosed Vulns

## TOOL: Aikido Intel github.com/aikidosec/intel

- Leveraged LLMs to make open source vuln database
- Monitoring Changelogs, identifying issues in changes
- Not all changelogs are on GitHub
- Human-in-loop verification hard and time consuming

### Architecture:
- 1: Scrape sources
- 2: LLM used to normalize changelog formats
- 3: LLM to identify flaws
- 4: Using

Found 550 undisclosed CVEs in 2024 with 48 crits
67% of those never disclosed/No CVE number
Eg: Axios Proto pollution (No CVE)
Eg: Craftcms/cms no CVE (Many CVEs in this project)
Eg: tecnickcom/tcpdf
Eg: mariadb - improper cert validation (highly exploitable)

**Aikido registering as a CNA**
# Discovering Malware

1. Used rule based toools to scan and detect malware
2. Supply this to LLMs to ingest data and make a determination of malware
3. Human reviewer

False positives create too much noise due to HUGE database of versions

### Architecture

1. Scrape sources for public registries
2. Scan for malware using open grep
3. LLM triage

+4000 package **versions** detected in June 2025

### Win Stories

Published on RSS feed and on GitHub
Fully public

#### The stupidest
Lazarus group malware react-html2pdf.js
Used whitespace to hide axios based payload
crypto steal

#### Biggest potential impact
xrpl.js is the ripple ledger library/SDK
1 version had a small change - v 4.2.4
NPM token stolen from dev, change made directly on NPM without GitHub push
Full writeup on blog

#### Most beautiful
os-info-checker-es6

decoding a b64 string - only 1 character? (No, Unicode PUA chars invisible in editors)
Another b64 string in stage 1, which was a google calendar link
This link was also titled a b64 string
finally stage

### TOOL: akido safe-chain NPM package

### GAME for Devs: Imposter
