# ICE (Intelligent Course Enrollment)
## _Bryn Mawr Course Scheduling Algorithm_

ICE is a course-scheduling software that uses a greedy algorithm to solve a version of the Registrar's Problem, where we are given/generate the input of students and a list of classes that they want to take, and want to optimize the number of students we can enroll in classes without conflicts. 

## Abstract
We explore the results and implications of our scheduling algorithm. In its implementation, we rely on a popularity based algorithm
that looks at the most desired classes first when matching professors, time
slots, and classes together. We then run this algorithm on real data from
the Bryn Mawr enrollment office and analyze both its performance and run
time in order to investigate what factors might contribute to minimal course
conflicts when considering the layout of a schedule.

## Introduction to Problem
Initially all the students, classes, rooms, and time slots are unpaired. Each
student has a preference list of four of the classes that they like. For each
class, the instructor is known, all must be scheduled in one room with a
specific time slot and a list of enrolled students.
A valid schedule would include a list of classes each assigned to a room, a
time slot, with a professor, and a list of students that are enrolled in it.
Other constraints include:
* Each time slot cannot be assigned more than |C|/|T| classes
* No professors are assigned to teach two different classes at the same time slot
* No student is enrolled in more than one classes that meet at the same time
* No two classes at the same time slot are assigned to the same room
* The number of students in each class must not exceed the classroom size

## Algorithm
We start by sorting the classes based on students’ interests. A class’s interest
score will increment by one if a student has it on his or her class preference
list. Subsequently, all of the classes will be sorted such that the class with
the highest interest score will be at the start of the list.
Next, we sort the available time slots such that the one with the least number
of overlaps with other time slots comes first. Subsequently, we sort all the
available rooms in decreasing order based on their capacity.
Now, we can begin to match classes to professors, time slots, and rooms.
To minimize conflict in the matching process, we iterate through the sorted
list of classes and assign each class a professor that can teach that class, an
available time slot with the least overlap, and the biggest room available at
that time.
We will repeat the above process of matching classes, time slots, professors,
and rooms until we have attempted to schedule every single class.
Finally, we terminate the scheduling process and begin to enroll students
in as many classes as they have expressed preference for while making sure
that those classes do not have overlapping time slots. This would allow us
to generate a valid schedule.

## Experimental Analysis on Generated Data
![Alt text](/data/ExperimentalData.png?raw=true "Experimental Data")
This table summarizes our findings when we ran our algorithm on generated data. We varied the number of rooms, classes, students, and observed how it affected our optimality and runtime.
Increasing the number of classes was the largest factor in improving our fit, which makes sense as we can schedule more students into more classes (providing that the preferences are randomly generated and there is not a huge skew in preferences for particular classes).

## Experimental Analysis on Real Data
A schedule is considered valid if it meets the class, room size, class time,teacher, and student preference  constraints. In order to calculate the fit, we  take  the sum of the number of total class slots filled, divided by the total number of preferences. Thus, we calculate the best fit score with the formula:

<img src="https://render.githubusercontent.com/render/math?math=Fit = \frac{Number of Preferences fulfilled}{Total Number of Preferences}">

![Alt text](/data/RealData.png?raw=true "Real Data")

After running our algorithm on all of Bryn Mawr’s unmodified registrar
data, with all constraints enabled besides pandemic protocol, we can see that our algorithm is able to achieve an average of fit
score of 73.23%, maximum fit score of 88.26%, and a minimum fit score
of 59.99%.


## Comparing our generated schedule to actual schedule from registrar (math classes only)
### Our schedule:
![Alt text](/data/ourschedule.png?raw=true "Our schedule")
### Registrar's schedule:
![Alt text](/data/realschedule.png?raw=true "Real schedule")
## Note about `isValid.pl`:
Our `test.py` script (can be found in `~/Algorithm`) has a function called `test` that contains the transcribed functions of `isValid.pl` and it evaluates the validity and performance of the schedule the exact same way. Our algorithm uses this python script to initiate the scheduling process and evaluate the output of the scheduling process. 


## Constraints
We decided on the following constraints:
### First year Writing Seminar: 
We know writing seminars are necessary to schedule correctly for every student who is a first year. Additionally, we know that a student cannot take 2 writing seminars
consecutively. Therefore, we can schedule them in similar time slots
in the smallest room available given their size (as seminars), at the
end of our main loop.
In order to implement this, we must reconsider how we sort the classes
by their interest. We must move all of the first year seminars to the
back of the list, such that we process them last as we know that they
cannot take another writing seminar (so it is okay to schedule them
in timeslots with more conflicts). Additionally, we must rethink how
we match them with classes. Rather than giving them the biggest
room available, we want to give them to smallest room available, as
logistically it is better to have a smaller classroom for a seminar to
facilitate discussion.
### First year language requirement: 
Because students generally take
only one language, we can schedule them last in timeslots with other
languages. We can handle first year language classes identical to how
we handle first year writing requirements. We schedule them last, and
give them the smallest possible room to facilitate discussion.
### Classes in appropriate classrooms:
Realistically speaking, it will not be possible for a Chemistry laboratory class to be held in an auditorium. Hence, we thought that it would be important for us to make
sure that our matching algorithm matches classes to the right set of
possible classrooms.
In order to implement this, we can adjust how we preprocess the classes
into the roomsF orClasses dictionary. Specifically, we would need
this dictionary to contain only the possible rooms that match a given
classes requirements. This allows us to check if a room is in the building we want a class to be in and that it meets any other requirements
the class needs. In order to determine what rooms are eligible for a
given class, we can parse all historical data and accumulate a list of all
rooms that are have been scheduled for a given subject, and use that
information to determine if we can schedule in a given room.
### Avoid conflicting high crossover major classes: 
We realized that
an important fact when considering how students choose their classes
in that their classes rarely if ever chosen independently. For example, if
a given student wants to take a class in biology, we would consider them
more likely to also take a class in chemistry. The same might apply
to Computer Science and Math, or Physics and Math. Generally, at
every level (100,200,300) there is one corresponding class in the other
department at that same level that we do not want to schedule at the
same time, so students can take both classes.
In order to implement this, we would need to define a list of ’corresponding’ classes for each class. By ’corresponding’, we mean that two
classes have a high number of students that need to take both for their
major requirements. As an example, this could include a Physics class
and its corresponding lab. This could also include discrete math and
other Computer Science classes. Another example could be 200-level
Biology with 200-level Chemistry, since these classes are often taken
together by majors in either department. Once we have this definition,
we can add an additional check before scheduling a class that it does
not conflict with any of its corresponding classes.
### Schedule additional classes for professors on the day/s when they already teach:
We consider the fact that professors would
prefer to come in to teach not every day if possible. So when a professor
is already assigned a class, when we schedule another class for that
professor, in line with the greedy algorithm we check the timeslots
that have the same number of minimum conflict, and schedule the
non-overlapping timeslot which has the same days as the other class
(if it is a valid timeslot to schedule).
### Pandemic Protocol: In the case of a pandemic, we want to reduce
the number of students in each classroom to facilitate social distancing.
We decided that dividing the classroom sizes by 3 would help lessen
the spread of a pandemic between students and professors.


In addition to the constraints above, we also included all extensions present
in the real data. This includes:
* Overlapping time slots.
* Time slots that are different times on different days.
* Professors are eligible to teach an unlimited number of classes but will
only be matched with at most 2.
* Students can enroll in more or less than 4 classes.
With these constraints, we are able to run it on the real data unedited, with
none of the real information missing.

## Recommendations
Based on the results and output of our algorithm, we can analyze the factors
that cause our algorithm to develop a higher percentage of fit and see how
those factors could be optimized in a real life setting to allow for less overall
time conflicts.
Based on the analysis of the math classes scheduled by our algorithm and
how they were actually scheduled, we can suggest the following:
### Increase the number of classes scheduled in once a week, longer time slots: 
This can be seem by comparing the generated and
real schedule and noticing how heavily the algorithm prefers scheduling
classes in these long slots. Because our algorithm works by picking the
time slots with the least overlap first, this implies that these long slots
actually minimize conflict when attempting to schedule the classes.
### Schedule 100 and 300 level classes together in each department: 
This is an idea that is also present in the sample output schedule where it has many of the 300 levels overlapping with some of the
introductory courses in a given department. This idea also makes
sense intuitively since the chances of a student enrolling in both a 100
and 300 level in a given department is highly unlikely. Additionally,
scheduling them at the same time minimizes their potential total conflict with other courses, allowing more students to take them, and the
other courses they are interested in.

### Schedule each class in a given department around the same time of day: 
We noticed in the sample schedule that the math courses
were entirely scheduled in the afternoon of Monday through Thursday.
While some students will take 3 math courses in a given semester, or
3 courses in any given department to finish their major, they are in
the minority. Therefore, to increase total enrollment, it makes sense
to condense these classes together such that they only take up a small
section of the day allowing for flexibility elsewhere. In the example of
the math schedule, we have that scheduling the courses in the afternoon will give students the entire morning to take non-math courses
without them having any conflicts with math.

## To run it on all of Bryn Mawr's data

1. Navigate to `~/Algorithm` and run the following command

```sh
python3 test.py -f ../data/parsed_data/ -s ../data/schedule_real.txt -a
```

To enable or disable constraints for the scheduling process, navigate to `~/Algorithm/main.py` and change the default keyword argument values for the constraint of interest. This would toggle the constraint and allow the algorithm to run with or without the constraint.

## To run it on random generated data

1. Checkout the `without-constraints` branch from the Github Repository
2. Navigate to `~/Algorithm/misc` and run `bash generateRandomInstances2.sh [instances] [rooms] [classes] [times] [students]` to generate folder with test cases
2. Move the entire folder with test cases into `~/Algorithm/misc/test_cases`
2. Navigate to `~/Algorithm` and run the following command

```sh
python3 test.py -a
```


