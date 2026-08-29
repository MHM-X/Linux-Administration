# Lab 07: "Apia": Needle in a Haystack

## Description
 In a directory /home/admin/data, there are multiple files, all of them with same content. One of these files has been modified, 
 a word was added. You need to identify which word it is and put it in the solution file (both newline terminated or not are accepted).

🔗 **Lab Link:** [SadServers - "Apia": Needle in a Haystack](https://sadservers.com/scenario/apia)

<br>

## 🪜 Steps

### Step 1: Thinking
We can write a script to calculate the MD5 hash of each file. The file with a different word will have a different hash, so we can identify it.

Then, we can use a tool to compare the different file with one of the original files.

In the previous stage, I wrote a Bash script that can be useful for this task: [Text-based Diff tool](https://github.com/MHM-X/Bash-Scripting/blob/main/Text-Processing-Projects/05-text-based-diff-tool.md)

However, using the tool this way is overkill. We can simply use:

```bash
diff normal_file different_file
```

### Step 2: Finding the unique file

```bash
md5sum /home/admin/data/* | sort | uniq -w32 -u
```

<img width="876" height="75" alt="Screenshot 2026-08-29 124132" src="https://github.com/user-attachments/assets/c3ece31a-b8d1-400b-b0d9-78c77e0737d5" />

### Step 3: We can choose one of the previous identical files using the following command:

```bash
ls /home/admin/data
```

<img width="1102" height="367" alt="Screenshot 2026-08-29 124213" src="https://github.com/user-attachments/assets/3a8fc091-c702-4ecd-be2d-fc2d852708cb" />

### Step 4: Finding the different word

```bash
diff /home/admin/data/file76.txt  /home/admin/data/file0.txt
```

<img width="1177" height="372" alt="Screenshot 2026-08-29 124356" src="https://github.com/user-attachments/assets/9e87419d-f2f0-4533-946c-a72fa3549e7d" />

`3c3` means that line 3 in the first file differs from line 3 in the second file.

`c` stands for **change**, meaning that the line was changed.


### Step 5: Then, after finding the different word:

```bash
echo "eureka" > /home/admin/solution
```



