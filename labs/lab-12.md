# Lab 12: "Rio de Janeiro": Do we have another option?

## Description
This scenario server is dedicated to Jenkins, a Java application managed by systemd. Jenkins is failing to start. Troubleshoot and find the problem, then apply the solution so Jenkins runs properly.

🔗 **Lab Link:** [SadServers - "Rio de Janeiro": Do we have another option?](https://sadservers.com/scenario/rio)

<br>

## 🪜 Steps

### Step 1: Check Jenkins service status

```bash
sudo systemctl status jenkins
```

<img width="1001" height="93" alt="image" src="https://github.com/user-attachments/assets/afc987ce-b25f-47a5-969c-da38f598b9f0" />

### Step 2: Trying to start jenkins

```bash
sudo systemctl start jenkins
```

<img width="1021" height="111" alt="image" src="https://github.com/user-attachments/assets/98c967e0-8844-4fed-8795-f84d4aac23a3" />


### Step 3: Searching in the logs

```bash
sudo journalctl -u jenkins.service -n 50 --no-pager
```

<img width="1175" height="707" alt="Screenshot 2026-08-30 221712" src="https://github.com/user-attachments/assets/34d89ced-97cb-49c9-b2c5-e7b9ee292e3b" />

> **The most important line is: java.lang.UnsupportedClassVersionError:
executable/Main has been compiled by a more recent version of the Java Runtime
(class file version 55.0),
this version of the Java Runtime only recognizes class file versions up to 52.0

>What does it mean?
There is a mismatch between the version of Java that Jenkins needs and the version of Java currently installed.

### Step 4: Configuring Java version

```bash
java -version
```

<img width="761" height="96" alt="Screenshot 2026-08-30 221733" src="https://github.com/user-attachments/assets/42e3f470-f7d4-40f5-aa55-ddcda5843000" />

>You have: openjdk version "1.8.0_462" Which means your default Java is: Java 8 But the logs said: class file version 55.0 And 55.0 means Java 11.

- Before we install anything, we check if Java 11 is already on the machine.

```bash
ls /usr/lib/jvm/
```

<img width="938" height="67" alt="Screenshot 2026-08-30 221914" src="https://github.com/user-attachments/assets/432eaf7f-5ca0-42d7-ba8a-d5ecf83bc720" />

- updating the Java version:

```bash
sudo update-alternatives --config java
```

<img width="918" height="227" alt="Screenshot 2026-08-30 222239" src="https://github.com/user-attachments/assets/63477314-8d58-49c7-a897-fba0df5b1c55" />

- Making sure the change was done

```bash
java -version
```

<img width="888" height="112" alt="Screenshot 2026-08-30 222258" src="https://github.com/user-attachments/assets/f53d1587-27dd-454c-a8c3-aa9912973c03" />

```bash
sudo systemctl restart jenkins
```

```bash
sudo systemctl status jenkins
```

<img width="1182" height="555" alt="Screenshot 2026-08-30 222421" src="https://github.com/user-attachments/assets/e99ff032-b407-4bec-83e1-8f666f5a0580" />

- Testing

```bash
curl -s localhost:8888/login | grep Jenkins | head -n1
```

<img width="1177" height="178" alt="Screenshot 2026-08-30 222543" src="https://github.com/user-attachments/assets/afaf0b6a-098e-4bc8-81fc-368917a405e7" />
