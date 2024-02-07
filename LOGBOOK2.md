# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-2022-22954 is a critical vulnerability affecting VMware Workspace ONE Access and Identity Manager and is pertinent to Linux operating systems.
- It allows malicious actors to remotely trigger a server-side template injection.
- If successfully exploited, an unauthorized attacker with network access to the web interface can execute arbitrary shell commands as the VMware user.

## Catalogação

- **Reporting:** CVE-2022-22954 was reported by Steven Seeley (mr_me) of Qihoo 360 Vulnerability Research Institute.
- **Reporting Date:** The vulnerability was reported on 6th April 2022.
- **Severity:** VMware assessed it as critical with a maximum CVSSv3 base score of 9.8.
- **Bug-bounty:** not recorded.

## Exploit

- There's a Metasploit module that takes advantage of vulnerability CVE-2022-22954.
- It's an unauthenticated server-side template injection (SSTI) vulnerability in VMware Workspace ONE Access, to execute shell commands as the "horizon user".
- **Automation:** There is automation provided by the Metasploit module, but no other known public exploits or tools.

## Ataques

- **Harm Potential:** This vulnerability poses a high risk, enabling remote code execution and potential full system control.
- **CISA Warning:** Malicious actors, possibly advanced persistent threat (APT) groups, are actively exploiting CVE-2022-22954, according to the Cybersecurity and Infrastructure Security Agency (CISA).
- **Real-world Exploitation:** VMware has verified that CVE-2022-22954 has been exploited in actual incidents, emphasizing its critical nature.
- **CISA Report:** Trusted sources revealed multiple threat actors compromising a public server using VMware Workspace ONE Access. They employed various tactics, including data theft and deploying webshells and a SOCKS proxy.
