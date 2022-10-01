# road-writeup
A writeup of the TryHackMe CTF challenge 'Road'


# Initial Enumeration 

Starting with a nmap scan I found the services being run on the machine. nmap syntax: "nmap -T4 -sV -v $IP"

After the scan was completed I found two open ports (22,80)

<img width="524" alt="image" src="https://user-images.githubusercontent.com/66540055/193384380-cdf1c9a6-0aa1-4f69-91e1-0836b1c634a0.png">


# Web Enumeration

Opening the webpage I see that the webserver is hosting a web app. Initially going through all of the links I found a login page:

<img width="952" alt="image" src="https://user-images.githubusercontent.com/66540055/193384463-25dd6e14-98f9-418a-888f-949ce9508da0.png">


I see that we can register as a user, after successfully registering a new account I navigated to the profile page and seeing that the programming language is php I looked to create a reverse shell and upload it via the profile picture. As I get to the section to upload a profile picture I notice something very perculiar, only an admin may upload a profile picture and a critical oversight, the admin account has been listed!

<img width="960" alt="image" src="https://user-images.githubusercontent.com/66540055/193385290-36a4c5cf-a7aa-4ccc-ba29-fbd3bf47352d.png">


After doing some exploring through the site I see that there is a 'ResetUser' funciton, my first thought is great let's open up BurpSuite and see if we can tamper with some parameters! After firing BurpSuite up and enabling my proxy I enter in a password and see what data is being transmitted:

<img width="468" alt="image" src="https://user-images.githubusercontent.com/66540055/193385658-c8099cba-d0b2-427b-9486-1d897e91459b.png">


Looking at the data we see that a username and the new password is being sent over, I attempted to change the username to the admin account that I found earlier in the profile seciton.

<img width="538" alt="image" src="https://user-images.githubusercontent.com/66540055/193385708-5a04c579-28b4-4eb5-abee-bc63b24f49e7.png">

Changing the username originally supplied to the admin account worked! I have successfully changed the password of the admin account!


<img width="335" alt="image" src="https://user-images.githubusercontent.com/66540055/193385767-0846466f-4739-4675-af48-851224fb331f.png">


Now that the password has been changed I attempted to login to the admin account:

<img width="960" alt="image" src="https://user-images.githubusercontent.com/66540055/193385829-e7ad6c8f-cbe5-4651-b9df-ac4f13f2f45d.png">


Success! Next I navigated back to the profile section to attempt to upload a reverse shell via the profile picture function. First I will need to see where the uploaded picture will be hosted, after viewing the page source I found the location where the images will be hosted:

<img width="499" alt="image" src="https://user-images.githubusercontent.com/66540055/193385892-6d3c3346-8b26-435b-b3b6-1468ecb45174.png">


After constructing the reverse shell and uploading it I fired up a netcat listener and then navigated to the hosting location:

<img width="815" alt="image" src="https://user-images.githubusercontent.com/66540055/193385940-779aa9a0-bbe2-4986-b271-945af049e1e9.png">


Success, I'm in!

<img width="960" alt="image" src="https://user-images.githubusercontent.com/66540055/193386006-31da0739-818a-48b1-a574-f17059b2715c.png">


![download](https://user-images.githubusercontent.com/66540055/193386036-b82fc8f5-cbcb-418e-a130-de90d3ac4802.jpeg)


# Initial Access

Now that I am the dub-data user it's time to do some userenum.
By catting the /etc/passwd file I find a user as well as an interesting system account:

<img width="468" alt="image" src="https://user-images.githubusercontent.com/66540055/193386385-155ec676-0e96-49cb-984d-a1390191f0c2.png">


# User.txt Flag

By looking into the user's directory I found the first flag.

<img width="197" alt="image" src="https://user-images.githubusercontent.com/66540055/193386449-6af0a736-6a88-413a-acf1-4e3213f37e8f.png">


# Priviledge Escalation

After spawning a proper shell I moved into the /tmp directory and tested my write priviledges:

<img width="264" alt="image" src="https://user-images.githubusercontent.com/66540055/193386545-0a455ebd-1d72-43fd-a96e-5845fc48d3a5.png">


Now that I know I can write I upload linpeas to do all the heavy lifting for me to find a great privesc vector:

<img width="471" alt="image" src="https://user-images.githubusercontent.com/66540055/193386665-9bb20b44-d91e-4517-8c10-44c0310bd239.png">


Next step is making linpeas executable with the simple command "chmod +x linpeas.sh"
After executing the script I see that the machine is vulnerable to CVE-2021-4034!

<img width="220" alt="image" src="https://user-images.githubusercontent.com/66540055/193386775-f1055595-7f47-40b7-ab9e-349189fd7a43.png">

In a previous CTF I found a really useful toolkit created by Ly4k designed for this very vulnerability! Using this one-liner the toolkit is downloaded "curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit -o PwnKit" I uploaded the toolkit using the same method as uploading linpeas, made it executable and upon executing it I got root!

<img width="323" alt="image" src="https://user-images.githubusercontent.com/66540055/193387491-7b36dbee-48d7-4702-bb3b-536e304d5f69.png">


# Final Thoughts

This was a fun CTF, after fiddling around with the machine some more I decided to checkout MongoDB that I saw as I have never interacted with that database program before, this lead to finding out the password of the user that I had found earlier.

# MongoDB

After going through the help I used some of the commands to enumerate the database and find the following databases:

<img width="122" alt="image" src="https://user-images.githubusercontent.com/66540055/193392179-6dd8a96f-00c5-48a9-8a72-8254788087dd.png">


After using the " " command I was able to uncover a username and password:

<img width="646" alt="image" src="https://user-images.githubusercontent.com/66540055/193392258-ef97fe2d-a997-412f-b195-f68298479f4f.png">


Testing this with SSH it proved to provide another foothold into the server!

<img width="469" alt="image" src="https://user-images.githubusercontent.com/66540055/193392396-c7e1f2d7-6848-43c0-8264-ed424ebf8d3f.png">


# Final... uh Final Thoughts

This was a very cool room for me personally as I have never used MongoDB so researching the syntax to use was enlightening and humbling! 

Looking forward to doing more CTF's, if you have any reccomendations please let me know!


