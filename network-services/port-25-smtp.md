---
description: Chat GPT3 description generated
---

# Port 25 - SMTP

SMTP (Simple Mail Transfer Protocol) is a network protocol used to send and receive email messages between email servers. It is a text-based protocol that defines the rules and format for email messages and how they are exchanged between email servers.

SMTP operates over TCP/IP and is primarily used by mail servers to send outgoing messages to other mail servers for delivery to their intended recipients. SMTP uses a client-server architecture, where an email client or sending server connects to the destination server and sends the email message using the SMTP protocol.

SMTP works in a step-by-step process, where the sending server first establishes a connection with the receiving server using the Transmission Control Protocol (TCP). Once the connection is established, the sending server sends a series of commands and responses to the receiving server, which includes identifying the sender and recipient of the email, checking for message errors, and transmitting the actual message content. The receiving server then stores the message and sends a confirmation message back to the sending server, indicating that the message was received and accepted.



## User enumeration

### Automated tools

```bash
### smtp-user-enum ### 
smtp-user-enum -M VRFY -U users.txt -t 10.0.0.1
smtp-user-enum -M EXPN -u admin1 -t 10.0.0.1
smtp-user-enum -M RCPT -U users.txt -T mail-server-ips.txt
smtp-user-enum -M EXPN -D example.com -U users.txt -t 10.0.0.1
```

### VRFY

The `VRFY` command in SMTP (Simple Mail Transfer Protocol) is used to verify the existence of a specific email address on a mail server.

```bash
$ telnet 10.0.0.1 25
Trying 10.0.0.1...
Connected to 10.0.0.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO
501 HELO requires domain address
HELO x
250 myhost Hello [10.0.0.99], pleased to meet you
VRFY root
250 Super-User <root@myhost>
VRFY blah
550 blah... User unknownde
```

### EXPN

```
$ telnet 10.0.10.1 25
Trying 10.0.10.1...
Connected to 10.0.10.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO
501 HELO requires domain address
HELO x
EXPN test
550 5.1.1 test... User unknown
EXPN root
250 2.1.5 <ed.williams@myhost>
EXPN sshd
250 2.1.5 sshd privsep <sshd@mail2>
```

### RCPT TO

```
$ telnet 10.0.10.1 25
Trying 10.0.10.1...
Connected to 10.0.10.1.
Escape character is '^]'.
220 myhost ESMTP Sendmail 8.9.3
HELO x
250 myhost Hello [10.0.0.99], pleased to meet you
MAIL FROM:test@test.org
250 2.1.0 test@test.org... Sender ok
RCPT TO:test
550 5.1.1 test... User unknown
RCPT TO:admin
550 5.1.1 admin... User unknown
RCPT TO:ed
250 2.1.5 ed... Recipient ok
```



