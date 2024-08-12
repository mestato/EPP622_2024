Welcome to EPP622, Fall 2024!
--------------------------------------

The purpose of this site is to provide a resource for students and instructors participating in the course. It's easier for me to update and build labs than Canvas.

[Syllabus Version 1](https://docs.google.com/document/d/1-FMBLeK94SaN2WR3EV2tai4k7xH_i_bF/edit?usp=sharing&ouid=117671002449312371819&rtpof=true&sd=true)

## Quick Log In
```
ssh username@sphinx.ag.utk.edu
```

## Quick Schedule

  |   |   | Topic | Class Format
-- | -- | -- | -- | --
1 | Tuesday | August 20 | Command Line I | In person and recorded.
2 | Thursday | August 22 | Command Line II | In person and recorded.
3 | Tuesday | August 27 | Command Line III | In person and recorded.
4 | Thursday | August 29 | Command Line IV | In person and recorded.
5 | Tuesday | September 3 | Command Line V | In person and recorded.
6 | Thursday | September 5 | No new material. | Optional work on Test 1 in class.
7 | Tuesday | September 10 | Short Read Variation I | In person and recorded. TEST 1 Due at midnight.
8 | Thursday | September 12 | Short Read Variation II | In person and recorded.
9 | Tuesday | September 17 | Short Read Variation III | In person and recorded.
10 | Thursday | September 19 | Short Read Variation IV | In person and recorded.
11 | Tuesday | September 24 | KMers | In person and recorded.
12 | Thursday | September 26 | No new material. | Optional work on Test 2 in class.
  | Tuesday | October 1 | No new material. | Optional work on Test 2 in class.
13 | Thursday | October 3 | ISAAC | In person and recorded. TEST 2 Due at midnight.
14 | Tuesday | October 8 | No Class | Fall Break
15 | Thursday | October 10 | ISAAC | In person and recorded.
16 | Tuesday | October 15 | Long Reads I | In person and recorded.
17 | Thursday | October 17 | Long Reads II | In person and recorded. Project proposal due at midnight.
18 | Tuesday | October 22 | Long Reads III | In person and recorded.
19 | Thursday | October 24 | DMPs | In person and recorded.
20 | Tuesday | October 29 | No new material. | Optional work on Test 3 in class.
21 | Thursday | October 31 | No new material. | Optional work on Test 3 in class.
22 | Tuesday | November 5 | No Class | Election Day
23 | Thursday | November 7 | No new material. | Optional work on project in class. TEST 3 Due at midnight.
24 | Tuesday | November 12 | No new material. | Optional work on project in class.
25 | Thursday | November 14 | No new material. | Optional work on project in class.
  | Tuesday | November 19 | No new material. | Optional work on project in class.
26 | Thursday | November 21 | No new material. | Optional work on project in class.
27 | Tuesday | November 26 | FINAL PROJECT PRESENTATIONS | In person. Attendance required.
28 | Thursday | November 28 | No Class | Thanksgiving
29 | Tuesday | December 3 | FINAL PROJECT PRESENTATIONS | In person. Attendance required.

## Full Schedule

### Class 1. Tuesday August 20 - Command Line I
* In person and recorded.
* Slides - Bioinformatics - what is it and where did it come from? [old](https://docs.google.com/presentation/d/1sgBG19exSznuIj0HNufhdBB-A_SvysKIlbnXWsknYcg/edit#slide=id.p)
* Lab: 
  * [[Get on Sphinx]]
  * [[Getting started with PowerShell (Windows users)]]
  * Software Carpentry
    * [Intro to the shell](https://swcarpentry.github.io/shell-novice/01-intro.html)
    * [Navigating files and directories](https://swcarpentry.github.io/shell-novice/02-filedir.html)
    * Command to download data for command line lessons:

First, make sure you're in your home directory:

```/home/your_utk_username```

Next get the files:

```wget https://swcarpentry.github.io/shell-novice/data/shell-lesson-data.zip```

* Reading: 
  * [A brief history of bioinformatics](https://academic.oup.com/bib/article/20/6/1981/5066445) Gauthier et al 2019
  * [How Margaret Dayhoff Brought Modern Computing to Biology](https://www.smithsonianmag.com/science-nature/how-margaret-dayhoff-helped-bring-computing-scientific-research-180971904/) - McNeill, Smithsonian 2019
* Book Recommendation: [Bioinformatics Data Skills: Reproducible and Robust Research with Open Source Tools](https://www.amazon.com/Bioinformatics-Data-Skills-Reproducible-Research/dp/1449367372/ref=asc_df_1449367372/?tag=hyprod-20&linkCode=df0&hvadid=312130957577&hvpos=&hvnetw=g&hvrand=2428639128252987670&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9013452&hvtargid=pla-458238281452&psc=1) Buffalo 2019


### Class 2. Thursday August 22 - Command Line II
* In person and recorded.
* Lab:
  * [Slides: Computers and HPCs](https://docs.google.com/presentation/d/1wo8OOqeOwEouIJNXpWHPEqkHpKTVYn58eYxT9pS7xxI/edit?usp=sharing)
* Software Carpentry - [Working with files and directories](https://swcarpentry.github.io/shell-novice/03-create.html) 
* Reading, HPCs in Oak Ridge
  * [Frontier](https://www.olcf.ornl.gov/frontier/)

### Class 3. Tuesday August 27 - Command Line III
* In person and recorded.
* Lecture:
  * AgBioData Intro [Slides](https://docs.google.com/presentation/d/1lpFfg62PfzenpyyKFFH7ZG4QGg9cDhjQiMv7nuAylv8/edit?usp=sharing)
  * AgBioData Lesson 1 - Intro to Biological Databases and Repositories [Slides](https://docs.google.com/presentation/d/1k1uPpOOsCBghNjKzCEr98NaMPrVciMJzbdXveoIjyqs/edit?usp=sharing)
* [SWC 4](https://swcarpentry.github.io/shell-novice/04-pipefilter/index.html) - Pipes and Filters 
* Reading
  * [The 40 Most-Used Linux Commands You Should Know](https://kinsta.com/blog/linux-commands/)

### Class 4. Thursday August 29 - Command Line IV
* In person and recorded.
* Sequence Alignment: [Slide](https://docs.google.com/presentation/d/1NecHweqYZ9HMICOp_YQnfKBCq_wmDiScjMQbIF_uFxE/edit?usp=sharing) 
* [[Lab: BLAST and Diamond]] 
* Additional reading for learning more if you are interested:
  * Buchfink et al. 2001 [Sensitive protein alignments at tree-of-life scale using DIAMOND](https://www.nature.com/articles/s41592-021-01101-x)
  * Software Carpentry's [Regular Expressions for Biologists](https://carpentries-incubator.github.io/regex-novice-biology/index.html)
  * [Diamond software & documentation](https://github.com/bbuchfink/diamond)


### Class 5. Tuesday September 3 - Command Line V
* In person and recorded.
* Test 1 distributed. (TEST 1 Due at midnight on September 10th.)
* Slides: [Data Management](https://docs.google.com/presentation/d/1OqNMe2Pr_hDXOtaj1-0N3wKSEkjG0FQZQYe9Sdv9UQ0/edit?usp=sharing)
* [SWC 5](https://swcarpentry.github.io/shell-novice/05-loop/index.html) - Loops (Allyson)
* [Class & Lab Recording](https://utk.instructuremedia.com/embed/b690e564-b2dd-4a26-95d0-d6a00c49b5a5)

### Class 6. Thursday September 5 - No new material.
* Optional work on Test 1 in class. TEST 1 Due at midnight on September 10th.

