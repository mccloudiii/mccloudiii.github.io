---
layout: post
title:  "Automating Network Device Commands with Paramiko and Python"
date: 2021-08-15
categories: [Automation]
tags: [ssh, automation, paramiko, python]
description: Use concurrent futures module to create multiple SSH sessions using Paramiko
image:
 path: /assets_2/PARAMIKO.png
---


<!-- ![Paramiko](/assets_2/PARAMIKO.png){: .shadow } -->

<!-- <span style="font-family: 'Roboto', sans-serif;">This text will use the Roboto font.</span> -->


In this post, we'll walk through a Python script that automates the execution of commands on network devices using Paramiko for SSH connections and concurrent.futures for concurrent processing.


## Overview of the Script

The script performs the following tasks:

Establishes SSH connections to network devices.
Executes a list of commands on each device.
Saves the output of these commands to separate text files for each device.
Handles multiple devices concurrently using threading for better performance.
## Key Components of the Script

### Imports
```python
import paramiko
import concurrent.futures
import time
```
* **paramiko**: A Python library for SSH2 connections, used to securely connect to network devices.
* **concurrent.futures**: Enables concurrent execution of function calls using threads or processes.
* **time**: Provides time-related functions, like sleeping for a few seconds between operations.

## Devices

For this demo we will take advantage of always ON cisco devices available at **(Devnet website) <https://developer.cisco.com/>** 

|         IOS-XE        |
|------------------------------|
| sandbox-iosxe-recomm-1.cisco.com|

|         IOS-XR        |
|------------------------------|
| sandbox-iosxr-1.cisco.com |


```python

import paramiko
import concurrent.futures
import time

def execute_commands_on_device(hostname, username, password, commands, output_file):
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(hostname, username=username, password=password)

        channel = ssh.invoke_shell()
        time.sleep(1)  # Allow time for the shell to be ready

        with open(output_file, 'w') as f:
            for command in commands:
                channel.send(command + "\n")
                time.sleep(2)  # Wait for the command to be executed and output to be generated
                
                output = ""
                while channel.recv_ready():
                    output += channel.recv(1024).decode()

                # Write output to the file
                f.write(f"Command: {command}\n")
                f.write(f"Output:\n{output}\n")
                f.write("\n" + "-"*50 + "\n\n")

        channel.close()
        ssh.close()
    except Exception as e:
        print(f"Failed to execute commands on {hostname}: {str(e)}")

def main():
    # Enable Paramiko logging
    paramiko.util.log_to_file('paramiko.log')
    devices = [
        {"hostname": "sandbox-iosxe-recomm-1.cisco.com", 
         "username": "admin", 
         "password": "C1sco12345"},
        {"hostname": "sandbox-iosxr-1.cisco.com", 
         "username": "admin", 
         "password": "C1sco12345"}
    ]

    commands = [
        'term len 0',
        'show version',
        'show ip interface brief',
        'show running-config'
    ]

    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = []
        for device in devices:
            output_file = f"{device['hostname']}_output.txt"
            futures.append(executor.submit(execute_commands_on_device, device['hostname'], device['username'], device['password'], commands, output_file))

        for future in concurrent.futures.as_completed(futures):
            if future.exception() is not None:
                print(f"Execution generated an exception: {future.exception()}")

if __name__ == '__main__':
    main()
```


## References

- [Complete JNCIS-ENT (YouTube playlist)](https://www.youtube.com/playlist?list=PLsPPnwREYxwvQMlVtfpKU34uTwShws-3b)
- [JUNOS RIB-GROUPS (1/2)](https://momcanfixanything.com/junos-rib-groups-1-2/)




<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
