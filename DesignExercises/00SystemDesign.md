# System Design general tips

## Table of Content

- [System Design general tips](#system-design-general-tips)
  - [Table of Content](#table-of-content)
  - [Questions to ask at the beginning of the project](#questions-to-ask-at-the-beginning-of-the-project)

## Questions to ask at the beginning of the project

Do not try to satisfy too many constraints at once. Start with the 5 main constraints and build your system from here. Try to understand what is actually true in what you are being told. If without the system, the users do a task in 2 hours, you bring value if the system does the task in 1h59. No need for it to be immediate.

- What would happen if this system went down for an hour?
- What's the most sensitive data it handles?
- How many users do you expect in 12 months?
- Which part of the system, if it broke or slowed down, would cause the most damage?
- Is there anything in this system with legal, compliance, or security implications?
