# Secure Chat Application using OpenSSL and MITM attacks
A secure peer-to-peer chat application using openssl in C/C++ and demonstrate how Alice and Bob could chat with each other using it. It also includes evil Trudy who is trying to intercept the chat messages between Alice and Bob by launching various MITM attacks.
## Table of Contents
+ [Overview](https://github.com/Harsha1969/secure_chat_app/edit/main/README.md#overview)
+ [Instructions to run Secure Chat App](https://github.com/Harsha1969/secure_chat_app/edit/main/README.md#instructions-to-run-secure-chat-app)
## Overview
This includes 4 tasks:
### Task-1: Generate keys and certificates
In this task, using OpenSSL keys are generated for root CA, Intermediate CA, Alice and Bob. Then root CA generates a self-signed certificate and intermediate CA generates certificate which is signed by root CA. Alice and Bob generate Certificate Signing Request (CSR) and send them to intermediate CA. Then the intermediate CA sign these CSR and issue certificates to ALice and Bob.
### Task-2: Secure Chat App
In this task, secure chat application logic is implemented. Initially server will be waiting for connection requests. The client who send chat_hello messages will be connected with the server. In return client recieves chat_hello_reply as 
an acknowledgement for chat_hello. Then client sends chat_start_SSL to establish a secure SSL connection with the server. Then server sends chat_start_SSL_ACK as an acknowledgement. When the client recieves the acknowledgement then the process of SSL handshake starts which includes client hello,server hello , server and client authentication etc. Timers are maintained to send these control messages, if acknowledgement is not recieved in the stipulated time then the message will be retransmitted. After this process a secure SSL connection will be established between client and server which can be used to chat.
### Task-3: START_SSL downgrade attack for eavesdropping
In this Trudy will be poisoning the dns of Alice and Bob and so Alice messages will be recieved by Trudy and it forwards them to Bob. Also Trudy performs a downgrade attack so that Alice and Bob communicates over a unsecure channel. For doing this, Trudy doesn't forward chat_start_SSL to Bob but in return it replies back to Alice with chat_START_SSL_NOT_SUPPORTED. Then Alice downgrades the version and sends chat_without_SSL then this will be forwarded to Bob by Trudy. Bob sends chat_without_SSL_OK as acknowledgement. In this way Alice and Bob starts to chat without SSL on a unsecure channel which is eavesdropped by Trudy.
### Task-4: Active MITM attack for tampering chat messages and dropping DTLS handshake messages
In this task, Trudy hacks into intermediate CA server and issues fake certificates to Alice and Bob. Now Trudy poisons the dns of Alice and Bob. In this way Trudy doesn't need to perform a downgrade attack now, as Trudy recieves messages from Alice then they can decrypted and can be modified if needed then re-encrypted and can be sent to Bob. In this case,there will be two SSL connections, one between Alice and Trudy and other between Trudy and Bob. In this way Trudy can perform active MITM attack for tampering messages and drop messages.

## Instructions to run Secure Chat App
### Task 1: Generating keys and Certificates
step 1.1 create a Ecc 512 bit private key for RootCA
```
openssl ecparam -name prime256v1 -genkey -noout -out root.key
```

step 1.2 create a Certifcate signing request for RootCA certificate using :
```
openssl req -new -key rootCA.key -out root.csr
```

step 1.3 Self sign the root CSR using a pre created extension file with all required extensions for Root CA:
```
openssl req -new -key root.key -out root.csr -extensions v3_req -extfile v3_extensions.txt
```

step 1.4 create a 4096-bit RSA public key for Intermediate CA
```
openssl genpkey -algorithm RSA -out int.pem -pkeyopt rsa_keygen_bits:4096
```

step 1.5 create a Certifcate signing request for intermediate CA certificate using :
```
openssl req -new -key int.pem -out int.csr
```

step 1.6  get the int.csr signed by the root CA
```
openssl x509 -req -in int.csr -CA root.crt -CAkey root.key -CAcreateserial -out int.crt -extfile exten.txt -extensions v3_req
```

step 1.7 create a 1024-bit RSA public key for Intermediate CA
```
openssl genpkey -algorithm RSA -out alice.pem -pkeyopt rsa_keygen_bits:1024
```

step 1.8 create a Certifcate signing request for alice certificate using :
```
openssl req -new -key alice.pem -out alice.csr
```

step 1.9  get the alice.csr signed by the int CA
```
openssl x509 -req -in alice.csr -CA int.crt -CAkey int.key -CAcreateserial -out alice.crt
```

repeat the similar process for bob as well.

### Task 2: Secure Chat App
Make a copy of the repository in your computer.

step 2.1 open a terminal from Alice folder , and open the other terminal from bob folder
          and compile the code using :
```
g++ Secure_chat_app.cpp -lssl -lcrypto
```

step 2.2 then start the server using :
```
./a.out -s alice1
```
          
step 2.3 then start the client using :
```
./a.out -s bob1
```
step 2.4 now after the control messages get exchanged , you will be asked to send messages, you can continue the chat.

step 2.5 once you want to exit from the chat session , you can send "chat_close" message from client or from server.

### Task 3: SSL downgrade attack for eavesdropping

step 3.1 use the script given to poison the DNS of Alice and Bob to eavesdrop their communication.
         use the command for dns poisoning:
```
bash poison-dns-alice1-bob1.sh
```

step 3.2 open a terminal from Trudy folder and compile the code using :
```
g++ Secure_chat_interceptor_app.cpp -lssl -lcrypto
```

step 3.3 then start the server using :
```
./a.out -s alice1
```

step 3.4 then start Trudy using :
```
./a.out -d alice1 bob1
```
step 3.5 then start the client using :
```
./a.out -s bob1
```

you can clearly see the messages eavesdroped by trudy in the Trudy's terminal.

step 3.6 use the script given to unpoison the DNS of Alice and Bob after completing the attack.
         use the command for dns unpoisoning:
```
bash unpoison-dns-alice1-bob1.sh
```

### Task 4: Active MITM attack for tampering chat messages

step 4.1 use the script given to poison the DNS of Alice and Bob to eavesdrop their communication.
         use the command for dns poisoning:
```
bash poison-dns-alice1-bob1.sh
```

step 4.2 open a terminal from Trudy folder and compile the other code using :
```
g++ Secure_chat_active_interceptor_app.cpp -lssl -lcrypto
```

step 4.3 then start the server using :
```
./a.out -s alice1
```

step 4.4 then start Trudy using :
```
./a.out -d alice1 bob1
```

step 4.5 then start the client using :
```
./a.out -s bob1
```

you can clearly see the messages eavesdroped by trudy in the Trudy's terminal.

step 4.6 for every message sent from alice to bob and from bob to alice , you will be given a option to modify the message,
        you can replace the message as you wish.

step 4.7 use the script given to unpoison the DNS of Alice and Bob after completing the attack.
         use the command for dns unpoisoning:
```
bash unpoison-dns-alice1-bob1.sh
```      



