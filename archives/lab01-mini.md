# Lab 01 Mini Problems

## Learning Objectives
- Students are able to translate an imperative/procedural code to object-oriented code.
- Students are able to write simple Java program and use `jshell` for testing and prototyping.

## Mini Problem 1
It's a new academic year and the big companies are starting to hiring for interns for next summer. You and your friend Adhy found a magical website that can tell you the minimum resume score and minimum interview score that is required to join these big boys. 

However, since the data is a bit outdated and the talents are getting better, the numbers on this website must be adjusted by a certain multiplier, depending on the type of the company:

- For MNCs, the resume score must be multiplied by 1.5 and the interview score must be multiplied by 1.5.
- For Startups, the resume score must be multiplied by 1.1 and the interview score must be multiplied by 1.4.
- For SMEs, the resume score must be multiplied by 1.2 and the interview score must be multiplied by 1.2.

Using their newest algorithm, this magical website is also able to predict one's resume and interview score by seeing their resume and a recording of their interview. Luckily, the numbers are already accurate and you don't need to adjust them. 

Given this information, you want to know which companies are willing to offer you an internship next summer. Different company has different hiring strategies: 
- For MNCs, both the resume and interview scores must be at least the same as the minimum scores.
- For Startups, the interview score must meet the minimum score. The resume score is not considered.
- For SMEs, the resume score must meet the minimum score. The interview score is not considered.

You want to print out the company names together with their requirements to standard output. Each company name must be printed on a single line using this format:

- If the company X is an MNC, the minimum resume score is YY, and the minimum interview score is ZZ, it should be printed as "[X]: YYR ZZI" (without the double quotes)
- If the company X is a Startup and the minimum interview score is ZZ, it should be printed as "(X): ZZI" (without the double quotes)
- If the company X is a SME and the minimum resume score is YY, it should be printed as "{X}: YYR" (without the double quotes)

Sample output:
```
[Google]: 120.0R 105.0I
(StartGoogle): 126.0I
[Apple]: 105.0R 90.0I
{SmallMediumApple}: 96.0R
```
Adhy wrote a procedural code as follows:
```Java
import java.util.Scanner;

/**
 * The main class for CS2030S Mini Lab 1.
 *
 * @author Adhy
 * @version CS2030 AY23/24 S1
 */
class Lab01Mini1 {
  public static final int MNC = 1;
  public static final int STARTUP = 2;
  public static final int SME = 3;

  public static void main(String[] args) {

    // Create a scanner to read from standard input.
    Scanner sc = new Scanner(System.in);

    int numOfCompanies = sc.nextInt();

    // NOTE: These four arrays are the data you should work with
    // HINT: Intead of working with these separately, you might want to combine them together
    String[] companyNames = new String[numOfCompanies]; 
    int[] industryTypes = new int[numOfCompanies]; 
    int[] minResumeScores = new int[numOfCompanies];
    int[] minInterviewScores = new int[numOfCompanies];

    // Read the data for each company from stdin
    for (int i = 0; i < numOfCompanies; i++) {
      companyNames[i] = sc.next();
      industryTypes[i] = sc.nextInt();
      minResumeScores[i] = sc.nextInt();
      minInterviewScores[i] = sc.nextInt();
    }

    int myResumeScore = sc.nextInt();
    int myInterviewScore = sc.nextInt();

    /* TODO: Refactor this poorly written for loop */
    for (int i = 0; i < numOfCompanies; i++) {
      double resumeScore = minResumeScores[i];
      double interviewScore = minInterviewScores[i];

      if (industryTypes[i] == MNC) {
        resumeScore *= 1.5;
        interviewScore *= 1.5;
        if (myResumeScore - resumeScore >= 0 &&
          myInterviewScore - interviewScore >= 0) {
          String company = "[" + companyNames[i] + "]";
          String scores = String.format("%.1fR %.1fI", resumeScore, interviewScore);
          System.out.println(company + ": " + scores);
        }
      } else if (industryTypes[i] == STARTUP) {
        interviewScore *= 1.4;
        if (myInterviewScore - interviewScore >= 0) {
          String company = "(" + companyNames[i] + ")";
          String scores = String.format("%.1fI", interviewScore);
          System.out.println(company + ": " + scores);
        }
      } else if (industryTypes[i] == SME) {
        resumeScore *= 1.2;
        if (myResumeScore - resumeScore >= 0) {
          String company = "{" + companyNames[i] + "}";
          String scores = String.format("%.1fR", resumeScore);
          System.out.println(company + ": " + scores);
        }
      }
    }

    // Clean up the scanner.
    sc.close();
  }
}
```

While the code works, it's poorly written and hard to maintain. Using concepts you've learnt so far, how can you refactor this code to make it more readable and extendable?

### Running and testing your code
To run your code, first you need to compile it:
```bash
adhy@pe111:~$ javac Lab01Mini1.java
```

If you created more class/java files, you should compile all of them together:
```bash
adhy@pe111:~$ javac *.java
```

Then, you can run the class file using the following command:
```bash
adhy@pe111:~$ java Lab01Mini1 < inputs/Lab01Mini1.1.in
```

To compare the output of your program with the expected value, you can run the following command: (remember, no news is good news!)
```bash
adhy@pe111:~$ java Lab01Mini1 < inputs/Lab01Mini1.1.in | diff - outputs/Lab01Mini1.1.out
```

If you want to test your code against multiple inputs/outputs files, you can write a simple bash script like so:
```bash
adhy@pe111:~$ for i in {1..5}; do java Lab01Mini1 < inputs/Lab01Mini1.${i}.in | diff - outputs/Lab01Mini1.${i}.out; done
```

## Mini Problem 2
Alice, a startup founder, wants to design a simple payroll system for her company. The main goal is simple: she wants to know how much money she needs to prepare to pay her employees each month. On top of that, she mentioned few other requirements, listed below:
- Each employee should have a unique consecutive ID, starting from 0.
- There are four types of employees: managers, full-timers, part-timers, and interns. 
  - Managers and full-timers are paid monthly, while part-timers and interns are paid by the hour.
  - Managers and part-timers are eligible for overtime, paid by the hour. The overtime rate is the same as their hourly rate (for managers, it's monthly salary / 4 weeks / 40 hours = monthly salary / 160, rounded down to nearest integer).

Initially, Bob was tasked to handle this project. However, since his timeline is very tight, he rushed this project and the resulting code does not adhere to basic OOP principles. Bob asked for your help to refactor his code and make it more readable and extendable. Help Bob to improve the code quality of this payroll system!

### Running and testing your code
You can find Bob's code under the `mini2` directory. The `mini2` directory contains two files: `Employee.java` and `Payroll.java`. There is no `main` method in these files; you are encouraged to run and test your program using `jshell`.

To load the files using `jshell`, you can either pass the `java` files as an argument, or open them interactively using `/open`.

```bash
# option 1
bob@pe111:~$ jshell Employee.java Payroll.java
|  Welcome to JShell -- Version 17.0.8.1
|  For an introduction type: /help intro
jshell>

# option 2
bob@pe111:~$ jshell 
|  Welcome to JShell -- Version 17.0.8.1
|  For an introduction type: /help intro
jshell> /open Employee.java
jshell> /open Payroll.java
```

If you made any changes to your code, you can easily reload them using the `/open` command (option 2) to update the class definition in your `jshell` session.

To test your program, you can create objects and call their methods:
```bash
jshell> Employee m = new Employee(1, 10000, 0, 0, 10);
m ==> Manager0
jshell> Payroll p = new Payroll();
p ==> 
jshell> p.register(m);
jshell> p
p ==> Manager0
jshell> p.getTotalSalary();
$8 ==> 10625 
```

You can find a sample `jshell` script called `Sample.jsh` in the `mini2` directory.
```
/open Employee.java
/open Payroll.java

// manager, 10000 monthly salary, 10 hr overtime
Employee m = new Employee(1, 10000, 0, 0, 10); 

// fulltimer, 5000 monthly salary
Employee ft = new Employee(2, 5000, 0, 0, 0); 

// parttimer, 40 per hr, 100 hr, 20 hr overtime
Employee pt = new Employee(3, 0, 40, 100, 20); 

// intern, 30 per hr, 50 hr
Employee i = new Employee(4, 0, 30, 50, 0); 
Payroll pr = new Payroll();

pr.register(m);
pr.register(ft);
pr.register(pt);
pr.register(i);

// pr
// pr.getTotalSalary()
```

You can load this script using the following command:
```
bob@pe111:~$ jshell Sample.jsh
|  Welcome to JShell -- Version 17.0.8.1
|  For an introduction type: /help intro

jshell> pr
pr ==> Manager0 FullTimer1 PartTimer2 Intern3

jshell> pr.getTotalSalary()
$14 ==> 21925
```