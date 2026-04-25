# Havoc C2 Lab: Teamserver to Windows Callback

## Overview

For this lab I wanted to get experience with how a C2 framework works in a real world environment. I used Havoc on my Kali lab machine and a Windows VM as the target The goal was to set up the teamserver, connect to it with the client, create a listener, generate a payload, and catch a callback from the Windows machine.

This was done completely inside my own lab environment. The purpose was to understand the  C2 flow, not to target any real systems.

## Lab Setup

| Role | System | Purpose |
|---|---|---|
| Operator / Teamserver  Kali Linux | Runs Havoc teamserver and client |
| Target  Windows VM  Executes the lab payload |
| Network  Lab network  Allows the Windows VM to reach Kali |

I used my Kali machine as the operator system. This is where I installed Havoc, started the teamserver, and connected with the client. The Windows VM was used as the target machine for testing the callback.

Before starting with Havoc I made sure the Windows VM could reach my Kali machine over the lab network. This was important because the payload needed to be able to call back to the listener running on Kali.

## Installing Havoc

I started by installing the dependencies needed to build and run Havoc on Kali. Havoc needs several packages because the teamserver, client, and payload-building parts all rely on different tools and libraries.

After the dependencies were installed, I cloned the Havoc repository and built the teamserver and client from source.
Once everything finished building, I had the main Havoc binary ready to use.

## Setting Up the Profile

After building Havoc, I created a lab profile based on the default profile. The profile is where Havoc stores settings for the teamserver and operator accounts.

In the profile, I configured a lab operator account I also checked the teamserver host and port settings so I knew where the client would connect.

The profile mattered because the teamserver uses it when starting up, and the client uses those same credentials to log in.

## Starting the Teamserver

Once the profile was ready, I started the Havoc teamserver from the Havoc directory.

The teamserver is basically the backend of the C2 setup. It handles listeners, callbacks, sessions, and communication between the operator client and any connected agents.

After starting it, I left the terminal open so the teamserver could keep running.

## Starting the Client

With the teamserver running, I opened a second terminal and started the Havoc client.

The client is the GUI that connects to the teamserver this is where I can create listeners, generate payloads, and interact with sessions after a callback came in.

Since the teamserver and client were both running on my Kali machine, I connected the client to the local teamserver using the operator account I created in the profile.

## Creating a Listener

After logging into the client, I created a listener.

For this lab, I made an HTTP listener and pointed it at my Kali lab IP. This allowed the Windows VM to reach the listener over the lab network.

The listener is an important part of the setup because the payload has to know where to connect back to. If the IP or port is wrong, the payload will run but no session will appear.

## Generating the Payload

Once the listener was active, I used Havoc to generate a Windows payload. I selected a Windows x64 EXE payload and configured it to use the listener I had just created.

At his point, the payload was ready, but I still needed to move it over to the Windows VM.

## Transferring the Payload

To transfer the payload, I started a simple Python HTTP server on Kali in the folder where the EXE was saved.

Then, from the Windows VM, I opened a browser and downloaded the payload from the Kali machine.

This was a simple way to move the file across the lab network without needing shared folders or extra setup.

## Getting the Callback

After the payload was downloaded onto the Windows VM, I executed it inside the VM.

A few seconds later, Havoc caught the callback and a new session appeared in the client. This showed that the payload successfully reached the listener and connected back to the teamserver.

This was the main goal of the lab: proving that the full chain worked from payload execution to session creation.

## Verifying the Session

Once the session was open, I ran a simple command to prove that I could interact with the Windows VM through Havoc.

The first command I ran was:
cmd
who ami

What I Learned

This lab helped me understand the basic flow of a C2 framework a lot better. I understood the general idea of what a C2 was, but actually setting it up made the pieces make more sense.

The main flow was:

Start teamserver → connect client → create listener → generate payload → transfer payload → execute on VM → catch callback → verify session

The biggest thing I learned is that the listener configuration matters a lot. The payload has to be built with the correct listener, and the Windows VM has to be able to reach the Kali IP and port. If the networking is wrong, the payload might run, but the callback will never show up.

I also got a better understanding of the difference between the teamserver and the client. The teamserver is the backend that manages the listeners, callbacks, sessions, and operator accounts. The client is the GUI that connects to the teamserver and gives the operator a way to control the lab.


From the defensive side, this lab also helped me think about what this activity might look like in logs. A defender might see an unknown EXE running, outbound traffic from the Windows VM to the Kali machine, HTTP callback behavior, or command execution through the session.

Overall, this lab gave me a better understanding of how C2 frameworks work at a basic level. For future labs, I want to add more defensive visibility by checking Windows event logs, Sysmon logs, or Wireshark traffic so I can see what the same activity looks like from the blue team side.
