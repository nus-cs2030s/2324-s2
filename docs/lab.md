# CS2030S Lab Guide

## GitHub Setup 

You need a one-time setup at the beginning of semester to link your PE account to your GitHub account.  Following [the instructions here](github.md) to set up your GitHub account for CS2030S.

## GitHub Classroom

We will use GitHub Classroom for our lab release and submission for CS2030S.

Here are what you need to do for every lab assignment:

### 1. Accept the Lab Assignment

Make sure that you have logged into GitHub.  

_If you have multiple GitHub accounts, make sure you use the one with the same GitHub username you have submitted to us_.

Click on the given URL to accept the lab. 

A repo will be created automatically for you.

!!! warning "WARNING"
    Do not interact with the lab repo directory using GitHub or other `git` commands.


### 2. Read and Understand the Lab Question

The lab question will be given in a link on Canvas.

Read through the question carefully before starting your lab.


### 3. Get a Copy on PE Hosts

For your first-ever lab, you will need to create a configuration file (see the [GitHub Setup](github.md) guide). After that, proceed as per normal below.

Run the command `~cs2030s/get-labX` (where X is the lab number) to clone a copy of the lab on your home directory.  You will see a new lab directory named something like `labX-username` created, with the skeleton files and questions inside.

You can edit the code, compile, test, etc, on the PE hosts.

You can open two files side-by-side with `vim -O file1 file2` or in different tabs with `vim -p file1 file2`.

!!! warning "WARNING"
    Do not edit your code directly on GitHub.

### 4. Submit a Copy 

When you are ready to submit, run `~cs2030s/submit-labX` (where X is the lab number).  This will submit a copy to GitHub.  You can submit multiple times, but only the last copy will be graded.

!!! warning "WARNING"
    Do not use `git push` or other `git` commands to submit your code to GitHub.

### 5. Receiving Feedback

The tutors will grade and comment on your submission on Github after the deadline.  You should receive both your comments and your preliminary marks on GitHub.  You can reply to their comment, etc, on GitHub as well.  Communicate with your grader via Piazza (or directly if they preferred) if you think the grading is unfair.

!!! warning "WARNING"
    Do not change your code on GitHub after the deadline (by either re-running `submit-labX` or using `git` commands directly).  If you wish to improve upon your code after feedback from the tutors, replicate it in your own personal repo.

### 6. Receiving Final Grades

A file named `feedback.md` summarizing your marks will be placed into your GitHub repo.  Your marks will be posted on Canvas Gradebook.

### Warning

If it is not clear to you by now, let us repeat: You should only interact with your lab submissions on GitHub using the provided scripts `get-labX` and `submit-labX`.  Failure to do so will break our workflow and will not be appreciated.  We may deduct marks for students who do not follow this instruction.

If you accidentally break your repo by running `git` commands on it or edit it directly on GitHub, you should save a copy of your code elsewhere, then reset your lab directory, by (i) requesting your tutor to delete the repo on GitHub, (ii) deleting the corrupted lab directory on PE nodes, (iii) go through Steps 1 and 2 again, then copy back your edited code into the lab directory.

### Grace Period

You have until Lab 2 to get familiarized with the procedure above.  From Lab 3 onwards, we will not entertain requests or appeals for students who failed to follow the procedure (e.g., forgot to run the `submit` command, submitted the wrong lab, wrong password).

## Lab Timeline

The lab assignment is released before every Thursday at 8 am, with a deadline (usually Tuesday night in the following week) given.  You must submit each lab assignment before the deadline.

## General Advice

You are advised to (i) spend some time thinking before you begin coding, (ii) practice incremental coding, and (iii) test your programs thoroughly.

Remember to spend some time thinking about the design and approach to solving each question.

Incremental coding means do NOT type in the whole long program in a single session and then compile it. Instead, type your program in bits and pieces and compile it incrementally. Try to maintain a compilable program even while you are working on it. Submitting a compilable program that partially works is better than submitting an un-compilable one; this is especially important for your practical exams.

You should test your program thoroughly with your test data before submission.

You may assume that all input data are correct unless otherwise stated. Hence, you do NOT need to do input data validation. This is to allow you to focus on getting the program right, instead of worrying about making your program fool-proof.

## Peer-Learning

We encourage students to discuss and seek help from each other and from the lab tutors if they are stuck.  Piazza is a great forum for that.  However, do note that while students are encouraged to _discuss the approach_ to the solution, students are _expected to write their code independently_.  Copy-pasting of code or coming up with the code together, line-by-line, is considered [plagiarism](lab.md#plagiarism).

All labs are designed to be completed within half a day.  If you get stuck on an issue for longer than that, you should talk to others.

## Late Submission

All lab assignments must be submitted on time.  If you need an extension, please ask for one and provide a justification for approval.  Only academic reasons and compassionate reasons can be considered (e.g., representing NUS for a sports event is OK; Attending a wedding is not.)

For late submission, there is a 1% penalty (of the total awarded marks for that particular assignment) for every 5 minutes after the deadline, capped at 80%.  For example, if an assignment is awarded 40 marks, and it is submitted 100 minutes after the deadline, the student will get 32 marks instead (20% penalty).  If it is submitted 10 hours after the deadline, the student will get 8 marks (as it has hit the cap of 80% penalty).

Late submission is no longer accepted one week after the deadline.

## Submissions with Compilation Errors

Writing code that compiles without any compilation error is the most basic requirement for all our labs and practical assessments.  You will get 0 marks for the corresponding question if your code cannot be compiled.

## Identifying Yourself

In every Java file that you submit, you need to identify yourself by writing your name and lab group. Marks may be deducted if you fail to do so. You need to edit the line:

```
@author XXXX (Group YYYY)
```

and change it to something like:

```
@author Gamora (Group 9A)
```

## Method of Submission

Please follow the instructions above to submit your homework.  Programs submitted through other means, such as emails, will NOT be accepted.

## Use of Piazza

If you have doubts about the problem statements of an assignment, you may raise them on Piazza.  But before that, please read through the problem statements carefully first, and check if the same questions have been asked and answered on the forum.

Please exercise discretion when posting to Piazza.  Before the deadline, you are NOT to post the solution to the assignment, complete or partial, on Piazza (or any publicly accessible online site).

## Disallowed Syntax

Some lab assignments may explicitly disallow the use of certain syntax.  If the objective of the assignment is undermined due to the use of forbidden syntax, the penalty will be heavy. If in doubt, please ask for clarification on Piazza.

## Plagiarism

While we encourage discussions among students for programming assignments, each student should be responsible for writing his/her own code and should give credit to others when appropriate.

NUS and the School of Computing have a high standard of academic honesty and take any violation seriously. In the context of computing modules, source code plagiarism -- copying code from another source and attributing it as one's own code -- is a serious violation. Please read the page [Preventing Plagiarism](https://www.comp.nus.edu.sg/cug/plagiarism/) from the school's website to familiarize yourself with the policy.

We adopt a "no mercy" policy when it comes to disciplinary action on plagiarism. Both parties, the student who copied, and the student who allowed others to copy, will be penalized equally.

This means that you should also guard your solution carefully, not posting them to publicly accessible places, or changing the permissions of the files on the PE hosts so that it is accessible by others.

Copying others' programs will only offer a short-term reprieve. When Practical Exam (PE) time comes, your inadequacy will be exposed and the consequence would be dire.
