# 31 Oct 2018
I like how visual studio looks. So I am attempting to do my logs over here instead of notebook.
Today I will like to focus on finding out why the OTSubControl table is not exported out in RSSTE. I wish to finish this in 7 Tomato. So I have to be really focus. 

Possible Cause
- RSSTE is not reading Hebrew script properly
- GDef in XML is wrong

Strategy
- Should I check how GS4 is handling OT Sub control? 
- Maybe I should see the sequence. 
- Maybe do a documentation on it


Generating OTSubControl in GS4
This is what is in GBT_tstOTScript

struct GBT_tstOTScript
{
    GBT__tstChainSubCache   *pstSubstitution;
    GBT__tstMarkBaseCache   *pstMarkToBaseTable;
    GBT__tstMarkBaseCache   *pstMarkToMarkTable;
    GBT__tstOTArab          *pstArabicShaping;
}

struct GBT_tstOTSubControl
{
    GBT_tstOTScript         *apstScript[GBT__nNumberOfScriptsToProcess];
    uint16                  *pu16ClassTable;
    GBT__tstKerningCache    *pstKerning;
}
Call stack when allocating memory for 
FNTEG_pstFTCreateFont->FNTEG_boLoadLayoutTables->FNTEG_boAllocateClassCache->FNTEG_vLoadMarkClass

[0] = {u32OTScript=1818326126 u32OTLanguage=1145457748 u16ScriptCache=0 }
[1] = {u32OTScript=1634885986 u32OTLanguage=1145457748 u16ScriptCache=1 }
[2] = {u32OTScript=1952997737 u32OTLanguage=1145457748 u16ScriptCache=2 }
[3] = {u32OTScript=1751474802 u32OTLanguage=1145457748 u16ScriptCache=3 }


code crash at font_resource_maps.clear().....
or is it at dat_index_maps.clear()

okay let's see
okay I have verified that call clear on an empty map should not crash the system. 


okay I have some clue
it_fonts.pstOTSubControl is allocated memory in thai

After elimintaing many possiblity, the issue lies in the script table is not being fill up. 
Why? I dont know. 

Code wrong or input wrong

u32Language = 1145457748


u32Script = 1818326126
u32Version	256	unsigned long
u16ScriptList	2560	unsigned short
u16FeatureList	21504	unsigned short
u16LookupList	36864	unsigned short


u32Language = 1145457748
u32Script = 1751474802


u32Version	256	unsigned long
u16ScriptList	2560	unsigned short
u16FeatureList	9216	unsigned short
u16LookupList	12800	unsigned short

at this point I am so tired
Question will be is it possible to just use Gdef without GSub?
So the font that Toyota gave us does not have GSub


Things to do
* check if Hacked toyota font works
* check if anything can be done to by pass GSUB
* Read up about dynamic Memory
* SOS
* Debug DN8? 
* Split Font files/GLTF Files into different group 

# 1 Nov 2018
Can't believe it is already the start of Nov

Anyhow got to be focus. 

What is the goal of today? 
I think it will be to split rsst. into different group. 
and test it. especially with the special fonts. 

What is excatly inside GPOS, GSub and GDef? 
This will be my target today. 
Maybe will spend around 1-2 Tomato. 


Okay, unfortunetely the generated toyota XML does not works. Sad. It went into an infinite loop. 
I have to investigate it. 

Static Hebrew Text from Google is in a infite loop
Hypothesis
> We should not add Latin in the script. reason is because latin is being check but,the gsub tables does not have hebrew and for some reasons It went into a endless loop. 
> verify and it is correct. But not sure why this happens. Have to debug deeper to find out why.
> Check what is really happening. 
> I think we should be able to replicate the same issue in FT font


## Things to do
* Replace current data (done)
* Add it in to msvc (done)
* Test it (done)
* Commit changes (done)
* Move task from gltf to other test story (done)

# 2 Nov 2018
## Things to do
* Check if Toyota is being emailed (Done)
* check difference in code for Toyota GS (Not Needed)

* build protobuf on Target (Not Needed)
* Document settings for Target changes by Heiko (Bring Forward)
* come out with plan for Retrospective (Done)
* Read Up about davinci for RSDL (Bring Forward)
* follow up with Natasha about Memory increased for RSDL (Bring forward)

# 7 Nov 2018
Retrospective points
* hand over for Vikas
* Wiki page structure

## To do list
* reply HMI team - (Done)
* Check Toyota Font - (Done)
* Clone RLE Analyzer tool - (Done)
* Read up RSDL (Scrap this, do it next time maybe)
* check why is Hebrew font is in infite loop if latin is configured (Bring forward) (created a task for it)
* Document settings for Target changes by Heiko (Bring Forward)
* follow up with Natasha about Memory increased for RSDL (Bring forward)
* Read Up about davinci for RSDL (Bring Forward)
* Davinci Transformation

* Read up RSDL - 2
- Check if gltf branch is already being merge with main branch. (merge already) 
- Intergrate RSDL into GLTF branch. 
- see if there is any issues with just running rsdl

Log
Seems like RSDL is able to compile properly in GLTF branch. 
I think I will need to find out how to read it properly. 



# 8 Nov 2018
## To Do List
* Generate GSRen TestReport and code coverage 2 (DONE but have to resend)
* Take a look at the font issue (Send an email for now)
* Prepare for SOS 2 (DONE)
* Release RLEANAL? (Move Forward)
* RSDL Update (ShuHui Working on it)

# 9 Nov 2019
To Do List Carry Forward
* Release RLEANAL? (Move Forward)
* RSDL Update (ShuHui Working on it)
* Check FPKM stuff (Done)

## To Do List
* Support Diacritical marks issue
    - run rsste (done)
    - draw strings (done)
    - missing script option is rsste xml
    - add it in and rerun
    - okay gpos is apply which is good
    - but diacritical mark is indeed wrong
    - try to freetype and see
    - reduce string to only issue part
    - check excatly what is being called for the positioning
    - see why is it the rsste is not being called
    - okay even ttf is not working. have to check and see why.
    - must be some tables there is shaping it and it is not working. why why why. 

    - unicode involve
    - 0x644, gid- 135, name=uni0644
    - 0x627, gid- 6137, name=uni0627
    - combine to gid-138 0xfedf uniFEDF
    - 0x64B, gid-302, name=uni064B



Log
I always, always.... have this problem that I cannot find a testcase to test my fonts rendering. Why why why!

The MarkLigPosFormat1 subtable begins with 
a format identifier
two offsets (markCoverage Offset, baseCoverageOffset) to coverage tables that list all the mark glyphs (MarkCoverage) and Ligature Glyphs (ligatureCoverage)
referenced in the subtable.

What on earth is coverage?


# 9 Nov 2019
* Release RLEANAL? 
* RSDL Update
* Support FPKM HarfBuzz
    - check if harfbuzz code are correct?
* Diacritical marks (Done)
    - clean up M2L
    - check logic
        - when to use component 0/1 and class 0/1
        - read it in GTRL
    - Check RSSTE
        -modify RSSTE Data
        - read in RSSTE Data
        - check GTRL

    - verify working version


To do List
- Check how component works (Done, still not sure which component to choose)
    - what to use and differences between base and lig
- Reply Ulrich (Done)
- check RSSTE part (Not Done)
- See what should we proceed to do?

Mark To Ligature 
with markToBase attachment, described previously, each base glyph has an attachment point defined for each class of marks. 
markToLigature attachment is similar, except that each ligature glyoh is defined to have multiple components.
Each component has a seperate set of attachment points defined for the different mark classes. 

As a result, a ligature glyph may have multiple base attachment points for one class of marks. 

For a given mark assigned to a particular class, the appropriate base attachment point is determined by which ligature component the mark is associated with. 

This is dependent on the original character sting and subsequent characteror glyph sequence processing, not  the font alone. 
While a text-layout client is performing any character-based preprocessing or glyph substition operations using the GSUB table, the text layout client must keep track of association of marks to particular ligature glyph components. 

Example:
Assume 2 mark classes: all marks positioned above base glyphs(Calss 0), and all marks positioned below base glyphs(class 1)

# 13 Nov 2018
## To Do List
 - Check why on earth font cannot be drop in harfbuzz (Check)
 - Check the re-setting of config (check)
 - Release RLE? (In process)

Log 
hb_font_t* font that is being in to hb_ft_font_changed
the font->user_data size is null

hb_ft_font_t *ft_font = (hb_ft_font_t *) font->user_data;
FT_Face ft_face = ft_font->ft_face;
ft_face->size->metrics.x_scale 


What the shit!
Why is my FNTEG__stFTFontAdmin.astFTFont[i] screw up?

# 14 Nov 2018
## To Do List
- Check static fonts issue (Done)
- Release RLE (Done)
- Write a report on finding in FPKM
- Clean up code (For what? Kinda forget for what)
- Keep it properly code for M2L 
- try hashtag for articles and see if I can put them together
- Read on what is expected for SOS (Done)
- Clean up RSSTE and push code (Push but not clean up I guess. Let's clean up today.)

Log
What should I focus on? 
This is the question to ask. 
Let's focus on the endless loop issue. Maybe take 2 tomato.
What should I do with my future?

Okay code stops at GTRL__vDoPositioning, what does DoPositioning does in the first case? 

How/what is the PreferedLanguage affect the code? 

# 15 Nov 2018
## To Do List
- Clean up RSSTE and push code (Done)
    - Read Code
    - Build RSSTE
    - Push it to GS4TF/ JGP? 

- try hashtag for articles and see if I can put them together (Doing)
- Write/Email a report on finding in FPKM (Done)
- Read UP RSDL 
- Keep it properly code for M2L 

Log 
I will attempt to do 10 Tomato today.
Succeed

# 16 Nov 2018
## To Do List
- try to see what can I hashtag
- keep code properly for M2L?


How do you focus? 
I just read an article by Mayo Oshin on 
Zanshin: How to master the art of focus and concentration 
from a legendary Japanese archer

It says the way to the goal is not to be measured!
Of what importance are weeks, months, years?

Instead of focusing on the goal. 
Focus on the task at hand. 
The relax focus. 
The breathing. 

Focus on quatitative improvement.
Find out about small changes that make big differences.
This is the so call secret to life. 
Everyone successful people has them. 
It might works for everyone and it might not work. 
but I have to find out and test this changes. 
Remember it's small but quality changes. 


# 19 Nov 2018
PI 18.3.4
Review Davinci
Review RSDL -> create Testframe in GS4 TF? 
Training
- ROMG/ROCH
- Context
- Layers/Surface
- Activities, create a sandbox to play
- Colourlization 
- GRLC Functions

# 23 Nov 2018
I need to some how seperate my localtoparent matrix from update. 
Seperate and multiply outside. 


# 25 Nov 2018
To Do List
- Fix GLTF Jenkins
- Continue on Transformation

- Document FNTEG Features

Daily Log
What should I be focusing for the week? 
This is the last week of the PI and I have to get things to be done. 
The transformation is a pain in the ass because of the fact that I have to do calculation and I am pretty bad with it.
But you just got to do what you got to do. 
Also I have to plan for the traveling from Narita to Downtown Japan. 
A pain in the ass again, again I got to do what I got to do. 

The transformation is getting shit difficult. 
I am not even sure at this point should I even be pursing over it. 
Can it be done? 
I am not entirely sure. 
If it cannot be done, I have to discuss on why it cannot be done. 
.....

# 26 Nov 2018
To Do
- Transformation
    - come out with a sample case too
    - try the inverse method, even if it does not works properly at the start.
- Fix Jenkins (Done? )
- Document FNTEG Features

Daily Logs

2T
- just found out that making a parent node is not really that straight forward. 
- why? Just why! zzz! There are so many things on my plate right now I need to simpfly


Personal Logs
Will start to log things over here and add it to google keep at the end of the day? 
okay interesting that the company has unblock google docs. hmmm

Things that I need to do
- Change money. about $1.5k? Will that be enough? I certaining hope so. 
- read up how to withdraw money from overseas ATM
- book transport from Narita to Tokyo and via versa
- book disney sea tickets
- check how to go mount fuji

# 28 Nov 2018
To Do List
- transformation matrix again!!!


Am I an effective programmer?
- Does my code make people feel good? 
- Does it automate some user task?
- Does it make some user task easier?
- Does it amplify users' power?
- Does it create value?
- 10x dev that can help others be 10x more effective
- write code that is easy to understand/use/maintain
- does your code has a good bus factor? If the maintainer dies tmr, can anyone pick up where he/she left off? 

Daily Log
What is my mantra? 
I want to write code that is beautiful. When people see it, they feel a sense of beauty. 
Just like design. There is intention and mindfulness applied to it when created. 
In everything that I do, I will try to do it with intention. 
Not possible all the time but I try. 

I am not determine by my tools but my mind. 
I am not sure how to improve my mind, but I will do all things to improve the level of thinking/communication. 

okay... shouldn't have go for lunch. But that is life right?

# 29 Nov 2018
To Do List
- Clean up code for world transformation
- check again code for multiple child
- mail Jen about the changes?

Daily Log
To be a good programmer it is to be mindful of everything that is being written.
It comes with intention and a mindful though of every single line that is being written.
It might be slow but it comes with thoughts. 

Yesterday was a bad day in terms of tomato. 
I think I need to realign my focus. 
And go back to the plan. Follow it as closely as I can.
Forget about the result. For the result is not in my control. 
What I can do is the process. 

I have to keep a collection of my skill set

Things to improve on.
- writting
- speech
- practice thinking
- dont be lazy 


# 30 Nov 2018
To Do List
- RH (Done)
- Finish Transformation
- Check list for previous issues brought up
- reply Jan


Daily Log
- So super duper sian. 

# 10 Dec 2018
Can't believe it's already dec. 
- crazy ideas for work
- demo for all the GS APIs
- Check PI Capacity


WK - 3.5
HP - 7.5
Gin - 5
Nishin - 3 
Del - 8.5

Alisa - 0
Jefferson - 0 
Hosea - 0
Sarun - 0
Bach - 2
ShuHui - 7.5
David - 8.5
Neha - 8.5

max - 8.5

how to draw component for mark to ligature. 

To do list
- add objective 
- add risk
- add out of scope

# 12 Dec 2018
To Do List
- Claim air ticket
- Upload excel to network
- clear unwanted emails
- set up email directions

philosophy of programming
- Easy to understand code
- Code with clear intention
- manage complexity, able to manage between current code and future code
- as optimize as possible with the effort given
- complexity in terms of run time
- and complexity in terms of cognitive, syntax, boilerplate, indirection
- we humans are program in a way to always look for the easiest way to do something, if it gets too hard
    we tune out. It is like answering 1+1, we can immediately answer it but what if it is 49 * 49, we can for sure still answer it but it start to justific if it is worth the extra energy. 
    Good programs should not fall into this stage. But it takes experience and energy and time to manage this. 
    one way to manage this is to break things into smaller parts, so 49*49 can be broken into 7 * 7 * 7 * 7.
    We just have to be more intentional with the things that we are doing, instead of letting things run to its course. 

Daily Log
- the only way to move up is to be able to communicate effectively both verbally and in writing. 
    It is the skill that can be improve over a period of years and not worried that things will be obsolete. 
    That is why I need to improve on my writing skill and verbal communication. 
    My weakness is that I have no idea how grammar works. And my spelling sucks. 
    To add to insult, my pronuciation is bad. 
    So issues
    1. Lack of a strong grasp in grammar
    2. weak spelling skills
    3. Poor pronuciation of words

    So how do I fix this? 
    - I can do a little a day, improve a little a day, my target will be 5 years maybe? 
    - To be a world class communicator.


# 14 Dec 2018
To do list 
- read finish mental model
- read finish javascript

# 17 Dec 2018
To do list
- transformation thingy
- component -> see how it works

Daily Log
- SH
- Gin
- HP
- DY
- Kyle
- HR
- Nishin
- Bach
- Neha
- Srini
- Steven
- Alisa
- me


# 18 Dec 2018
Daily Log
- TBH I am so bored with work.
- what can I do as a a software engineer?
- I think writing skills are important. 


- How is material being used? 
- how is uniform being used? 

# 19 Dec 2018
To Do List
- Submit Lee Tian Resume (Done but company screw up)
- Look at what uniform location does (done?)
- Review Davinci code 
- Finish transformation

just focus on 8 Tomato a day
block off all other distraction

Daily Log
I am kinda bored with work. What is the purpose of my work? 
Felt kinda unmotivated. No motivation at all. 
Why is that so? Also I need to practice my guitar. Zzzz
I think to go with my timeboxing process really works well. 
So I think I should really do it. 
And writing down my thoughts work really well too. 
It keep me motivated for some reason.

1:53pm 
- Start first tomato of the day. 
- maybe I will end work at 6pm? Let's see.
- first sprint feels good. a little long but good. 
- managed to read up alittle on uniform and vertex attribute. 
- ran gltf branch. 
- next to do is to actually run the model given my jens
- then follow by my model. 
- not too difficult right?
- physically feeling alittle hungry... zzz

2:55pm
- model didnt get drawn.. need to investigate why
- shuhui fixes it so I will do a beyond compare and see what is the differences
- read some pbr and webgl extension
- what do I want to achieve? 
- finish transformation and understand how webgl works

# 21 Dec 2018
To Do List
- Finish World Scale
- Check Matrix calculation

Daily Logs
Okay Let me just chop chop finish the world scaling first. 
Using the tomato rule. 
8 is good enough.
First one is always the hardest.


# 24 Dec 2018
Daily Log
- think deeply about simple problems

What is a static variable in a class?
What is a static function in a class?

# 26 Dec 2018
Daily Log
- How do you study programming?
- Or rather how to be a wizard?


Career Learning Log
What do I need to learn?
How do I learn it? 
How do I apply it?
How do I monitor it?
How do all this knowledge comes together? 
What is my utimate aim in doing it?
I think having a website like learncpp or learnopengl helps alot.

How to be an expert? 
An expert has a very strong interconnection of different mental models. 
He is able to link ideas from different places togther. 

How to be an expert?
Do deliberate practice
1. Try to recall stuff that u have just learn. We have issues not with remembering but recalling. 
    We have to practice recalling on this
    Immediate recall tech
2. Explain. Often we thought we know but most of the time we don't. We can only find this out when we are trying to teach
3. Build Chunks

CPP
C++ Basic
types
variables
functions

Class
Stack Vs Heap
STL

# 27 Dec 2018
To Do List
- Finish up the scaling API
- Verify if it is correct
- Submit MC

Daily Log
A 2018 year of reflection
How is 2018 going for me? 
A good aspect is that I managed to reduce to student loan to under $25K but I must admit that I kinda over spent.
For work wise it is good that I started working on 3D. But it is not to the effect that I wanted. 
I think I should really document or start blogging so that I can look back with acccomplishment. 

Pomodoro Log
I am not able to focus, I must admit. 
I hope that with the help of pomodoro, I can improve my focus.
Treat work as a form of medidation. 
Single task work. 25 mins at a time.

1st Tomato
What did I do?
- Attempted to do a rotation using glm::rotation

What should I focus next? 
1. Check if my glm is in radian or degrees (in radian)
2. check how concatenation of scale and rotate affect the inverse matrix. 
3. check what about multiple concatenation. 

2nd Tomato
What did I do?
- Managed to verify that it is indeed radian 
- Check Rotation in GLM using Quat and Rotation but somehow it is incorrect. 

What should I do next?
- check with real PI/2
- check with PI. 


To DO 
- even the rotation is kinda screw up!!!!! Why!!!!!


# 28 Dec 2018
First thing first 
- 8 tomato

To DO 
- even the rotation is kinda screw up!!!!! Why!!!!!

Daily Log 
3T
- Matrix is still as screw up as possible. 
- Goal is still being acurate

# 31 Dec 2018
Daily Log 
OMG... It is the last day of the year.... 
I still have this shitting problem to solve.. or else it will be actually an home run. I guess you can't have everything in life. 
Maybe just write a reflection on 2018. How was it? And what lessons can I actually bring with it. 
I think one thing that I managed to breakthrough is in terms of writing and public speaking. 
Both from the same event, helping to promote the company which I was reluctant initially. 
But I guess, I am glad I took up the pludge. 

Work has been a rollar coaster. Ribhu left the company so, there are mant slack to pick it up from. 
I think I am actually declining in terms of my programming skills. 
But I guess I made it up in terms of thinking. 
I need to be more focus. And I think one way to focus is to actually document down what did I do. 
I think I will do that more often next year. Document every single hour of my work life. 
This kinda spur me to do more? 

SO what is my goal today? 
Let us explore my programming philosophy again. 
It is not good just to be doing things the way that it is. 
Or to do things to to see it end. 
There needs to be certain quality about it. 
Accurate is the most important factor. 
Results has to be as accurate as I possibility can. 
The rest like space and time complexcity are secondary to accurately. 
If Accuratecy is being achieve then we can consider the rest. 

Write easy to read code is important too after which. 
Code that are easy going for the congnotive. 
We spend more time reading code that actually coding. 
Then lastly will be easy to maintain. 
We do not want to be breaking things whenever we need to change or add a feature. 

So to sum it up
1. Accurate
2. Time Complexity 
3. Space COmplexity 
4. Easy Going for the conginitive
5. Easy to maintain 

To Do List
- Come out with a solution for the transformation matrix

Matrix log issue.... 
zzz... really zzz.. But I have to focus... 
Let's do with 4 tomato first and see how things goes. 

Lets see what do we have first... 
End result
Given a worldtranslate/ rotate/ scale
I will have a node that does the above transformation without the influence of the parent/grandparent transformation.
Okay this is clear. 

What have I tried? 
1. transform the position from world to local. 
    and store it as the local position. 
    - Let me verify this. 

# 2 Jan 2019
To Do List
- Change Relative to Absolute Functions

Daily Log
1T

# 3 Jan 2019
To Do List
- SOS (DONE)
- Check what SFML is doing for Translation and Scale (DONE)
- Remove transformation Functions in Node (DONE)
- Add Dirty Flag in Transformation (DONE)
- Change Demo Transformation functions (DONE)
- Change TF Functions (Done)
- Apply MW (DONE)
- Scan MC (DONE)

# 4 Jan 2019
To Do List
- Understand RSDL Dependency 
- Read through entire logic

Study System
- Read through the content page
- Read through the content one time quickly
- It is okay that you understand everything. As you often do. 
- Then go back to content again. 
- This time read it line by line slowly. Ask yourself if you understand the sentences 100% percent. 
- Do not cheat yourself. If you don't, ask yourself why, what information is missing. Trace back to the missing information. And repeat again, until you fully understand. 
- Next try to explain to yourself the concept like how you would explain to a 5 year old kid. No terminlogy. 
- explain as simply as you can. If you can't go back to what u find it hard to explain simply and disect it to simpler terms. 
- Repeat the process. 
- Write a blog about the topic then. 

## Daily Log
---
https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern
Curiously recurring template pattern
A class X derives froma class template instantiation using X itself as template argument. 

General form

template<class T>
class Base
{
    //methods within base can use template to access members of Derived 
};

class Derived: public Base<Derived>
{
    //
};

RSDL
- Why on earth is font not being called? 
- What is RSMG_pstGetPipeMFontByID? 

To Do list
- Remove RSDL Surface? 
- Check font
- standardize naming of .proto 



2 Week of 2019
Alight... First week was okay... I did pretty alright in pomodoro. 
i hit at least 8T everyday, except for Friday. Okay my bad. 
But even though I was not focusing on the productivity of my work I actually got more productive. 
So can I safely say that focus work is equal to productivity? 
One more thing I notice that, I am done with my task on Thursday. So Friday, I was feeling alittle less motivated and to some extend did not really know what should I focus my work on. 
I think having sepcific and just a little challenging task is important for productivity. 
Uncertaintly, easy or over challenging stuff overwhelms me. 

So the next time, when I am working on something, I need to be aware. 

1. Task has to be specific. 
2. Task has to be just right skillswise. 
    If it is too easy, just quickly do it. 
    too difficult, see if I can break it down into easier things. 

# 7 Jan 2019
To Do list
1. Go through the source code for RSDL... 
2. precept by precept make sure I know every single line. 

Daily Log
I am looking into RSDL right now. 
So the question is what do I wish to achieve by the end of this? 
big statement, I want to understand every single line. 
What it does and why was it written in such a manner.
this is funny consider that I am the one that wrote it.
But I have touch it for some time. 
So this will be a good touch. 
Also, I should not be afraid of changing code if neccessary. 

What is the differences between GBT_tstMSrfcDesc and GBT_tstMSurface? 

I seriously need to consider reducing the number of containers. 
Place that need to seriously consider reducing
- Dependency


what is the differences between pair and map?
- you need to use pair if you use insert for map

8 Jan 2019
Yesterday wasn't excatly that productive but yesterday is gone. Let's focus on today. 
Today I only have two goals. 
1. Finish 8T
2. Understand and clean up dependency


1T
- Update RSDL code from git and rerun tool chain
- Wasted some time on this
- but focus on the work on hand rather than the goal. 
- work as a medidiation

Daily Log
Starting a Tomato is always the hardest but once u get started, everything will fall into place. 
So I have to trust the process. 
Treat every tomato as an mediation session. 

At about 120pm I am starting to get a little hungry. 
Just a little not to the extend of want to go out to eat whatever. 

2T
- I have try to set a bitmap without default dependency.
- This is bad. It crashes the program. But at least We set an Exception, so this is ok.
- Message seems to be clear to me too. So this is okay too. 


# 9 Jan 2019
To do list
- add ryan and jeff to Git (Done)
- request for access for Sarun (Done)
- read through every single line for RSDL dependency (Partly Done)
- check tiberu mail for dependency (DONE)
- check GS4TF for RSDL (Done)

It's 1:34 pm and shit nothing has been done yet.. 
GG.. Let's focus. 1st T is always the hardest but let's do it/ 

Daily Log
- Treat work as a form of medidation
- be clear what u need to do.
- write down everything that is needed to be done. 
- Prioritise as much as you can. 


# 10 Jan 2019
To Do List
- Code Review for Harfbuzz (7T)
- SOS (1T)

Let's focus on 2T now for Harfbuzz Review
and 1 more for SOS.  


- FNTEG First


# 11 Jan 2019
To Do List

# 14 Jan 2019
To Do List
- have a end of the day to do list(Done)
- Finish GTHB Review (Done)

# 15 Jan 2019
TDL
- send Delvin GTHB Code Comments (done)
- check the differences between IIP Gauxl and JGP (done)
- check with Anandh to see if there are code that needs to be remove for BMW (Done)
- check with QA about tracebilty  (done)
- check how to create ORM (done)
- Look through WIKI and if there is none, document it (add it in but more can be done)

Daily Log
1T
- clean up GTHB Code (Done)
- go through comments again (Done)
- Send to Delvin (Done)

2T
3T
4T
- I am kinda stuck. 
- need to wait for Anandh to get back to me. Should I push for it?
- What should I do next? 
- check how to do review management? 
- update release site? 



A plan to release GAUXL
1. check which code to release
2. Check if any code differences for IIP
3. Check if we need a new variant in MKS
4. Check in Code
5. If new variant, need to update MTS, MRS, MS, MTF? 
6. Get metrics, QAC, CTC, DC, PM, KW
7. Check in Docs
8. Create ORM
9. Docs Review
10. Close all ORM
11. Send Release Email


Note
- RSST Linker check by Nishin. 

# 16 Jan 2019
To Do list
1. Check in GAUXL TO CDS_GRA JGP (Done)
2. CHECK GAUXL TF RESULTS (Done)
3. PR IIP_EXT (Done)
4. CHeck in code to MKS (Done) 
5. Update Docs 
6. Review Managment
7. Ask Ryan to run Gauxl (done)
8. Update Wiki on release
9. TF for 64 bits? 

# 17 Jan 2019
To Do List
- SOS Page (Done)
- Facilitate Code Review (Done)
- check how to do review management
- Check TF Results (Done)
- Check CTC (Done)
- Check in CTC (Done)
- Check if need to do MRS (Done, No need)
- Update MTR (Done, No need)
- Fill up checklist (Done)
- Fill up ORM when reviewers are done. 
- Close Review Management
- Update H1
- Run FRNFT TF

# 18 Jan 2019
To Do List
* Gauxl
- Close Review Management
- Update MTR (Done)
- Update TF (Done)
- Fill Up ORM
- Update H1


* FNRFT
- Figure out what is the Testframe doing

* GTHB
- Create Review Management?

# 21 Jan 2019
To Do List
- Review MRS and MTS
- Check FNRFT TF


Log
- from FNRFT1ce.h to fnrft_ce.h

hmmmm
tlsf_c1.h was added previously
- now we have added an additional "fnrft_ci.h"


# 22 Jan 2019
To Do List
- check FNRFT TF
- EMAIL TingYi
- check story status



# 23 Jan 2019
To Do List
- 

Daily Log
- Today I am going to go slow to evaluate my work
- why move so fast when u don't know where u are moving to?

# 29 Jan 2019
## To Do List
- Update RSDL Wiki
- Check GRLC Code
- Iterate with Nishin on GRLC .fix release
- sync gauxl to JCP
- check FNRFT SVN


Daily Log
-

Release GRLC
- get Klockwork
- get product metrics
- get CTC
- get DC
- get QACPP
- Create Code Review
- Close Code Review
- Update H1
- test it on hw


# 30 Jan 2019
## To Do List
- Create a branch in JGP
- Run JCP but this will be depending on if I get admin rights
- Take a look at RSDL wiki and see if it needs any update

Daily Log
- software/hardware complexity is high. 
- As an engineer I want to treat this as a challange and not this is not my domain.
- I wont touch it. Take itas a problem solving kinda challange. 

# 31 Jan 2019
## To Do List
- try to run JCP again
- compile in JGP

Daily Log
- Not sure what to work on, work on skills that are universal
    - Self Discipline
    - Personal Effectiveness
    - Communication
    - Negotiation
    - Persuasion
    - Physical Strength and stamina
    - Flexibility

    “The most important thing to keep in mind is to do the smallest, least expensive experiment you can.”


    - go slow and think
    - look at small things
    - it is the small things that build the big things
    

# 1 Feb 2019
## To Do List
- Create a branch in JGP
- RUN GRLC in JGP
- Check the metrics
- Check JCP Jenkins

Daily Log
It's already the first day of Feb. 
One month has gone, and we are left with another 11 months. 
I think I need to slow down and think. 
What is important and what is not.
Somehow I had this dream last night. 
It is better to go slow in the right direction than fast in the wrong direction. 
Many times when we are going to fast, we failed to stop and think, am I in the right direction. 
Of course, if you are in the right direction, good for you. But what if you are wrong? 
It might take a longer time to turn back than if you are heading in the right direction but at a slower pace in the first pace. Be more delibrate in your intend and stop going with the flow.  

# 7 Feb 2019
## To Do List
- PR TLSF
- Update GRLC TF with code
- Check with Nishin on GRLC code for release


# 11 Feb 2019
## To Do List

# 12 Feb 2019
## To do list
- check Jan transformation issue (DONE)


Problem Solving issues
- Check previous issues
- See if anything has been done about it
- Discuss with kenneth
- Check with Scrum Master about any issues they are facing

# 13 Feb 2019
## To Do List
- Check what is needed for FNRFT and GTHB Release (DONE)
- Check what is wrong with klockwork (Think Jenkins is screw up)
- Check Devi mail on RSMG (DELEGATE TO GIN)
- Check what is needed for RSMG release
    - Update MS
    - Get QAC, PM, Doublecheck, Klockwork
    - Update TF
    - Get CTC
    - Update H1

- Check what is needed for ROCH release
    - Update MS
    - Code Review
    - Get QAC, PM, Doublecheck, Klockwork
    - Update TF
    - Get CTC
    - Update H1
- Voice out Jenkins is not working
- See what is Gin opinion on setTransformation

Daily Log
Good Engineering
- things should be as standardize as possible so that there is as little friction as possible. 
- just like road signs. 
- daily goal is to be better than yesterday
- to think better and clearer
- to think before acting

# 28 March 2019
## To Do List
---
1. SOS
    * Email Sent

2. Check Klockwork for Davinci
3. One chapter of learnOpenGL
4. Create story for Unplanned support

## Daily Log
---
* goal is to solve problem
* goal is to be as productive as possible
* target October 2019. Need to be as prepared as possible. Everyday, learn for 1 hour

# 29 March 2019
## To Do List
---
1. Check Klockwork for Davinci
2. One chapter of learnOpenGL
3. Look through Mesh class 
4. 

## Daily Log
---
* goal is to solve problem
* goal is to be as productive as possible
* target October 2019. Need to be as prepared as possible. Everyday, learn for 1 hour

# 1 April 2019
## To Do List
---
1. Mesh Class 
2. One chapter of learnOpenGL (done)
3. Start Reading Vulkan and set up enviroment
4. Sign up for workshop 
5. Survey

## Daily Log
---
* goal is to solve problem
* goal is to be as productive as possible
* target October 2019. Need to be as prepared as possible. Everyday, learn for 1 hour

# 2 April 2019
## To Do List
---
1. Fix alpha blending  (done)
2. One chapter of learnOpenGL (blending)
3. Start Reading Vulkan and set up enviroment (started)
4. Sign up for workshop (done but not successful)
5. Survey 
6. Claim flexi benefits (done but not successful)

## Daily Log
---
* goal is to solve problem
* goal is to be as productive as possible
* target October 2019. Need to be as prepared as possible. Everyday, learn for 1 hour


# 3 April 2019
## To Do List
---
1. Read through Davinci (2T)
2. Write code for Vulkan( 2T)
3. Modify flexi benefits 
4. Survery (1T)

## Daily Log
---
 
# 4 April 2019
## Daily Routine
* Start the day by 
    1. Read Philosophy.md
    2. Read goals
    
* Start the day by learning (1T)
## To Do List
---
1. Read through Davinci (2T) (Done)
2. Write code for Vulkan( 2T) (Done)
3. Modify flexi benefits 
4. Survery (1T)


## Things to check/learn
---
* How to create offscreen surface with OGL (Frame Buffer>)
* 
## Daily Log
---
* start my day by learning Vulkan (1T)


# 5 April 2019
## Daily Routine
* Start the day by 
    1. Read Philosophy.md
    2. Read goals

* Daily trys
    1. list and solve problems that can be by little effort on my side
    
* Start the day by learning (1T)
## To Do List
---
1. Read through Davinci (2T) 
2. Write code for Vulkan( 2T)
3. Modify flexi beefits 
4. Survey (1T)


## Things to check/learn/build
---
* Offscreen rendering

## Daily Log
---