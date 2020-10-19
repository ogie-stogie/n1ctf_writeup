# n1ctf_writeup
[Write Up Link](https://github.com/tbart27/n1ctf_writeup/blob/main/README.md)

### Team: 466 Crew
Taylor Bart<br>
Mohamed Alqubaisi<br>
Matt Evans<br>
James Taylor<br>
<br>
### rev: oflo


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
