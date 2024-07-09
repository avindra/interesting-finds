TURING AWARD LECTURE

# Reflections on Trusting Trust

_To what extent should one trust a statement that a program is free of Trojan horses? Perhaps it is more important to trust the people who wrote the software._

**KEN THOMPSON**
(_Converted to markdown by John Martinez, 2024_)

## INTRODUCTION

I thank the ACM for this award. I can't help but feel that I am receiving this honor for timing and serendipity as much as technical merit. UNIX swept into popularity with an industry-wide change from central mamframes to autonomous minis. I suspect that Daniel Bobrow[^1] would be here instead of me if he could not afford a PDP-10 and had had to "settle" for a PDP-11. Moreover, the current state of UNIX is the result of the labors of a large number of people.

There is an old adage, "Dance with the one that brought you," which means that I should talk about UNIX[^2]. I have not worked on mainstream UNIX in many years, yet I continue to get undeserved credit for the work of others. Therefore, I am not going to talk about UNIX, but I want to thank everyone who has contributed.

That brings me to Dennis Ritchie. Our collaboration has been a thing of beauty. In the ten years that we have worked together, I can recall only one case of miscoordination of work. On that occasion, I discovered that we both had written the same 20-line assembly language program. I compared the sources and was astounded to find that they matched character-for-character. The result of our work together has been far greater than the work that we each contributed.

I am a programmer. On my 1040 form, that is what I put down as my occupation. As a programmer, I write programs. I would like to present to you the cutest program I ever wrote. I will do this in three stages and try to bring it together at the end.

## STAGE I

In college, before video games, we would amuse ourselves by posing programming exercises. One of the favorites was to write the shortest self-reproducing program. Since this is an exercise divorced from reality, the usual vehicle was FORTRAN. Actually, FORTRAN was the language of choice for the same reason that three-legged races are popular.

More precisely stated, the problem is to write a source program that, when compiled and executed, will produce as output an exact copy of its source. If you have never done this, I urge you to try it on your own. The discovery of how to do it is a revelation that far surpasses any benefit obtained by being told how to do it. The part about "shortest" was just an incentive to demonstrate skill and determine a winner.

Figure 1 shows a self-reproducing program in the C[^3] programming language. (The purist will_note that the program is not precisely a self-reproducmg program, but will produce a self-reproducing program.) This entry is much too large to win a prize, but it demonstrates the technique and has two important properties that I need to complete my story: 1) This program can be easily written by another program. 2) This program can contain an arbitrary amount of excess baggage that will
be reproduced along with the main algorithm. In the example, even the comment is reproduced.

```c
chars[] = {
    '\t',
    '0',
    '\n',
    '}',
    ';',
    '\n',
    '\n',
    '/',
    '*',
    '\n',
    (213 lines deleted)
    0
};

/*
* The string s is a
* representation of the body
* of this program from '0'
* to the end.
*/

main()
{
    int i;

    printf("char\ts[ ] = {\n");
    for(i=O; s[i]; i++)
        printf("\t%d, \n", s[i]);
    printf("%s", s);
}

Here are some simple transliterations to allow
a non-C programmer to read this code.

= assignment
== equal to .EQ.
!= not equal to .NE.
++ increment
'x' single character constant
"xxx" multiple character string
%d format to convert to decimal
%s format to convert to string
\t tab character
\n newline character
```
<p style="text-align: center;"><bold>FIGURE 1</bold></p>

## STAGE II

The C compiler is written in C. What I am about to describe is one of many "chicken and egg" problems that arise when compilers are written in their own language. In this case, I will use a specific example from the C compiler.

C allows a string construct to specify an initialized character array. The individual characters in the string can be escaped to represent unprintable characters. For example,

`"Hello world\n"`

represents a string with the character "\n," representing
the new line character.

Figure 2.1 is an idealization of the code in the C compiler that interprets the character escape sequence. This is an amazing piece of code. It "knows" in a completely portable way what character code is compiled for a new line in any character set. The act of knowing then allows it to recompile itself, thus perpetuating the knowledge.

Suppose we wish to alter the C compiler to include the sequence `"\v"` to represent the vertical tab character. The extension to Figure 2.1 is obvious and is presented in Figure 2.2. We then recompile the C compiler, but we get a diagnostic. Obviously, since the binary version of the compiler does not know about `"\v,"` the source is not legal C. We must "train" the compiler. After it "knows" what `"\v"` means, then our new change will become legal C. We look up on an ASCII chart that a vertical tab is decimal 11. We alter our source to look like Figure 2.3. Now the old compiler accepts the new source. We install the resulting binary as the new official C compiler and now we can write the portable version the way we had it in Figure 2.2.

This is a deep concept. It is as close to a "learning" program as I have seen. You simply tell it once, then you can use this self-referencing definition.

## STAGE III

Again, in the C compiler, Figure 3.1 represents the high level control of the C compiler where the routine "compile" is called to compile the next line of source. Figure 3.2 shows a simple modification to the compiler that will deliberately miscompile source whenever a particular pattern is matched. If this were not deliberate, it would be called a compiler "bug." Since it is deliberate, it should be called a "Trojan horse."

```c
...
c = next();
if(c != '\\')
    return(c);
c = next();
if(c == '\\')
    return('\\');
if(c == 'n')
    return('\n');
...
```
<p style="text-align: center;"><bold>FIGURE 2.1</bold></p>

```c
...
c = next();
if(c != '\\')
    return(c);
c = next();
if(c == '\\')
    return('\\');
if(c == 'n')
    return('\n');
if(c == 'v')
    return('\v');
...
```
<p style="text-align: center;"><bold>FIGURE 2.2</bold></p>

```c
...
c = next();
if(c != '\\')
    return(c);
c = next();
if(c == '\\')
    return('\\');
if(c == 'n')
    return('\n');
if(c == 'v')
    return(11);
...
```
<p style="text-align: center;"><bold>FIGURE 2.3</bold></p>


The actual bug I planted in the compiler would match code in the UNIX "login" command. The replacement code would miscompile the login command so that it would accept either the intended encrypted password or a particular known password. Thus if this code were installed in binary and the binary were used to compile the login command, I could log into that system as any user.

Such blatant code would not go undetected for long. Even the most casual perusal of the source of the C compiler would raise suspicions.

The final step is represented in Figure 3.3. This simply adds a second Trojan horse to the one that already exists. The second pattern is aimed at the C compiler. The replacement code is a Stage I self-reproducing program that inserts both Trojan horses into the compiler. This requires a learning phase as in the Stage II example. First we compile the modified source with the normal C compiler to produce a bugged binary. We install this binary as the official C. We can now remove the bugs from the source of the compiler and the new binary will reinsert the bugs whenever it is compiled. Of course, the login command will remain bugged with no trace in source anywhere.

```c
compile(s)
char •s;
{
    ...
}
```
<p style="text-align: center;"><bold>FIGURE 3.1</bold></p>

```c
compile(s)
char •s;
{
    if(match(s, "pattern")) {
        compile("bug");
        return;
    }
    ...
}
```
<p style="text-align: center;"><bold>FIGURE 3.2</bold></p>

```c
compile(s)
char•s;
{
    if(match(s, "pattern1 ")) {
        compile ("bug1");
        return;
    }
    if(match(s, "pattern 2")) {
        compile ("bug2");
        return;
    }
    ...
}
```
<p style="text-align: center;"><bold>FIGURE 3.3</bold></p>



## MORAL

The moral is obvious. You can't trust code that you did not totally create yourself. (Especially code from companies that employ people like me.) No amount of source-level verification or scrutiny will protect you from using untrusted code. In demonstrating the possibility of this kind of attack, I picked on the C compiler. I could have picked on any program-handling program such as an assembler, a loader, or even hardware microcode. As the level of program gets lower, these bugs will be harder and harder to detect. A well-installed microcode bug will be almost impossible to detect.

After trying to convince you that I cannot be trusted, I wish to moralize. I would like to criticize the press in its handling of the "hackers," the 414 gang, the Dalton gang, etc. The acts performed by these kids are vandalism at best and probably trespass and theft at worst. It is only the inadequacy of the criminal code that saves the hackers from very serious prosecution. The companies that are vulnerable to this activity, (and most large companies are very vulnerable) are pressing hard to update the criminal code. Unauthorized access to computer systems is already a serious crime in a few states and is currently being addressed in many more state legislatures as well as Congress.

There is an explosive situation brewing. On the one hand, the press, television, and movies make heros of vandals by calling them whiz kids. On the other hand, the acts performed by these kids will soon be punishable by years in prison.

I have watched kids testifying before Congress. It is clear that they are completely unaware of the seriousness of their acts. There is obviously a cultural gap. The act of breaking into a computer system has to have the same social stigma as breaking into a neighbor's house. It should not matter that the neighbor's door is unlocked. The press must learn that misguided use of a computer is no more amazing than drunk driving of an automobile.

**Acknowledgment.** I first read of the possibility of such a Trojan horse in an Air Force critique[^4] of the security of an early implementation of Multics. I cannot find a more specific reference to this document. I would appreciate it if anyone who can supply this reference would let me know.

### REFERENCES

[^1]: Bobrow, D.G .. Burchfiel. J.D., Murphy, D.L., and Tomlinson, R.S. TENEX, a paged time-sharing system for the PDP-10. Commun. ACM 15, 3 (Mar. 1972), 135-143.
[^2]: Ritchie, D.M., and Thompson, K. The UNIX time-sharing system. Commun. ACM 17, (July 1974), 365-375.
[^3]: Kernighan, B.W .. and Ritchie, D.M. The C Programming Language. Prentice-Hall, Englewood Cliffs, N.J., 1978.
[^4]: Unknown Air Force Document.

_Author's Present Address: Ken Thompson, AT&T Bell Laboratories, Room ZC-519, 600 Mountain Ave., Murray Hill. NJ 07974._

Permission to copy without fee all or part of this material is granted provided that the copies are not made or distributed for direct commercial advantage, the ACM copyright notice and the title of the publication and its date appear, and notice is given that copying is by permission of the Association for Computing Machinery. To copy otherwise, or to republish, requires a fee and/or specific permission.
