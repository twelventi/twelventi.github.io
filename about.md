---
layout: page
title: About
permalink: /about/
---



`$ uname -a`

This website is my personal technical blog and resume site. Hopefully I have some interesting blog posts, but I'll be excited if even one person reads them. I'm working on adding projects to my [github](https://github.com/twelventi), and also finding interesting topics to write blog posts about. My first post and the inspiration for this site came from me hosting an event for the Fordham Computer Science Society, wherein we hacked a WiFi enabled "smart" coffee maker. I first had to reverse engineer the communications protocol and then figure out how to teach this in a way that beginner cs students could understand. 


`$ whoami`

My name is David. If you are here because this is linked from a job application, you already know my last name, so I will omit that here. My favorite languages are Python, C++, and Italian. 

I currently am back working at Scalzo Property Management as the IT Specialist; essentially I am the resident nerd, the person to disseminate everything related to computers. 

As of writing this I am currently the Research Director for the [Fordham Computer Science Society](https://fordhamcss.org), but I'll be handing over this position in about two weeks, after I officially graduate. If you're looking at this site and thinking "hmm this style looks awfully familiar", I may or may not have learned how to make a github pages site with Jekyll and Minima by inheriting the club website when I took over this position. I may be planning on making a more ~unique~ theme  to minima, but that's on the bottom of the TODO list. 

My previous experience includes the following:

<div id="work-exp">
    {% for exp in site.data.workexp %}
        <div class="exp-item">
            <img src="{{ exp.img }}" alt="{{ exp.alt }}">
            <div class="details">
                <h3>{{ exp.company }}</h3>
                <h5>{{ exp.title }}</h5>
            </div>
        </div>
    {% endfor %}
</div>