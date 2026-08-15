# SSH Hardening — Keys, fail2ban and Tunnelling

Student Name: Mohammed Omer Ahmed

Student ID: 12303667

Project: SSH-Hardening-12303667

# 1. Overview
This activity focused on securing an SSH server using Ed25519 key-based authentication, SSH configuration hardening, fail2ban protection and SSH tunnelling. The aim was to improve remote-access security and securely access an internal web service.

# 2. Network Setup
The GNS3 network contained four Linux hosts connected through a switch.
| Node | IP Address | Purpose |
|------|------------|---------|
| Admin | 10.10.1.10/24 | Administration workstation |
| Bastion | 10.10.1.20/24 | SSH jump/tunnel host |
| Server | 10.10.1.30/24 | Hardened SSH server |
| Internal | 10.10.1.40/24 | Internal web server |

Connectivity was verified using ping.

# Evidence
<img width="1919" height="1079" alt="Screenshot 2026-08-13 183632" src="https://github.com/user-attachments/assets/cfa4ffe2-3882-403c-9b4c-fcac444116a1" />

<img width="1919" height="1079" alt="SSH-Hardening-12303667" src="https://github.com/user-attachments/assets/ef1875e9-5f31-4ec8-ad48-8aaa45032e27" />


# 3. SSH Key Authentication

- An Ed25519 key pair was generated on Admin and copied to the Server.

ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""

ssh-copy-id -i ~/.ssh/id_ed25519.pub root@10.10.1.30

- The same Admin public key was added to the student account:

ssh-copy-id -i ~/.ssh/id_ed25519.pub student@10.10.1.30

ssh student@10.10.1.30

- Result:

 The student account successfully logged in without a password.

# 4. SSH Hardening

- The /etc/ssh/sshd_config file was configured with:

PermitRootLogin no

PasswordAuthentication no

AllowUsers student

MaxAuthTries 3

- The SSH configuration was reloaded:

kill -HUP $(pgrep sshd)

- Testing confirmed that:

Student key-based login worked.

Password authentication was rejected.

Root SSH login was rejected.

# Evidence
<img width="1919" height="1079" alt="SSH-Hardening-12303667 sshd" src="https://github.com/user-attachments/assets/85cd8749-db93-4aff-94ea-34def3a9358b" />


# 5. fail2ban Protection
fail2ban was configured to block repeated failed SSH attempts.

[sshd]

enabled = true

maxretry = 3

findtime = 600

bantime = 600

- The service was started and checked using:

fail2ban-client start

fail2ban-client status sshd

Repeated failed SSH attempts were generated from Bastion.

- Result:

 Bastion IP 10.10.1.20 appeared in the banned IP list, confirming that fail2ban was working.

# Evidence 
<img width="1919" height="1079" alt="Screenshot 2026-08-13 183051" src="https://github.com/user-attachments/assets/f6299658-3b30-4efd-988c-37ba8ef81010" />


# 6. SSH Tunnelling

- TCP forwarding was enabled on Bastion:

AllowTcpForwarding yes

A simple HTTP service was started on Internal at port 8080.

- The SSH tunnel was then created from Admin:

ssh -f -N -L 9090:10.10.1.40:8080 root@10.10.1.20

- The tunnel was tested with:

curl http://localhost:9090/

- Result:

 The Internal Server page was successfully returned through the SSH tunnel.

# Two packet captures were saved

<img width="1919" height="1079" alt="Screenshot 2026-08-13 184313" src="https://github.com/user-attachments/assets/5fc8e7e0-1d9a-42ab-bd57-eeb127d44155" />

<img width="1919" height="1072" alt="Screenshot 2026-08-13 184333" src="https://github.com/user-attachments/assets/016ee4e4-f8fb-4f46-81f8-dcabde6b1aea" />


# Analysis Questions
- Why is Ed25519 recommended over RSA?

Ed25519 provides strong modern security with smaller keys and efficient performance. RSA is still secure with a suitable key size, but its keys are much larger. Ed25519 is therefore a good choice for new SSH deployments, while RSA may still be useful for compatibility with older systems.

- How does fail2ban protect SSH?

fail2ban monitors failed login attempts and temporarily blocks an IP address after too many failures. One limitation is that attackers can use multiple IP addresses, so fail2ban should not be the only security control.

-  How do key authentication and fail2ban work together?

Key-only authentication prevents attackers from gaining access by guessing passwords because a valid private key is required. fail2ban blocks IP addresses that repeatedly attempt unsuccessful logins. Together, they provide stronger protection against unauthorised SSH access.

-  Local (-L) vs Remote (-R) Port Forwarding:

Local port forwarding (-L) allows the local computer to securely access a service through an SSH server. In this activity, Admin used it to access the Internal web server through Bastion.

Remote port forwarding (-R) works in the opposite direction by creating a port on the remote side that forwards traffic back toward a service accessible from the local machine.

-  Packet Capture Comparison:

The Admin–switch capture shows encrypted SSH traffic between Admin and Bastion, so the HTTP request cannot be read.

The Internal–switch capture shows the HTTP request and response in readable form because the traffic has already exited the SSH tunnel at Bastion. This demonstrates that the SSH tunnel protects traffic between Admin and Bastion, but not the HTTP traffic after it leaves the tunnel.

Conclusion

This activity demonstrated how SSH security can be improved using key-based authentication, restricted user access, disabled password and root login, fail2ban and SSH tunnelling. Together, these controls provide a stronger and more secure approach to remote access.
