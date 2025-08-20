
![[PXL_20250809_194156852.jpg]]

# GitHub Actions

Native CICD in GitHub 
## Workflow
## Event

## Jobs

## Runners

# Common Risks

- hard coded secrets
- inadequate secret redaction
- secrets in logs
- 3rd party actions (untrusted third party actions)
	- pin to a sha instead of a version
	- ![[PXL_20250809_194703088.jpg]]
- over privileged tokens usage
- ![[PXL_20250809_194815725.jpg]]
- script or code injection (directly executing pr components etc)
- ![[PXL_20250809_194909944.jpg]]

	- Misuse of pull_request_target
	- ![[PXL_20250809_194954101.jpg]]
	- misconfigured self hosted runners
	- ![[PXL_20250809_195050828.jpg]]

# Real world tj-actions attack
![[PXL_20250809_195451063.jpg]]

# Case Study of CVE-2020-15228
![[PXL_20250809_195458300.jpg]]

# Hardening actions 
![[PXL_20250809_195654820.jpg]]
![[PXL_20250809_195838809.jpg]]
![[PXL_20250809_195903910.jpg]]

# Security Tooling

## ActChain

![[PXL_20250809_195952693.jpg]]
![[PXL_20250809_200034706.jpg]]
![[PXL_20250809_200100385.jpg]]

![[PXL_20250809_200136903.jpg]]
![[PXL_20250809_200140910.jpg]]
![[PXL_20250809_200205057.jpg]]
![[PXL_20250809_200221594.jpg]]