# CS2030/S Java Style Guide

## Why Coding Style is Important

One of the goals of CS2030/S is to move you away from the mindset that you are writing code that you will discard after it is done (_e.g., in CS1101S missions_) and you are writing code that no one else will read except you and your tutor.  CS2030/S prepares you to work in a software engineering teams in many ways. One of the ways is to enforce a consistent coding style.

If everyone on the team follows the same style, the intend of the programmer can become clear (_e.g., is this a class or a field?_), the code is more readable and less bug prone (_e.g., see the [Apple `goto fail` bug](https://www.wired.com/2014/02/gotofail/) for an example of buggy program due to style_).  Empirical studies support this:

!!! quote
    "It is not merely a matter of aesthetics that programs should be written in a particular style. Rather there is a psychological basis for writing programs in a conventional manner: programmers have strong expectations that other programmers will follow these discourse rules. If the rules are violated, then the utility afforded by the expectations that programmers have built up over time is effectively nullified. The results from the experiments with novice and advanced student programmers and with professional programmers described in this paper provide clear support for these claims."

    Elliot Soloway and Kate Ehrlich. "Empirical studies of programming knowledge." IEEE Transactions on Software Engineering 5 (1984): 595-609.

Many major companies enforce coding styles, and some have published them.  For CS2030, we base our (_simplified_) coding style on [Google's Java Coding Style](https://google.github.io/styleguide/javaguide.html).  You should bookmark the link because you need to come back to it again and again.

## CS2030/S Coding Style

1. No tabs.  Use only whitespace.  

    !!! info "VIM Setting"
        For `vim` users, you can add the following line in your `~/.vimrc` file:
        
        ```
        set expandtab
        ```
        
        So that when you press ++tab++ it is expanded to whitespace.
    
    !!! note "For CS2030S"
    
        This option is already included in the `~/.vimrc` file if you follow the [Vim setup guide](vim/setup.md). In fact, we recommend you follow the [Vim setup guide](vim/setup.md) instead and don't manually manage your own version of your `~/.vimrc` file.

        Most other source code editors have similar configuration.  

2. Exactly one blank line after import statements and exactly one top-level (_i.e., non-nested_) class.

3. Each top-level class resides in a source file of its own.

4. When a class has overladed methods (_e.g., multiple constructors or methods of the same name_), they appear sequentially with no other code in between.

5. Braces are always used (_even if the body is empty or contains a single statement_)

6. Use "[Egyptian brackets](https://en.wikipedia.org/wiki/Indentation_style#K&R_style)": 

    - Opening brace have no line break before; but has line break after
    - Closing brace has a line break before; and has a line break after (_except when there is `else` or comma following a closing brace_).

    !!! success "Good Example"
        ```Java
        if (x == 0) {
          x++;
        } 
        ```

    !!! failure "Bad Example"
        ```Java
        if (x == 0) { x++; }
        ```
        ```Java
        if (x == 0) // Allman style (do not use)
        {
          x++;
        }
        ```
        ```Java
        if (x == 0) // Pico style (do not use)
        {
          x++; }
        ```

7. Block indentation is exactly two spaces.

    ```Java
    if (x == 0) { 
      x++;
      for (i = 0; i < x; i++) {
        x += i;
      }
    }
    ```

    !!! info "VIM Setting"
        For `vim` users, in `~/.vimrc`, add the following:
        
        ```
        set tabstop=2
        set shiftwidth=2
        set autoindent
        set smartindent
        " For Java: enabling this includes Java-specific indentation settings that handles annotations like @Override
        filetype plugin indent on
        ```

        To help you with indentation.

    
    !!! note "For CS2030S"
    
        These options are already included in the `~/.vimrc` file if you follow the [Vim setup guide](vim/setup.md). In fact, we recommend you follow the [Vim setup guide](vim/setup.md) instead and don't manually manage your own version of your `~/.vimrc` file.
    
        Most other source code editors have similar configuration.  

8. Each statement is followed by a line break, no matter how short the statement is.

    !!! success "Good Example"
    
        ```Java
        x++; 
        i++;
        ```

    !!! failure "Bad Example"

        ```Java
        x++; i++;
        ```

9. Each line is limited to 80 characters in length.  You can break a long
    line into multiple lines to enhance readability, this is called _line wrapping_.  When you do so, each continuation line is indented at least 4 spaces from the original line.

    !!! success "Good Example"

        ```Java
        System.out.println("Daenerys of the House Targaryen," + 
            "the First of Her Name, The Unburnt, Queen of the Andals," +
            "the Rhoynar and the First Men, Queen of Meereen," +
            "Khaleesi of the Great Grass Sea, Protector of the Realm," +
            "Lady Regnant of the Seven Kingdoms, Breaker of Chains and" +
            "Mother of Dragon");
        ```
        
        Note the 4 spaces indentation at line 2 to line 6.

    !!! failure "Bad Example"
    
        ```Java
        System.out.println("Daenerys of the House Targaryen, the First of Her Name, The Unburnt, Queen of the Andals, the Rhoynar and the First Men, Queen of Meereen, Khaleesi of the Great Grass Sea, Protector of the Realm, Lady Regnant of the Seven Kingdoms, Breaker of Chains and Mother of Dragon");
        ```

        ```Java
        System.out.println("Daenerys of the House Targaryen, the First of" +
        " Her Name, The Unburnt, Queen of the Andals, the Rhoynar and the" +
        " First Men, Queen of Meereen, Khaleesi of the Great Grass Sea, P" +
        "rotector of the Realm, Lady Regnant of the Seven Kingdoms, Break" +
        "er of Chains and Mother of Dragon");
        ```
        
        This bad example omitted the 4 spaces indentations.

    !!! note "80 vs 100"
        While we prefer lines to be limited to 80, we are OK if the length is up to 100.  Any longer, however, will be frowned upon.

10. There should be a blank line between constructors, methods, nested classes and static initializers.  Blank lines can be used between fields to create logical groupings.

11. White space should separate Java keywords from parenthesis and braces, and be added on both sides of binary operators (_`+`, `-`, `/`, etc_) as well as `:` in enhanced for-loop.  Space should also appears before and after `//` comments

    !!! success "Good Example"

        ```Java
        if (x == 0) { 
          x++; // to make sure x is at least one.
          for (i = 0; i < x; i++) {
            x += i;
          }
        }
        ```

    !!! failure "Bad Example"

        ```Java
        if(x==0){ 
          x++;//to make sure x is at least one.
          for(i=0;i<x;i++){
            x+=i;
          }
        }
        ```

12. One variable per declaration.  

    !!! success "Good Example"

        ```Java
        int x;
        int y;
        ```

    !!! failure "Bad Example"

        ```Java
        int x, y;
        ```

13. No C-style array declaration

    !!! success "Good Example"

        ```Java
        String[] args;
        ```

    !!! failure "Bad Example"

        ```Java
        String args[];
        ```

14. Switch statement always include a `default` case.

15. One annotation per line.

16. Always use `@Override`.

    ```Java
    @Override
    public boolean equals(Object o) {
        :
    }
    ```

17. Indent comments at the same level as the surrounding code.  For multiple comments, align `*` with the previous line.

    !!! success "Good Example"

        ```Java
        /*
         * Good style
         */
        ```

        ```Java
        /**
         * Good style for JavaDoc
         */
        ```

    !!! failure "Bad Example"

        ```Java
        /*
        * Not a good style
        */
        ```

        ```Java
        /**
        * Not a good style for JavaDoc
        */
        ```

        ```Java
        /**
          * Also not a good style for JavaDoc
          */
        ```

18. Class modifier appears in the following order:

    ```Java
    public protected private abstract default static final transient volatile synchronized native strictfp
    ```

    !!! success "Good Example"
    
        ```Java
        public static void main(String[] args)
        ```

    !!! failure "Bad Example"
    
        ```Java
        static public void main(String[] args)
        ```

19. Class names are writte in UpperCamelCase, method names and field names in lowerCamelCase, constant names in ALL_CAPS_SNAKE_CASE.  Type parameters in single capital letter.

20. Caught exceptions should not be ignored.

21. Static fields and methods must be accessed with class name.

## Using `checkstyle`

To automatically check for style violation, we use a tool call [`checkstyle`](http://checkstyle.org).

To run, 

```
java -jar ~cs2030s/bin/checkstyle.jar -c ~cs2030s/bin/cs2030_checks.xml *.java 
```

Hint: put the command into a `bash` script so that you do not need to type such a long string all the time.
