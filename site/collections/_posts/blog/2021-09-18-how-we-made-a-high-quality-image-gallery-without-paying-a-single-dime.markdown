---
date:   2021-09-18
title:  How we made a high quality image gallery without paying a single dime
description: A dive into how the poor man's necessity sharpens one's problem solving ability (or how we took advantage of multiple million dollar companies' free services).
tags:   [programming, project, virtual-photography]
image:  '/images/blog/post-1.jpg'
category: blog
---

In the last couple of years, I've been getting more involved with virtual photography: the art and practice of photography in virtual environments such as videogames.

I've been actively participating in <b>Framed</b>, a small discord community of very talented virtual photographers. We share screenshots, give each other feedback, discuss techniques, develop tools, or just kick back and hang out. Since I joined, admins have used pinned shots in the channel where we share them to show new members the "best" of what the community had to offer and get a glance of what's possible in the medium.

Unfortunately, for some reason, discord limits you to 50 pinned messages per channel. In order to pin new messages, admins had to unpin old ones. This was not an ideal approach. We thought that having a separate channel for showing the "best” shots would be nice. Some suggested that having a bot that could forward these “best” shots into a channel of their own would be closer to what we wanted. But that bot would have to be custom made as the specific features we required were too niche to exist. Fortunately for us, at that time I had a job interview coming up in a few weeks, so I figured I could practice some python while doing something useful with my time. And from the title of this article, you may have already tell that the project snowballed from there and ended up in a [website](https://framedsc.github.io/HallOfFramed/) of its own.

Before getting deeper into this, "article" I guess? You should know that I won't be talking in depth about the technical details behind the tools we used nor the design patterns I followed (or that at least tried to), since there is a ton of information about them online written by people way more capable and experienced than me. Instead, my approach with this post would be to present you with the problems we encountered along the way and the clever workarounds we managed to pull off, besides getting to talk about things I like without anyone interrupting me. Hope you enjoy it.
 
## The original plan

What we planned at first was a simple bot that read the amount of reactions a message with a shot had and, based on that, decide if it should send an embedded message like the image below with the screenshot and some extra information to another channel.

<p align="center"><img src="/images/blog/post-1/botembed.png"></p>

The obvious question arises, how do we determine that a shot should be highlighted? We thought about using a prebuilt bot that added a reaction to every shot, and depending on the amount of people reacting to a specific emoji it would highlight it or not, but we didn't want to change the way users interacts with the shots (which we didn't completely manage to achieve, more on that later). Besides, we wanted to highlight shots retroactively since we had 2 years worth of them, so a more thoughtful solution was needed.

What we ended up doing was counting the amount of unique users interacting with a shot through reactions, and if that number was higher than a trigger value then the shot was highlighted, easy peasy.

Some fine details had to be decided though, like how far back the bot should check shots reactions to decide if it should highlight them, because checking every shot ever (which at that time were about 25k shots) was excessive and quite dumb honestly. We settled up with just checking shots that are less than 7 days old and it has worked quite well so far.

So, with this now defined I was ready to start coding our little bot, but there was a catch. Since we wanted to highlight stuff retroactively, how can we decide on a trigger value when the oldest shots had so little reactions despite being great because of the server having less people at that time? The solution was, as many things in life, <b>MATH</b>.

What I did was use a trigger value that changed using a linear interpolation between the trigger value the bot would be using when checking the shots of the last week and whatever seems fit to use for when the community first started. And after playing with a couple of values we reached some triggers that gave us a list of shots pretty well distributed across the almost two years of life the server had at the time. As it turned out, sometimes simple solutions work the best.

Did you follow all of that? No? Don't worry, what you have to get from this is that math is awesome.

Because admins still wanted the ability to highlight shots manually (and because they hate democracy) I wrote a way for them to tell the bot to highlight a specific shot no matter the amount of reactions it has.

<p align="center"><img src="/images/blog/post-1/nodemocracy.gif"></p>

## The devil is in the details

Now it was finally time to start coding the damn thing. To do it I used <i>discordpy</i> python library which worked flawlessly and is very well documented. If you are planning on writing a bot of your own and are familiar with python you will feel very comfortable using it.

An important part of making this was actually figuring out where to host the bot in question. I didn't have to research much to find a website called [Heroku](https://www.heroku.com/) that allowed hosting running python scripts for free, but of course, there was a catch (more than one actually). You can only run the bot a certain number of hours each month, which means having the bot offline for a few days at the end of the month. But, since the bot crashed from time to time and was back online whenever Heroku decided to reset their servers (and because the nature of the bot did not require it to be on 24/7) we were able to save some hours each week. Did we beat the system with negligence? Absolutely.

Still was far from ideal. Heroku also offered a way to run the script using a scheduler, but after preparing the bot to work in that way I couldn't use it either because I didn't put money on the site. So, what we ended up doing was using one of those services that creates a virtual credit card with a limit of money on them (often used for creating accounts with free month promotions for ingressing your credit card like Netflix) to "verify" the account. And with that we cheated the system a second time.

With this non-freemium account we didn't need the scheduler anymore since we had the double amount of hours to run the script. This might not seem useful at first glance since a bot that highlights shots does not require to be on 24/7, but we figure we could add more functionalities as we came up with ideas, like letting the admins highlight whatever shot they want.

Another one of those ideas was one that we talked about quite a lot since almost the beginning of the server: making a site where we show these highlighted shots.
 
## The site

Now, a site might seem like a daunting idea, and it is. Fortunately some of the members have years of front-end experience, so I didn't have to worry about the site myself, I just needed to provide a way for it to read the shot information provided by the bot. And with the expertise of more intelligent people than I, we reached the conclusion of using the JSON files for storing this information, at least for now.

Fortunately, the server already had a static github site where we upload guides and information about how to take screenshots, so that was covered. And since all the shots are already on discord's own servers, and one can access files uploaded to a server without being part of it, we also had a free host where our shots were already uploaded. Not only that, but if enough people boost the server, everyone can upload shots up to 50mb, so its ends up being a great way of storing high quality images.

We actually had even more advantages going this route. We don't have to pay extra if the traffic increase, discord doesn't compress the shots like other services may do, and, according to the ever trusting internet people, its actually a [pretty good place to store files on the long run](https://alternativeto.net/list/149/reliable-long-term-image-hosting-sites-that-will-stand-the-test-of-time/).

I am actually kinda surprised none thought of doing something like this before. Hopefully we aren't taking advantage of discord's free will (too much) and they change how things work. We did however looked it up and apparently we are not breaking any discord's terms of services here, so please internet police officer, carry on.
 
## Databases make me lazy

Okay, we had our shots uploaded to a host, but we now needed a way of storing those links in our JSON file, so we needed some way of managing this file as a database. Because the database we needed would be way too small for a SQL database or something alike (and because I didn't feel like dealing with real databases) we ended up using a python library called <i>TinyDB</i>, which is a <i>lightweight document oriented database optimized for your happiness</i> according to its docs, so it was exactly what we were looking for.

Now we only needed a place to store and update over time the JSON file that would act as the database with the shots information, but one of Heroku's limitations was that, when the servers are restarted (once a day at least) the files generated were deleted, so we couldn't keep the JSON file there (besides, there wasn't a way to access that file either, or at least that I am aware of). So, with the need to store this file somewhere and the desire to keep using rich companies' resources for free I came up with the solution: <b>Github</b>.

It was perfect, not only do they host the file freely in a repository and it can be accessed very easily but, as one could have imagined, there was already a python library that allowed me to interact with github called <i>gitPython</i>. Isn't Python lovely?

Before continuing on, I would like to point out that, if you ever play with this library to push a repository, don't use your main github account credentials...

<p align="center"><img src="/images/blog/post-1/githubcommitsups.png"></p>
 
## Names are hard

Now, since we wanted to put these shots in a site, we figured it would be best to include the title of the game so the site was easier to use, so how do we find these?

One of the (fairly recent) rules of this discord community is that the only text allowed in the channel where we share screenshots were the names of the games the shots were from. So what I ended up doing was, given a shot that the bot wanted to highlight, pick up the closer message (within a range of messages) with text from that same author. And if I am able to say it myself, it works wonderfully.

Unfortunately this rule wasn't a thing since the beginning of the server, so some of the shots of the earlier days of the community didn't have the proper name attached to them (and a particular shot had a message so long that crashed the bot, shit happens I guess). Fortunately one member offered himself to correct the database manually to cover these cases (thank you Andy). And with that out of the way we were almost done!

We did however have to think about the shot thumbnails. Since we wanted some kind of grid for the site, we couldn't just load the original shots for every image displayed in the grid, it would be way too slow to load. So what do we do? We thought about maybe paying a host for the thumbnails, but I was stubborn to finish this project without paying a single penny, and after some minutes of going back and forth on the topic it finally hit me. We were already using discord as a host for the shots, why not also use it for hosting the thumbnails too? (Sorry discord devs, don't hate us).
 
## Final details and polishing

We still had to create said thumbnails tho. To do so we ended up using, you guessed it, another python library (without these python libraries we wouldn't have had a chance with this project). This one is called <i>Pillow</i>, and it's used to manipulate images in a lot of different ways. And after trying different combinations of resizing algorithms and formats we reached one we liked and were ready to move to the next task.

At this point the site was working pretty solidly, altho I cannot give a lot of details about its implementation since I didn't write it myself and because of my lack of knowledge of front-end development, but we had a prototype which we were iterating upon. One of the things I wanted to improve at this point was the author's information. I thought that having some social media links from the author when seeing a shot on the site would encourage people to look at more stuff from that person.

We had a channel in the community to share our social media links, portfolios sites, etc., so all I had to do was to tell the bot to save those links in another JSON database file. I also saved some info discord provided about an user so I could link every author's shot in the shots database with the associated information in the authors database, and with that we were ready to go.

I've talked a bit about these databases the bot created, which as you might have guessed, are kind of toy databases. So it's worth to ask the question: Is it scalable?

Well, it depends. It's been 6 months since we launched the site, and it already has over 1700 shots, BUT the size of the shots DB file is only 800kb, which is practically negligible. If the size becomes a problem in the future (the bot has to redownload the database files every time heroku resets its servers, so it might be) maybe we could use a compression algorithm like gzip. What could be a problem is if the site would be able to use a file like this with so many entries, but I don't have the front-end knowledge to know that. Databases way bigger than ours are being used by sites everyday, but they have proper databases systems and not some JSON file. The question is then if our method of handling the shots database will have to change with time, and my intuition says yes, but we can deal with that later on.

And with that the bot was pretty much done! It was then up to the members that were working on the site to polish it before we could announce it to the world. How exciting!
 
## The aftermath

<p align="center"><img src="/images/blog/post-1/hoflogo.png"></p>

We released the <a href="https://framedsc.github.io/HallOfFramed/">Hall of Framed</a> (get it?) website the 12th of February of this year with a pretty nice reception. We originally built the site as a little extra of our internal highlighting system, but it has proven to be not only a little glimpse of what the members of the server can do, but as a card of presentation of the hobby as a whole to anyone who isn't familiar with it. It's safe to assume we were more than happy with the result.
 
<blockquote class="twitter-tweet" align="center" data-lang="en"><p lang="en" dir="ltr">FRAMED (@FramedSc) <a href="https://twitter.com/FramedSc/status/1360336268832440328?ref_src=twsrc%5Etfw">https://twitter.com/FramedSc/status/1360336268832440328?ref_src=twsrc%5Etfw</a></p>&mdash; FRAMED (@FramedSc) <a href="https://twitter.com/FramedSc/status/1360336268832440328?ref_src=twsrc%5Etfw"></a></blockquote>
<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" align="center" data-lang="en"><p lang="en" dir="ltr">Jim2point0 (@jim2point0) <a href="https://t.co/mP1NPzi4JY">https://t.co/mP1NPzi4JY</a></p>&mdash; Jim2point0 (@jim2point0) <a href="https://twitter.com/jim2point0/status/1360297459168247817?ref_src=twsrc%5Etf"></a></blockquote>
<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" align="center" data-lang="en"><p lang="en" dir="ltr">Petri Levälahti (@Berduu)) <a href="https://t.co/qi5CxRQ3j0">https://t.co/qi5CxRQ3j0</a></p>&mdash; Petri Levälahti (@Berduu) <a href="https://twitter.com/Berduu/status/1360311432684920832?ref_src=twsrc%5Etfw"></a></blockquote>
<script async="" src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

However there were some other unprecedented consequences. Even though I tried to approach the system without changing how people think about sharing their shots there was a factor I was not taking into account: how the mere existence of a “gallery” could already alter how people engage with sharing their shots.
The close nature of the discord group meant that sharing a shot in the community was only meant to be seen by the members of it, since it's not really a <i>social media platform</i> where you can share or retweet other people's stuff. So instead of reaching for the whole internet's approval in the form of imaginary internet points, we shared shots to ask for feedback, show what we came up with and yes, still expect internet people's approval, but in an enclosed community. Which felt more honest in a way since there was no final goal, if that makes any sense. With the implementation of this gallery the goal, for a lot of people, changed (even unconsciously) from just sharing stuff in a selected group to trying to reach the site, which tied up with a topic some of you may already be familiar with, and that is social media anxiety.

I am fortunate enough to not suffer it myself (yet), but I have seen how it can deteriorate someone's mental health into smithereens. That feeling of thinking your work isn't good enough because its not doing numbers had lead to severe consequences to some people, bringing a hobby they once enjoyed to something they despise.

More than just merely our fault, I think the situation is more of a symptom caused by the larger social media platforms of today and our relationship with them. We weren't anticipating stepping into such a vast and complex problem when designing this whole thing, so we were not really sure how to tackle it. So out of nowhere we were dealing with designing some "social media system" that doesn't impact people's mental health negatively, if that's even possible.

What we ended up doing for now was adding an option for users to opt out of the system with a role, so the bot could ignore shots from people with said role to help reduce the anxiety the system might cause them. Far from being the solution, it's what we came up in the meantime while we reach for better ideas.

The problem is, I believe, way larger than what some programmers can deal with in a project like this, but we believe that there is a better alternative for building a healthier relationship with social media platforms, even though changing them is probably against the companies monetary interests. Hopefully someone more capable (and probably with a more fitting background) can design an alternative like this in the future, but I am sure it would still require effort from us, the users, to make it work.

That might have been a darker way of concluding this section than I anticipated. I do believe that if we were to put all the pros and cons that came out of this bot and site in a balance, the result would still be very positive.

## So, what did I learn?

I learned quite a few things as you might have expected. Besides obviously learning how to code a discord bot and how to handle async operations in python, I learned some lessons that I consider valuable enough to share (or at least they are for a dumb undegraduate computer science student with no work experience like myself):
 
<ul>
    <li>
    <b>- Python is one hell of a programming language:</b> I have experience with python from college, altho I had my problems with it. But after seeing how versatile it is with all the user created libraries, I now understand how powerful of a prototyping tool it is, and will definitely be using it whenever a new project allows it. Hopefully I don't grow too used to relying on other people's libraries since that can prove problematic, but only time will tell.
    </li>
    <li>
    <b>- You can't make the perfect skeleton:</b> Even tho I did whatever I could to plan all of the bot features ahead, trying to design all the systems following design patterns, single-responsibility principle and all kind of stuff I learned at a course (which I still have to do the final, mind you), I learned that you can't plan out every possible feature that you haven't even thinked about no matter how far ahead you think. You won't be able to design your code in a way that all the features you come up later will be easy and pretty to implement. The experience of putting your project to test will be the only way to come up with all the features worth implementing. It's inevitable you will contract technical debt, which will have to be paid sooner or later.
    </li>
    <li>
    <b>- Think how your code can affect people:</b> The last lesson I learned through this is how we, as programmers, probably don't have a notion on how much our software can impact other people, something that might be fairly common in the industry. Even though one can’t anticipate every single impact a project might have, not doing anything isn't the answer either. And giving at least some thought to the social and psychological impact our projects might have could be a good first step towards the right direction.
    </li>
</ul>

You can try out the site at [framedsc.github.io/HallOfFramed](https://framedsc.github.io/HallOfFramed/) or look at its [source code](https://github.com/framedsc/HallOfFramed) if you are interested in that sort of thing.
And you can look up the bot's code in [its github repo here](https://github.com/originalnicodr/CuratorDiscordBot) (try not to be too judgmental, I am kinda behind that technical debt I talked about).

Thank you so much to [Otis_inf](https://twitter.com/FransBouma), [Jim2point0](https://twitter.com/jim2point0), [Fraulk](https://twitter.com/Freaksboi) and [Choc](https://github.com/chocmake) for collaborating with this project. You are all truly wonderful.
 
<hr>

And that should be it, I apologise if this post wasn't as technical as you might have expected (or dived too deep in details, depending on where you are coming from). It's my first time writing anything like this so I am not expecting it to be the smoothest blog entry on the internet. I do hope however that it was an interesting read and you had at least a fraction of the fun I had while working on this and, more than anything, that I could successfully spread some of my passion for the project to you.
 
Have a good one.
