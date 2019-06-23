# Security

## Authentication vs. Authorization

|Authentication|Authorization|
|--------------|-------------|
|Verifies the identity of a user| Verifies the privileges of a user|
|Answers: Who are you?|Answers: What do you have access to?|

## User/Client Authentication Mechanisms

### SCRAM-SHA-1

Default mechanisms for a user/client to connect to mongodb.

- Challenge/response mechanism
- username/password
- IETF Standard

### MONGODB-CR

- Challenge/response
- username/password
- Replace by SCRAM-SHA-1
- Deprecated as of v3.0

### X.509

- Cerification Based
- Introduced to v2.6
- TLS connection needed

### LDAP

- Lightweight Directory Access Protocol
- MongoDB Enterprise
- Used for directory info
- External mechism
  
### Kerberos

- Enterprise
- Developed @ MIT
- Designed for secure authentication
- External mechanism

## Internal Authentication Mechanisms
 
When internal authentication is enabled it will automatically enable client/user authentication.

### Keyfile (SCRAM-SHA-1)

- shared password
- copy exists on each member
- 6-1024 Base64 characters
- whitespace ignored

### X.509

- certification based
- recommended to issue different certs per member