---
layout: post
tag: Projects
author: JLN
---
## Automated CTF Solver in Python
Python program which takes in a configuration file and automatically completes the Bandit Over the Wire CT.

This script completes the first five levels of the Over The Wire's Bandit game which is located on a remote server accessible via a ssh connection. The script then goes through each level, executing commands to locate a file with the next password which is then supplied to another function reach the next round. The script contains eight functions in total, six for each of the levels and two establishing and closing the ssh connections. The script uses two external modules: paramiko and ConfigParser. Paramiko is used for logging in and communicating with the remote server, and ConfigParser is extracting the login details such as address and usernames from cfg files. There are six cfig files, each with specific login information for each level. These files do not contain any passwords with the exception for level zero. The script generates a txt file called "passwords" where all the passwords, that is six in total, are stored and prove that the program run successfully.

[To view the code and configuration files on Github click here.](https://github.com/jolana-veneny/Automated-CTF-Solver-in-Python)



``` python
from configparser import ConfigParser
import paramiko


# creates a login function which takes a cfig file and a password as variables and returns the ssh connection
def login(file, password):
    # extracts section name from file name
    file_section = file.replace(".cfg", "")
    # parses login details from a file
    config = ConfigParser()
    config.read(file)

    # assigns the details to variables
    address = config.get(file_section, "address")
    port = config.get(file_section, "port")
    username = config.get(file_section, "username")

    # establishes a connection to server using the login variables + removes a new line from the password variable
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(address, port, username, password.strip())
    return ssh


# creates a function which closes the ssh connection
def close_connection(stdin, stdout, stderr, ssh):
    stdin.close()
    stdout.close()
    stderr.close()
    ssh.close()


# creates a function which completes level 0
def level0():
    file = "bandit1.cfg"
    # loads up level0 password into a variable
    config = ConfigParser()
    config.read(file)
    password = config.get("bandit1", "password")
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password)

    # runs the bash commands
    stdin, stdout, stderr = ssh.exec_command("cat /home/bandit0/readme")
    password1 = str(stdout.readlines()[0])

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 1 is: ")
    f.write(password1)
    return password1


def level1(password1):
    file = "bandit2.cfg"
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password1)

    # runs the bash commands
    stdin, stdout, stderr = ssh.exec_command("cat /home/bandit1/-")
    password2 = str(stdout.readlines()[0])

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 2 is: ")
    f.write(password2)
    return password2


def level2(password2):
    file = "bandit3.cfg"
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password2)

    # gets a list of directory contents and reads in the content into a variable
    stdin, stdout, stderr = ssh.exec_command("ls")
    output = str(stdout.readlines()[0])

    # replaces spaces with escape characters in the file name
    filename = output.replace(" ", "\ ")

    # calls the command avoiding the spaces problem in Linux
    stdin, stdout, stderr = ssh.exec_command(f"cat {filename}")

    # gets the password, uploads it to a text file, and returns the value from the function
    password3 = str(stdout.readlines()[0])

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 3 is: ")
    f.write(password3)
    return password3


def level3(password3):
    file = "bandit4.cfg"
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password3)

    # changes directory to "inhere" and opens a hidden file ".hidden"

    stdin, stdout, stderr = ssh.exec_command("cd inhere; ls -a")
    output1 = str(stdout.readlines()[2])
    file_name = output1.strip()
    stdin, stdout, stderr = ssh.exec_command(f"cd inhere; cat \"{file_name}\"")
    password4 = str(stdout.readlines()[0])

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 4 is: ")
    f.write(password4)
    return password4


def level4(password4):
    file = "bandit5.cfg"
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password4)

    # creates a list of all the file in the inhere directory
    stdin, stdout, stderr = ssh.exec_command("cd inhere; ls")
    output1 = stdout.readlines()

    # loops over the list of files, trying to open one by one. When a file can be read in utf-8,
    # the file is read, the content (the password) is loaded into a variable
    for file in output1:
        try:
            file_clean = file.strip()
            stdin, stdout, stderr = ssh.exec_command(
                f"cd inhere; cat -- {file_clean}")
            password5 = str(stdout.readlines()[0])
            break
        except:
            continue

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 5 is: ")
    f.write(password5)

    return password5


def level5(password5):
    file = "bandit6.cfg"
    # uses the login function with a cfg file and a password as variables
    ssh = login(file, password5)

    # creates a list of all the file in the inhere directory
    stdin, stdout, stderr = ssh.exec_command("cd inhere; ls")
    output1 = stdout.readlines()

    # searches for and cleans a file name and path to a 1033 byte-sized file
    stdin, stdout, stderr = ssh.exec_command(
        "cd inhere; find -size 1033c")
    output2 = str(stdout.readlines()[0])

    # reads the file's contents, loads the password to a variable
    output2_cleaned = output2.strip()
    stdin, stdout, stderr = ssh.exec_command(
        f"cd inhere; cat {output2_cleaned}")
    password6 = str(stdout.readlines()[0])

    # closes the ssh connection
    close_connection(stdin, stdout, stderr, ssh)

    # gets the password, writes to a file, and returns it as a variable
    f = open("passwords.txt", "a")
    f.write("The password for level 6 is: ")
    f.write(password6)

    return password6


# calls level functions + takes their return values, assigns them to variables, 
# and passes them into the folllowing function
password1 = level0()
password2 = level1(password1)
password3 = level2(password2)
password4 = level3(password3)
password5 = level4(password4)
password6 = level5(password5)
```
