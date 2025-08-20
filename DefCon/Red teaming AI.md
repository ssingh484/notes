Kodo AI

# LLMs are non deterministic
Blind to natural language exploits
Known attack patterns
Chatbots can leak sensitive keys, system prompts etc

# Anatomy

- Model
- prompt templates
- app logic
- user input
- tools/plugins

# Threat Landscape
- poisoned context
- jailbreaking
- role confusion
- data exfil
- prompt injection

# where attackers start
Language over code
Ambiguity
Small wins chained together 
Exploit model desire to be "helpful"

# playbook

- recon
- attack surface mapping
- adversarial prompt gen
- exec, logging and testing
- analysis and gardening

# Adversarial prompt gen

Combine human prompts with AI-assisted fuzzing

- curiosity
- authority
- role-play tricking

# Prompt hardening and guardrails

- Structured prompts with minimal ambiguity 
- Pre-filter and post-filter
- Models can forget roles mid convo with enough context
# RAG risks venn
![[PXL_20250809_001537505.jpg]]


# Shift AppSec left

![[PXL_20250809_001713407.jpg]]

# Measuring genAI sec
![[PXL_20250809_001743078.jpg]]

# Final takeaways

![[PXL_20250809_001836608.jpg]]
# Demo

Red team agents all built via kodo as the agentic IDE

![[PXL_20250809_002430115.jpg]]
![[PXL_20250809_002557892.jpg]]
![[PXL_20250809_003026007.jpg]]