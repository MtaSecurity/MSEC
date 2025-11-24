# CSCV2025-FINAL-WRITEUP-CASECHARLIE
This is write up for challenge Case Charlie (Forensics) in CSCV2025 FINAL

**From MTA.SUPPORT (Military Technical Academy)**

Hi everyone, last Saturday I have solved this forensics challenge in the final round of Cybersecurity Contest Viet Nam 2025. Today, I will tell you how to solve this 400 points challenge.

<img width="608" height="706" alt="image" src="https://github.com/user-attachments/assets/963c8459-23e6-49bd-9751-8c03a5154149" />


**1, Preparation**

- First, I download the file gave by Author (This file is nearly 10GB so that I had downloaded it 2 days before the contest started). After download and unzip file, I have 3 files below
<img width="1632" height="361" alt="image" src="https://github.com/user-attachments/assets/54bf3da0-4483-4d40-9126-59b913aeef33" />

- Next, I read file Readme.pdf and read all the information of this situation
<img width="1154" height="654" alt="image" src="https://github.com/user-attachments/assets/17889a97-dce5-4030-9162-4bc9c83bc59a" />
- Collecting all the information and the password to unzip file evidences, I conclude that this is a cybercrime investigation involving unauthorized hacking, data theft and crypto. My task is to analyze the seized server, recover information related to the suspect, and answer investigative questions
  
=> I need to analysis file evidences.vmdk (get this file after unzip `evidences`) and connect to the instance and answer question

- Following, I fetch instance and connect to my virtual machine by netcat
  <img width="688" height="128" alt="image" src="https://github.com/user-attachments/assets/80402a2a-d47e-4f8b-a81a-02a1034ea404" />
  
=> This challenge request us to answer 6 questions relate to a cybercrime

**2, Analysis**

**Question 1:**  What was the last time the suspect logged into the server? (UTC / 24 hours format)  

- Because we all know that wtmp is a file that records every login/logout session. When a user logs in, Linux writes a new record to wtmp. When the user logs out, another record is written.

→ Every new login → wtmp is written → the file is “modified”. So that I find the wtmp in the disk and note the last time this file modified

`root/var/log/wtmp`

<img width="1913" height="916" alt="image" src="https://github.com/user-attachments/assets/f7c9b4ab-6311-4c3b-9a9b-f4f869f02a7d" />

- It can be easy to see that the answer is `2025-11-08 04:32:56`

**Question 2:** The suspect used a chat application to communicate. What is the name of this application? Format: chatapp. Example: whatsapp, telegram, Discord

- The author give us a example of chat application, so that I need to file a suspect app contain the communication between suspects
- After find information for nearly 30 minutes, I finally find a doubt conversation between s3v3n_wOndEr113 and the0nlymak3r1338
<img width="1895" height="915" alt="image" src="https://github.com/user-attachments/assets/9286141f-f5b1-4c22-b9d9-f5fe8c5344bb" />
- The first line is: **12:56 -!- Irssi: Starting query in liberachat with the0nlymak3r1338**

  -> the suspect app is `Irssi`

**Question 3** Identify the usernames (handles) of the suspect and their contact in the chat application. (Case Sensitive) Format: uS3rNaME13, c0nT4cTn4m3

- This question is easy because I have already found 2 peoples chatting in the logs
  -> the usernames is: `s3v3n_wOndEr113, the0nlymak3r1338`

**Question 4** What was the last time the suspect sent a message to their contact? (UTC / 24 hours format) Format: YYYY-MM-DD HH:MM  

- The author ask us when the time the conversation ended, read the log and scroll to the end, it is easy to see that the time is 04:36
- The chat is opened Fri Nov 07 12:56:16 2025 (On the top of the log)
  -> the answer is: `2025-11-08 04:36`

**Question 5** What command did the suspect use to overwrite the current user's .bash_history file to cover their tracks?

- I use a super trick in this question, that is STRINGS & GREP
- Use this command in CMD: `strings -n 5 E:\khang\cscv2025_final\forensics-Case_Charlie-f0f1f0e74a82270c9374bf0a9facd097\evidences\evidences.vmdk | findstr /i .bash_history`
- Because this file is so large, so that using strings is hard work, but **Practice makes perfect**, after nearly 5 minutes waiting, I see a suspect command
  `sudo shred -vfz /home/ubuntu/.bash_history`
 <img width="1699" height="406" alt="image" src="https://github.com/user-attachments/assets/093d13b8-adcb-4d40-af51-d46a641bc040" />

shred: a command that securely overwrites a file multiple times to make its contents extremely difficult or impossible to recover, even with forensic tools.

-v (verbose): Shows detailed output about what shred is doing (progress messages).

-f (force): Forces overwriting the file even if the file has certain protection, such as read-only permissions or immutable attributes.

-z (zero pass): After overwriting with random data, shred performs one final overwrite with zeros to hide the fact that the file was shredded.


-> so that the answer for this question is: `sudo shred -vfz /home/ubuntu/.bash_history`

**Question 6** The suspect accidentally left behind an email address. What is the email address? Format: name@domain. Example: this_Is_an_3xample_email1213@proton.me 

- This question request us to find an email address of the suspect, it isn't easy to find, I look for this by scroll all the file in the disk but it is nothing there. Then the author publish a hint to discover
<img width="538" height="222" alt="image" src="https://github.com/user-attachments/assets/91f45726-8e97-46e0-811a-83baba43ed49" />

- After reading a hint, I summary it with the log from Q2 to Q4, I have a important information
<img width="1919" height="913" alt="image" src="https://github.com/user-attachments/assets/b39734b6-47d0-4506-8ecd-f14c661f7a61" />
-> some image had been pulled in a file with old cred (I need to find it after) and use testing to pull it, this information remind me to Docker, so that I scroll to Docker file to dig for more information
- In .docker/config.json, I can find a registry url and a base64 strings
<img width="1918" height="911" alt="image" src="https://github.com/user-attachments/assets/0be3cc3b-86d5-4e2c-90f3-652a105e5cb1" />
- Decode this strings in Cyberchef and I have this asjdkhufh832:glpat-xwghQbDTsJbs1B2MubX_zG86MQp1OmlxOHVzCw.01.120yvtp2f
  -> user: asjdkhufh832
     PAT: glpat-xwghQbDTsJbs1B2MubX_zG86MQp1OmlxOHVzCw.01.120yvtp2f
     Credentials: registry.gitlab.com
- Having 3 keys to solve this question, I use this information to login in Docker and fortunately it's successful
`echo "glpat-xwghQbDTsJbs1B2MubX_zG86MQp1OmlxOHVzCw.01.120yvtp2f" | docker login registry.gitlab.com -u asjdkhufh832 --password-stdin`

<img width="1203" height="58" alt="image_2025-11-18_14-14-53" src="https://github.com/user-attachments/assets/8629c3eb-b213-4185-be38-62bb8c41dd89" />

- Then I use 'testing' to pull the image
- `docker image save -o solve.tar registry.gitlab.com/somegroup5803945/jkfhskdf2314:testing`

<img width="760" height="301" alt="image" src="https://github.com/user-attachments/assets/80b84d38-d903-47ca-9405-af2f3fd9edda" />

- After that, I got a file solve.tar so that I use a Command to extract layer of this file 

```mkdir extracted rootfs
tar -xf solve.tar -C extracted

cd extracted
for d in */; do
    [ -f "$d/layer.tar" ] && tar -xf "$d/layer.tar" -C ../rootfs
done
```

- Following this, I have a file name "roofts" so I use Grep to find an email 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ced4457f-82a0-4ebb-8802-886047ec35cb" />

-> finally I find the suspect email name: phuchungh96@gmail.com
-> Answer for the last question is: phuchungh96@gmail.com

**Get the flag**
After answer 6 questions, we have this

<img width="700" height="558" alt="image" src="https://github.com/user-attachments/assets/a7386d5d-50ae-4802-be0c-635f656068c7" />

So that the final flag is: **CSCV2025{docKER_St0r35_0uR_cR3deNtI41S-UneNCrYPTed-ln_lt5-cOnfig_file_lmao_xd_Ow0}**


