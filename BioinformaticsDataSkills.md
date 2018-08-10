# Chapter 2. SETTING UP AND MANAGING A BIOINFORMATICS PROJECT

## Project Directories and Directory Structures 

All Files and directories used in your project should live in a single project directory with a clear name.

```bash 
mkdir <project_name>
cd <project_name>
mkdir data/seqs scripts analysis #data contains all raw and intermediate data
tree
```

It's better practice not to use spaces in file or directory names in the first place.

It's important to always use ***relative*** paths rather than ***absolute*** paths. As long as your internal project directory structure remains the same, relative paths will always work. Using absolute paths leaves your work less portable and reduces reproducubility.

## Project Documentation

1.  Document your Methods and Workflows.
2. Document origin of all data in project directory.
3. Document when you downloaded the data.
4. Record data version information.
5. Describe how you downloaded the data.
6. Document software's version.

All this information is best stored in plain-text README.files, to be stored in each of of project's main directories.

## Shell Expansion Tips

Shell expansion is when your shell (e.g., Bash) expands text for you so you don't have to type it out.

A type of shell expansion is brace expansion, which creates strings by expanding out the comma-separated values inside the braces:

```bash
mkdir -p <project_name>/{data/seqs,scripts,analysis} #-p flag creates any necessary subdirectories
```

## Wildcards

OS X and Linux systems have a limit to the number of arguments that can be supplied to a command. When this happens you will see an "***Argument list too long***" error message. 

In general, it's best to be as restrictive as possible with wildcards to avoid accidental matches

***** 		Matches ***zero or more*** characters.

**?** 		Matches **one** character.

**[A-Z]** 	Matches **any** character between he supplied alphanumerical range.

While wildcards are powerful, they're useless if files are inconsistently named.

## Leading Zeros and Sorting

Another trick is to use leading zeros (e.g., file-0021.txt instead of file-21.txt) when naming files. This is useful because lexicographically sorting files leads to the correct ordering. 

## Use Markdown for Project Notebooks

It is a plain-text format easy to read that can be also rendered to HTML or PDF.



# CHAPTER 3. REMEDIAL UNIX SHELL

Bash is the default shell on operating systems like OS X and Ubuntu Linux. Tou can run `echo $SHELL`  and `echo $0` to verify you're using bash as your shell. 

Is possible to change your shell with `chsh` ( Z shell-zsh has more advanced features handy in bioinformatics).

## Useful Unix shortcuts

These are *essential* if you spend a lot of time in the shell. See the [preface readme](https://github.com/vsbuffalo/bds-files/blob/master/chapter-00-preface/README.md) for more details on how I set up my terminal (some of these changes are necessary for the following to work):

- `control-a`: move cursor to beginning of terminal line.
- `control-e`: move cursor to end of terminal line.
- `control-k`: delete (or *k*ill) all text from cursor to end of line.
- `option-delete`: delete an entire word
- `option-b`: move cursor backwards an entire word.
- `option-f`: move cursor forwards an entire word.
- `control-c`: cancel input text or when a command is running, stop it.
- up arrow: access last entered command.
- `control-r`: start searching shell history. Start typing to search -- `enter` will enter the current command. `control-c` will cancel.

## Redirecting Standard Out to a File

- The operator  **>** redirects standard output to a file and overwrites any existing contents of the file.

- The operator  **>>** appends to the file.

If there isn't an existing file, both operators will create it before redirecting output to it. 

 ## Redirecting Standard Error

Like standard output, standard error is by default directed to the terminal. 

To redirect each streams to **separate** files, we combine the **>** operator with the **2>** operator for redirecting the standard error stream.

## File Descriptors

All open files on Unix systems are assigned a unique integer known as a ***file descriptor***. Unix's three ***standard streams***: **standard input**, **standard output** and **standard error** are given the file descriptors 0, 1 and 2, respectively.

Unix-like systems have a special "fake" disk (pseudodevice) to redirect unwanted output to: `/dev/null` Output written to this pseudodevice disappears.

## Using Standard Input Redirection

With the **<** redirection operator we can read standard input **from a file**.

We can use then:

```bash
program < inputfile > outputfile #or
inputfile | program > outputfile
```



## The Unix Pipe

Unix pipes redirect a program's standard ouput stream to another program's standard input.

Pipes are very fast; writting or reading from a disk is often a bottleneck in data processing. Pipes allow to build larger, more complex tools from smaller modular parts. 

## Creating Simple Programs with `grep` and pipes

The `grep` Unix tool searches files or standard input for strings that match patterns. These patterns can be simple strings or regular expressions.

```bash
grep -v "^>" tbl.fasta \ #Remove all lines beginning with >
grep --color -i "[^ATCG]" #print lines containing non-nucleotide characters
```

When used in brackets, a caret symbol **matches anything that's not** one of the characters in the brackets.

We can ignore case with `-i` .

It is good practice to quote regular expressions.



## Combining Pipes and Redirection

The `2>&1` operator redirects standard error to the standard output stream.



## The `tee` Program

The UNix program `tee` diverts a copy of your pipeline's output stream to an intermediate file while still passing it through its standard output.

```bash 
program1 input.txt | tee intermediate-file.txt | program2 > results.txt
```



## Background Processes

Running a process in the background frees the prompt so we can continue working.

We append an ampersand `&` to the end of the command:

```bash
program1 input.txt > results.txt &
```

The number returned by the shell is the **PID** (unique) of program1.

We can check what processes we have running in the background with `jobs`

`fg` will bring the most recent process to the foreground.

To return a specific background job to the foreground we use :

```bash
fg %<number in the joblist>
```

Although background processes run in the background, closing the terminal window would cause these processes to be killed.



## Killing and Monitoring Processes

Killing a process ends it for good, and unlike suspending it with a stop signal, it's unrecoverable.

If the process is running in the foreground in the shell, we kill it by **Control-C**. Otherwise we need to use `fg`.

`top`

`ps`

`kill`

## Warning Exit Statuses

By Unix standards, an exit status of `0` indicates the process ran successfully, and any nonzero status indicates some sort of error has occurred.

The exit status is not printed to the terminal, but the value is set to the SHELL VARIABLE `$?` 

```BASH
program1 input.txt > results.txt
echo $?
0
```

The shell operator `&&` executes subsequent commands only if previous commands have completed with a nonzero exit status.

We can execute two commands sequentially by using a single semicolon `;`

## Command Substitution

Is another type of shell expansion which runs a Unix command inline and returns the output as a string that can be used in another command. 

```bash
echo "There are $(grep -c '^>' input.fasta) entries in my FASTA file"
```



# CHAPTER 4. WORKING WITH REMOTE MACHINES



## Connecting to Remote Machines with SSH

We use ***secure shell*** because it's encrypted and it's on every Unix sysytem.

To initiate an SSH connection to a host we use the `ssh` command.

***SSH config files*** store details about hosts we frequently connect to. To create a file, just create and edit the file at  `~/.ssh/config` .

To use SSH keys to log in into remote machines without passwords we first need to generate a public/private key pair with the command `ssh-keygen` 

## Maintaining Long-Running Jobs with `nohup` 

The `nohup` command ignores hangup signals, avoiding the interruption of the program we are running.

We use it adding `nohup` before the command we want to run:

`nohup program1 > output.txt &`

Alternatively we can use `tmux` : Tmux allows you to have multiple sessions and within each session have multiple windows. 

```bash
tmux new-session -s <name> #-s option gives this session a name
```



# CHAPTER 5. Git

Git is a version control system suited for project version control and collaborative work.

With version control systems, you create snapshots of your current project at specific points. Git commits allow you to easily reproduce and rollback to past versions of analysis, giving you line-by-line code differences across versions. 

```bash
git config --global user.name 'idelvalle'
git config --global user.email 'ignacio.delvalle.torres@gmail.com' #use email associated with GitHub account
git config --global --list
```



## Git Clients

Git and your Git client are not the same thing, just like R and RStudio are not the same thing.

The visual overview given by your Git client can also be invaluable for understanding the current state of things, even when preparing calls to command line Git.

You can literally do one operation from the command line, do another from RStudio, and another from GitKraken, one after the other, and it just works.

GitKraken works across all three OSes: Windows, Mac, and Linux.



## Connect to GitHub



### 1. Make a repository on GitHub

- Click green “New repository” button. Or, if you are on your own profile page, click on “Repositories”, then click the green “New” button.

- Repository name: `Bioinformatics-Summary` 

  **Public**

  **YES** Initialize this repository with a README

- Click big green button “Create repository.”

- Copy the HTTPS clone URL to your clipboard via the green “Clone or Download” button.



### 2. Clone the repo to your local computer

- Go to  `~/tmp`.

- Clone `Bioinformatics-Summary` from GitHub to your computer using **Clone with HTTPS**:

```bash
git clone https://github.com/idelvalle/Bioinformatics-Summary.git
cd Bioinformatics-Summary
git remote show origin


```



The `-m` flag is an important message that must be included with every commit, Git requires a commit message for every commit.



### 3. Make a local change, commit, and push

```bash
echo "Summary of important Pipelines" >> README.md
git status

On branch master
Your branch is up-to-date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

#Commit change and push to remot repo on GitHub

git add -A #stages ALL

git commit -m "First line"

[master 836bd5b] First line
 1 file changed, 1 insertion(+), 1 deletion(-)

git push

Username for 'https://github.com': idelvalle
Password for 'https://idelvalle@github.com': 
Counting objects: 3, done.
Delta compression using up to 24 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 297 bytes | 297.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/idelvalle/Bioinformatics-Summary.git
   a8ab645..836bd5b  master -> master
 
```



 ### 4. Confirm Changes on GitHub

 Go back to browser and refresh. We can also click on `commits`



### 5. Clean Up

***Locally***: Remove repository's directory

`rm -rf  RepositoryName`

***GitHub***:  Go to the repo’s landing page on GitHub. Click on `Settings`. Scroll down, click on `Delete this repository` and do as it asks.



## Set up Keys for SSH

SSH keys are nearly impossible to decipher by brute force alone. Generating a key pair provides you with two long strings of characters: a public and a private key. 

You can place the public key on any server (like GitHub), and then unlock it by connecting to it with a client that already has the private key (your computer). 

When the two match up, the system unlocks without the need for a password. You can increase security even more by protecting the private key with a passphrase.

```bash
ls -al ~/.ssh/ #list existing keys
```

If we see a pair of files like `id_rsa.pub` and `id_rsa`,  we have a key pair already. The typical pattern is `id_FOO.pub` (the public key) and `id_FOO` (the private key). 

```bash
ssh-keygen -t rsa -b 4096 -C "Key for GHUB"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/nacho/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/nacho/.ssh/id_rsa.
Your public key has been saved in /home/nacho/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:q6Z2J3L8RBA7RSGPEkMMKaa5IPaFilPQz8iXqXOYieA Key for GHUB
The key's randomart image is:
+---[RSA 4096]----+
| ..=+ o.+.       |
|o.o .o B         |
|o= +oo= .        |
|*.+.*o o         |
|B+o*.   S        |
|=E*..  . .       |
| . o .  o        |
|    o *o.        |
|   ..*.+.        |
+----[SHA256]-----+
```

We can now add the key to the `ssh-agent`

```bash
eval "$(ssh-agent -s)" #confirm ssh-agent is enabled
Agent pid 17216
ssh-add ~/.ssh/id_rsa #add the key
Identity added: /home/nacho/.ssh/id_rsa (/home/nacho/.ssh/id_rsa)
```

 We can finally store a copy of the public key on GitHub:

```bash
xclip -sel clip < ~/.ssh/id_rsa.pub #copy the public key onto the keyboars
```

Make sure you’re signed into GitHub. Click on your profile pic in upper right corner and go `Settings`, then `SSH and GPG keys`. Click `New SSH key`. Paste your public key in the “Key” box. Giving it an informative title. 

Test the connection by using:

```bash
ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.253.113' (RSA) to the list of known hosts.
Hi idelvalle! You've successfully authenticated, but GitHub does not provide shell access.
```



## Connect RStudio to Git and GitHub

1. In RStudio, **start a new Project**:

- *File > New Project > Version Control > Git*. In the `repository URL` paste the URL of your new GitHub repository.
- Notice the local directory for the project
- Check `Open in new session`
- Click `Create Project`

2. After making any changes we should **save and commit them**:

   From RStudio:

   Click the `Git` tab in upper right pane.

   Check `Staged` box for  the modified file.

   Click `Commit`.

   Type a message in `Commit message`.

   Click `Commit`.

3. **Push** local changes to GitHub:

   Click the green “Push” button to send your local changes to GitHub.

4. **Confirm** local changes refresshing the browser.







# CHAPTER X. R and R STUDIO

`update.packages(ask = FALSE, checkBuilt = TRUE) #update R packages `



 





