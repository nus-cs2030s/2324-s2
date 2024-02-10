# CS2030S Exercise Guide

## GitHub Setup 

You need a one-time setup at the beginning of the semester to link your PE account to your GitHub account.  Follow [the instructions here](github.md) to set up your GitHub account for CS2030S.

## Vim Setup 

You need a one-time setup at the beginning of the semester to install the standard vim configuration, color schemes, and plugins.  Follow [the instructions here](vim/setup.md) to set up your Vim for CS2030S.

You will not be able to retrieve an exercise if the expected Vim-related directory cannot be found. 

## GitHub Classroom

We will use GitHub Classroom for our exercise release and submission for CS2030S.

Here are what you need to do for every exercise:

### 1. Accept the Exercise

Make sure that you have logged into GitHub.

_If you have multiple GitHub accounts, make sure you use the one with the same GitHub username you have submitted to us_.

Click on the given URL to accept the exercise. 

A repo will be created automatically for you.

!!! warning "WARNING"
    Do not interact with the repo directory using GitHub or other `git` commands.


### 2. Read and Understand the Exercise Task

The exercise task will be given in a link on Canvas.

Read through the question carefully before starting to complete the task.


### 3. Get a Copy on PE Hosts

Run the command `~cs2030s/get exX` (where X is the exercise number) to clone a copy of the exercise on your home directory.  You will see a new directory named something like `exX-username` created, with the skeleton files inside.

You need to edit, compile, run, and test your code on the PE hosts.

!!! warning "WARNING"
    Do not edit your code directly on GitHub.

### 4. Submit a Copy 

When you are ready to submit, run `~cs2030s/submit exX` (where X is the exercise number).  This will submit a copy to GitHub.  You can submit multiple times, but only the last copy will be graded.

!!! warning "WARNING"
    Do not use `git push` or other `git` commands to submit your code to GitHub.

### 5. Receiving Feedback

The tutors will provide feedback on your submission via Github after the deadline.  You can reply to their comment, etc, on GitHub as well.  Communicate with your grader via Ed (or directly if they prefer) if you think the grading is unfair.

!!! warning "WARNING"
    Do not change your code on GitHub after the deadline (by either re-running `submit` or using `git` commands directly).  If you wish to improve upon your code after feedback from the tutors, replicate it in your own personal repo.

Your submission will also be automatically graded by a bot.

### 6. Feedback Report and Achievement Badges

A file named `feedback.md` that contains auto-graded output as well as links to your grader's feedback will be placed into your GitHub repo.  

Your submission will earn an achievement badge, which can be one of the following:

- **Excellent** Compiles without warning or style errors.  Pass all test cases.
- **Good** Compiles with one or more warnings or style errors.  Pass all test cases.
- **Need Improvement** Fail at least one test case {++ including internal test cases ++}
- **N/A** Late submission; Submitted skeleton only; No submission; Submitted non-compilable code.

The difficulty level of each exercise and the achievement level will determine your exercise grade.

### Warning

If it is not clear to you by now, let us repeat: You should only interact with your submissions on GitHub using the provided scripts `get` and `submit`.  Failure to do so will break our workflow and will not be appreciated.  We may deduct marks for students who do not follow this instruction.

If you accidentally break your repo by running `git` commands on it or edit it directly on GitHub, you should save a copy of your code elsewhere, then reset your exercise directory, by (i) requesting your tutor to delete the repo on GitHub, (ii) deleting the corrupted directory on PE nodes, (iii) go through Steps 1 and 2 again, then copy back your edited code into the directory.

### Dealing with Submission Errors

It is the student's responsibility to ensure that the code is submitted correctly, by checking the GitHub website.  There is no grace period for students to get familiarized with the code submission process.  However, every student is allowed to miss 10% of the total exercise marks (and still earn the 5 marks towards the final grade).

### Dealing with Broken Computers at Home, etc.

The School of Computing has more than 350 computers in various programming labs for students to use.  We do not accept excuses such as broken computers or the Internet at home as justification for late submission.  You can search for available slots in which a programming lab does not hold a class via [nusmods](https://nusmods.com/venues).

## Timeline

The exercise is released sometime before Thursday 8 AM, with a deadline (usually Tuesday night in the following week) given.  You must submit each exercise before the deadline to receive your achievement badge.

No late submission is accepted for all CS2030S formative assessments, include lab exercises.  Students, however, are given the leeway to miss 10% of all exercise marks.

## General Advice

You are advised to (i) spend some time thinking before you begin coding, (ii) practice incremental coding, and (iii) test your programs thoroughly.

Remember to spend some time thinking about the design and approach to solving each question.

Incremental coding means do NOT type in the whole long program in a single session and then compile it. Instead, type your program in bits and pieces and compile it incrementally. Try to maintain a compilable program even while you are working on it. Submitting a compilable program that partially works is better than submitting an un-compilable one; this is especially important for your practical exams.

You should test your program thoroughly with your test data before submission.

You may assume that all input data are correct unless otherwise stated. Hence, you do NOT need to do input data validation. This is to allow you to focus on getting the program right, instead of worrying about making your program fool-proof.

## Peer-Learning

We encourage students to discuss and seek help from each other and from the lab tutors if they are stuck.  Ed is a great forum for that.  However, do note that while students are encouraged to _discuss the approach_ to the solution, students are _expected to write their code independently_.  Copy-pasting of code or coming up with the code together, line-by-line, is considered [plagiarism](lab.md#plagiarism).

All exercises are designed to be completed within half a day.  If you get stuck on an issue for longer than that, you should talk to others.

## Submissions with Compilation Errors

Writing code that compiles without any compilation error is the most basic requirement for all our exercises.  You will get the lowest achievement badge if your code cannot be compiled.

## Identifying Yourself

In every Java file that you submit, you need to identify yourself by writing your name and lab group.  You need to edit the line:

```
@author XXXX (Group YYYY)
```

and change it to something like:

```
@author Gamora (Group 9A)
```

## Method of Submission

Please follow the instructions above to submit your homework.  Programs submitted through other means, such as emails, will NOT be accepted.

## Use of Ed

If you have doubts about the problem statements of an assignment, you may raise them on Ed.  But before that, please read through the problem statements carefully first, and check if the same questions have been asked and answered on the forum.

Please exercise discretion when posting to Ed.  Before the deadline, you are NOT to post the solution to the assignment, complete or partial, on Ed (or any publicly accessible online site).

## Disallowed Syntax

Some exercises may explicitly disallow the use of certain syntax.  If the objective of the assignment is undermined due to the use of forbidden syntax, the penalty will be heavy. If in doubt, please ask for clarification on Piazza.

## Plagiarism

While we encourage discussions among students for programming assignments, each student should be responsible for writing his/her own code and should give credit to others when appropriate.

NUS and the School of Computing have a high standard of academic honesty and take any violation seriously. In the context of computing modules, source code plagiarism -- copying code from another source and attributing it as one's own code -- is a serious violation. Please read the page [Preventing Plagiarism](https://www.comp.nus.edu.sg/cug/plagiarism/) from the school's website to familiarize yourself with the policy.

We adopt a "no mercy" policy when it comes to disciplinary action on plagiarism. Both parties, the student who copied, and the student who allowed others to copy, will be penalized equally.

This means that you should also guard your solution carefully, not posting them to publicly accessible places, or changing the permissions of the files on the PE hosts so that it is accessible by others.

Copying others' programs will only offer a short-term reprieve. When Practical Exam (PE) time comes, your inadequacy will be exposed and the consequence will be dire.
