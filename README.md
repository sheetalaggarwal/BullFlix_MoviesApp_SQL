# BullFlix_MoviesApp_SQL
Capacity Planning:
Capacity planning is done in order to utilize physical resources like disk space, memory, CPU, storage space etc. effectively. We plan this by keeping business requirements in mind. For instance, what would be the growth rate of tables in future, how much extra space we should keep etc.
Below we are going to show the available and allocated spaces in our database.
Select * FROM DBA_TEMP_FREE_SPACE

 
From the above screen shot we can see how much free space we have. Now, we are going to see how much space is consumed by each table.
SELECT table_name, num_rows, avg_space, avg_row_len
from all_tables
where owner = 'RELMDB'
 

From the above table we can see that the space utilized by each table in relmdb database. Here, we are going to calculate the space requirement for fans and movies table.
We can see that average row length for fans is 254. At present we have 7655 fans. So, together all these fans will take 254 * 7655 = 1944370 bytes. Similarly, movies table will require 271 * 283 = 76,693 bytes.
On an average if we say that 500 movies are added every year and 1000 fans, based on this our database tables will grow very fast. For this we should look for range portioning or distributed database.



Data Generation:
To load data into our tables, we have generated data sets using Excel. Using this data, we were able to simulate what an application’s social media elements could produce. 
Message and message recipient tables
One of the most important tables that tied together communications among fans is the MESSAGE table. Core of MESSAGE table is MES_MESSAGE_BODY attribute that is to store texts of the messages sent among fans. We further intended those messages to be ready for text sentiment analysis (see p. Sentiment analysis to gauge users’ evaluation of movies) so it could not be a mere random combination of characters. To satisfy that condition we came up with the following requirements to the strings making MES_MESSAGE_BODY attribute:
1.	Messages must be compiled of English words.
2.	Total number of messages to be generated was arbitrarily chosen to be 5000.
3.	Messages must be of a variable length uniformly distributed in a range from 1 to 20 words.
4.	Some portion of all messages (arbitrarily chosen to be 30%) shall contain movies’ titles as part of the message body. Such messages containing titles shall be randomly distributed among all messages. Title to be mentioned in such messages shall be randomly chosen from a list contained in MOVIE table (283 movies). Position of the title (among all words making such message’s body) must be randomly chosen from 1 to message’s length.
To satisfy the abovementioned requirements, and specifically p. 1, a population of 20 words was created to be used in messages:
good 	cool 	not a fan 	bad 	worst 
disgusting 	hello 	what 	when 	I 
you 	like 	want 	grab 	food 
address 	car 	pick up 	director 	know 
Words were chosen to include typical vocabulary that fans discussing movies and their plans would use. Though 20 words is obviously a “poor man’s version” of what actual vocabulary would be, it was used as an example and can be extended.
MS Excel, including its random(), norm.inv() and other functions, was used to put together the strings for MESSAGE table. In the picture below a screenshot of our working spreadsheet is presented.

 

Legend for the screenshot above:
 
Message size is determined. It is randomly chosen from 1 to 20 using Excel’s randbetween() function.
 
Based on the size of the message (determined at step 1) a position of the movie title is determined randomly from 1 to message size using Excel’s randbetween() function.
 
Movie title is selected from the list of 283 movies contained in MOVIE table using Excel’s randbetween() and lookup functions.
 
Flag is set to TRUE (1) if a message shall contain a movie title, selected at step 3, and put to position selected at step 2 (about 30% of the time).
 
Set of words that can be used in constructing a message.
 
Based on size of the message, determined at step 1, words are randomly selected from the set (see step 5) using Excel’s randbetween() and lookup functions.
 
All words selected at step 6 are concatenated to form a string using Excel’s concat() function.
 
Several base inputs like maximum message size are outlined in that control form.

After creation of messages’ bodies, the rest of the attributes of MESSAGE table were generated. Screenshot bellow shows a part of generated table.
 

Legend for the screenshot above:

 
Message identifier, determined as a positive integer, keeping track of number of messages.
 
Message subject is decided to be a movie title in case such title is present as one of the words in a message body; null if no movie title is mentioned in a message.
 
Sender is selected randomly from a list of fan’s IDs of the FAN table.
 
First 20 messages were arbitrarily chosen as the only messages that could be “parent” messages to other messages to simulate possible “threads” of messages. It was further assumed that each time a sender (see step 3) who sent one of the first 20 messages shows up as a sender in other message, such message is considered to be a part of “thread” and a MES_PARENT_MESSAGE_ID is set accordingly.
 
Message body, generated at previous steps as shown above.
 
On a separate sheet distribution of messages by time was determined. Such distribution was set to obey 3 assumptions: a) distribution within a week: Mon - 7%, Tue - 11%, Wed – 10%, Thu - 29%, Fri - 21%, Sat - 18%, Sun ¬5%; b) weeks 15 through 30 shall have 2 times more messages than weeks 1 through 15 and 50% less messages than weeks 30+; c) actual number of messages for a specific date shall be randomly determined assuming normal distribution with mean equals to calculated based on previous 2 assumptions and standard deviation of 1 using Excel’s random() and norm.inv() functions.
 
Messages are set to never expire by setting expiration date to Jan 1, 2099.

MESSAGE_RECIPIENT table was generated the following way: 4900 messages out of 5000 were set aside to be peer-to-peer messages between friends. Random friends pair were selected to attributeв ещ those messages. 100 messages out of 5000 were supposed to illustrate group messaging. Groups IDs were selected manually to evenly distribute 100 messages across 5 existing groups (see p. Groups and Friends tables).

Groups and Friends Tables
GROUP_CHAT was manually populated with 5 groups (“Tarantino movies”, “Matrix trilogy”, “I'm gonna make him an offer he can't refuse”, “Paulie from Goodfellas” and “Not a fan of Scarface”) all created in 2016.
GROUP_CHAT_LIST was manually populated with 5 groups’ members. Each group got 5 members, that were comprised of fans with FAN_Fan_ID equal to 1 through 25.
FRIEND_REQUEST table was populated to showcase a typical activity of fans willing to establish connections among each other. First illustrated case: fans with ID = 3, 8 and 16 sent requests (RET_REQUEST_TYPE_TITLE is “Send friend request”) to randomly chosen fans (41, 43 and 94 fans respectively). Each of those requests is answered with acceptance (RET_REQUEST_TYPE_TITLE is “Accept friend request”). Second case illustrated is fan with ID = 22 sent friend requests to 76 other fans but is answered with rejection (RET_REQUEST_TYPE_TITLE is “Reject friend request”). Finally, fan with ID = 3 “unfriends” 2 of his or her friends. All requests take place during January 2016.


Followers, Movie Ratings and Likes Table
‘FOLLOWER’ table was generated using similar techniques employed for MESSAGE table. Below a screenshot of our working model is presented.
 

Legend for the screenshot above:
 
On step 1 a random list of fans (followees) was generated with a number of items arbitrarily set to 364 or about 5% of the total number of registered fans using Excel’s randbetween() function. 
 
Random number of followers in a range of 1 to 20 was assigned to each followee using Excel’s randbetween() function.
 
Random fans’ IDs were assigned to each followee accounting for total number of followers such followee has (see step 2).


After a matrix of follower-followee relations was built, it was dynamically transposed into 2 columns for FOL_FOLLOWER_FAN_ID and FOL_FOLLOWEE_FAN_ID attributes of the FOLLOWER table. Date of statuses was randomly chosen sometime in January of 2016.
MOVIE_RATING table was generated to illustrate functionality that allows followers to “like” ratings and descriptions made by their followees.
 

Legend for the screenshot above:

 
Rating identifier, determined as a positive integer, keeping track of number of ratings.
 
Random fan ID selected from the FAN table using Excel’s randbetween() function.
 
Random movie ID selected from the MOVIE table using Excel’s randbetween() function.
 
Random numerical rating normally distributed with mean equals to movie’s IMDB rating and standard deviation of 4 using Excel’s random() and norm.inv() functions. Further adjusted to remain within 0 to 10 range using Excel’s min() and max() functions.
 
Rating description randomly generated by the same algorithm employed for MES_MESSAGE_BODY attribute of the MESSAGE table. Word bank was somewhat changed and included the following: good, cool, not a fan, bad, worst, disgusting, best, never, script, action, recommend, fake, real, president, gang, guns, car, happy end, director, know.
 
Date was randomly selected within 2016.
 
No favorite flag was set.

After generating data, the next step was to create tables and load this data into those tables. For this step, we have given DDL queries below.
