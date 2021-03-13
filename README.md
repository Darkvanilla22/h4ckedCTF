# h4ckedCTF
TryHackMe writeup for the h4cked CTF room

In this Blue-teamed focused room, it is our job to investigate a recently compromised machine that fell victim to a malicious threat actor. To begin our investigation, we turn to Wireshark, analyzing the captured activity that was caught when the attack occured.

To begin, we need to first find out what service the attacker has been exploiting. Here is a screenshot of the unfiltered PCAP file loaded into Wireshark:

![image](https://user-images.githubusercontent.com/53369798/111022335-3315ba00-83a0-11eb-8dc9-e766bebf9759.png)

Whenever I first open up an unfamiliar PCAP file, I want to know what protocols I'll be analyzing. Let's check out the Protocol Hierarchy:

![image](https://user-images.githubusercontent.com/53369798/111022413-969fe780-83a0-11eb-9a56-887ddd521383.png)

As you can see, this PCAP has three noticeable protocols being used: TCP, HTTP, and FTP (as well as FTP Data). After a quick skim through the PCAP content to see where major changes in the network traffic occurs, my attention became fixated onto the FTP activity, as I've noticed multiple login attempts being made:

![image](https://user-images.githubusercontent.com/53369798/111022567-6d338b80-83a1-11eb-8656-45e9a57f1dfa.png)

This is clearly the work of a dictionary attack. The user account being attacked, "jenny", appears to have set a weak password to their account, as we can see that the attack was successful in discovering her password:

![image](https://user-images.githubusercontent.com/53369798/111022663-209c8000-83a2-11eb-97f7-423f069a8cbd.png)

Since we found a crucial point in the event where the attacker has gained access into the system, we can follow their activity by tracking the data stream. Here is the whole stream that was captured:

![image](https://user-images.githubusercontent.com/53369798/111022762-d7006500-83a2-11eb-8491-a752e9fafdcf.png)

Based on what the evidence shows, the attacker has placed a reverse PHP shell into the webserver. We can also see the code of the malware by filtering the PCAP file to show only FTP Data. The attacker is using a payload from the given URL:

![image](https://user-images.githubusercontent.com/53369798/111022933-01065700-83a4-11eb-81f1-5ecef65bf50d.png)

Now we know how the attacker has gained intial access, we should continue observing the rest of the activity, such as what has happened after the attacker has connected the system through the shell. After looking through a few TCP streams, I've managed to discover the activity in which the attacker actually escalated their privileges:

![image](https://user-images.githubusercontent.com/53369798/111023066-d10b8380-83a4-11eb-80d4-84678f217cd1.png)

The data shows that the attacker has planted a backdoor (Reptile) into the root directory of the system. With the events now unfolded to us, we can replicate these events ourselves.

To break into the system ourselves, we should start with bruteforcing the user "jenny", as the attacker has most likely changed the password to their account after the previous attack. To do this, we can use Hydra:

![image](https://user-images.githubusercontent.com/53369798/111023222-cdc4c780-83a5-11eb-9605-328611c9b592.png)

Now that we know the login credentials needed to access the FTP server, we shall login to the service and place our own PHP reverse shell into the web server:

![image](https://user-images.githubusercontent.com/53369798/111023280-11b7cc80-83a6-11eb-847f-8c181940b1d6.png)

Alright, time to get fire up a Netcat listener and connect to the web server by entering the path to the reverse shell into the URL field:

![image](https://user-images.githubusercontent.com/53369798/111023327-5fccd000-83a6-11eb-80a9-d0a08f776d75.png)

And now that we've connected to the system, we can escalate our privileges to root and discover the flag waiting for us in the root directory:

![image](https://user-images.githubusercontent.com/53369798/111023375-b89c6880-83a6-11eb-9ebd-92e238ab1b2b.png)

And that's the end to this room!
