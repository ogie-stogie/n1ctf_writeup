# n1ctf_writeup
[Write Up Link](https://github.com/tbart27/n1ctf_writeup/blob/main/README.md)

### Team: 466 Crew
Taylor Bart<br>
Mohamed Alqubaisi<br>
Matt Evans<br>
James Taylor<br>
<br>
### rev: oflo
This is where Ryan and I spent most of our time in this CTF, but we were looking for a buffer overflow (since the name was oflo) the whole time. Once Mohamed looked at the challenge, he mentioned that it might be a "nanomite protected binary with a twist". I'll leave the description for his write-up and instead give an overview of our attempts.<br>
<br>
We were able to get three types of responses from the program:<br>
<br>
1 & 2:<br>
![](https://github.com/tbart27/n1ctf_writeup/blob/main/pwn1.jpg)<br>
![](https://github.com/tbart27/n1ctf_writeup/blob/main/pwn2.jpg)<br>
3:<br>
![](https://github.com/tbart27/n1ctf_writeup/blob/main/pwn3.jpg)<br>
<br>
Matt and I both thought that what the program was reading next had to be an address cause some responses would illicit "Stack-smashing detected!" response. W tried to give it various addresses from the code and we would receive either: seg fault, illegal instruction, or the program would exit normally. What was also indicative of a buffer overflow was that the program read additional input after the address (up to the letter q in the alphabet). With a combination of Ghidra and Binary Ninja we were slowly trying to map the execution flow of the program but there were around 9 sub_functions shown in Binary ninja, and through Ghidra it seemed that the code was calling an address so we reasoned that there must be a function saved within the memory.<br>
<br>
Matt was able to install some disassembler software for the function saved in memory where it would take the bytes and convert to assembly code that we could continue to reverse engineer. The self-named "Disassembler" was found through stack overflow and can be referenced [here](https://stackoverflow.com/questions/7200424/is-there-a-program-to-change-hex-code-to-assembly-code-in-x86). The code seemed reasonable and can be viewed [here](https://github.com/tbart27/n1ctf_writeup/blob/main/assembly), but when we tried to enter addresses in the functions range, we saw no output that lead us anywhere new.

### web: SignIn
This challenge involved sending an injection attack via php to a server. When you follow the challenge link, you are greeted with the php code. In the early part of the ctf, Matt proposed that the injection would happen in the url through an attack such as:<br>
<br>
`
http://101.32.205.189/?input=INJECTION_CODE_HERE
`

<br>
The php process the input variable through a get method and that is a red flag for injection attempts. After not seeing any new leads and having been in the middle of oflo, we returned to oflo (where we kept trying to find a non-existent buffer overflow). Towards the end of the CTF, James joined me and helped give me some new perspectives on how to approach this problem, as well as me attempting to work backwards from the flag to the injection code. Here are some of the notes and thoughts as we revisted this problem:<br>
<br>
<div>

http://101.32.205.189/?input=INJECTION_CODE_HERE

$input = INJECTION_CODE_HERE

php injection in unserialize($input)

we believe class ip will allow us to set the variables for later use in the flag class?
possibly use ip class to get file? and then flag uses readfile on that returned file?
Could possibly be two different methods for accessing the flag that are unrelated?

waf is a filter to remove remote command execution, however the function is empty

set HTTP_X_FORWARDED_FOR in $_SERVER #This is where super global environment variable exist?

flag.__construct("n1ctf")
$this->ip = $ip

flag.getflag()
this->check == key****************
calls readfile() #thats a syscall
returns flag

flag.__destruct()
calls flag
echos the output of flag.getflag()

the output should be returned as an html page
what is the purpose of the __wakeup? James believes it could be for testing purposes

testing plan:

use __wakeup() to see if __construct("n1ctf") sets $this->ip properly
see if "welcome to nictf2020" is printed when flag is destructed (what we really want is ip = key**************** for final injection attack)

injection attempts
<div>

`http://101.32.205.189/?input=new%20flag%20=%20flag.__construct(%22n1ctf%22);%20flag.__wakeup();%20flag.__destruct();`
<br>
`http://101.32.205.189/?input=$flag=new%20flag%20=%20flag(%22n1ctf%22);%20flag.__wakeup();%20flag.__destruct();`
<br>

<div>
By the time we were attempting the injection attempts, the CTF had finished. As it concluded, we were thinking that the code had two vulnerabilities, either through a sql injection where we could possibly see if we could append a read file command to the sql query, or what we focused on more heavily was creating the flag object with a correct string as a parameter and seeing if when the destruct method was invoked it would echo the output of flag.getflag() which would be the flag itself.
<div>

### Conclusion
This was a challenging CTF where the two challenges I spent all my time on were suppose to be most solved challenges of the competition. In hind sight, the web challenge SignIn was more fun and I wish I had swapped the amount I had spent on the two challenges. Crafting the actual injection code is still troublesome even after this many CTFs, and well reversing by nature can be tedious and time-consuming no matter the challenge. There are a lot of questions I have now that the competition is over and hopefully I will be able to find some write-ups that provide general attacks that I can use for later competitions. Sadly, there were no flags submitted once time was over.
