---
title: "How I made $7500 from My First Bug Bounty Found on Google Cloud Platform"
published: true
---

Greetings all, this is my first article about my experience learning about security vulnerabilities
and hunting for bug bounties. I've done some reading and sporadic hunting over the last year or
so, and more recently came across my first paid bounty as a side effect of my day to day work.

I discovered that Jupyter notebooks created through the Google Cloud UI were insecure over port 8080 due to 
misconfiguration of the jupyter server and virtual machine configuration. The vulnerability was discovered due to
malicious activity on the machine and a notification that we were violating an agreement with Google for their use 
policies because there was bitcoin mining activity on the machine. This notification led to an investigation of the 
machine, which resulted in discovering the method of unauthorized access.

This particular virtual machine was being used by our recommendations team to build and test AI models.
I had no knowledge or experience with the setup prior to investigating the issue, and wasn't
initially asked to look into the issue. The email from Google's support team indicated which virtual
machine was the culprit and since no one at my company admitted to mining bitcoin with company resources
we decided to dig a bit further.

When I started looking into the matter I ssh'd into the host and started looking around for symptoms of
the problem. I checked logs for login attempts and didn't see anything interesting. I also started looking
for signs of anything the attacker might have done to maintain persistent access. While checking the
running processes I noticed there was a juptyer server running on the host (if I had complete contxt
about the situation I would have known this from the start).

After discovering the jupyter server I started doing some basic research about the setup for
the server. From the documentation I learned where the config would be on the host. The config 
showed that the server was running on port 8080 and that I also took note of what looked like
a typo in the jupyter_notebook_config.py file.

```
c.NotebookApp.allow_origin_pat =  '(^https://8080-dot-[0-9]+-dot-devshell.appspot.com$)|(^https://colab.research.google.com$)|((https?://)?[0-9a-z]+-dot-datalab-vm[-0-9a-z]*.googleusercontent.com)’
```

Since I thought there was a server running on port 8080, I found the external IP address for the machine
and checked what came up in the browser. Sure enough, it takes me to the jupyter server interface.
I poke around a bit to see what I can do, and to be sure that my local cookies/etc aren't giving me
access I do this from a machine I don't access work projects from and through an incognito browser.
After about ten minutes I establish that I can run a terminal inside the browser, and I can do anything
I want with root permissions. What's worse is that a slightly stale version of our companies primary project
repository is on the machine. 

At this point I'm thinking someone at my company spun up a virtual machine, started running
a jupyter server and pretty much left the keys to the kingdom exposed. I quickly get on the email 
thread with our CTO and explain what I found. I also recommended shutting down the machine immediately
since there were few certainties about what the original attacker was able to accomplish with
their time on the host. I also start asking more questions about how they typically access the machine
and who created it. That's when my CTO shares a URL for the google cloud console which finally gives me
the entire context.

Thankfully it ended up not being a negligent setup by one of my coworkers. The URL my CTO
shared led me to the google cloud dashboard for the notebooks under the AI category of products.
After spinning up another instance like the one my company used I determined this was the default 
configuration. This is also where I determined that the access to these machines was meant to be
funneled through the cloud console UI. Inside the dashboard for your notebook instances , there 
were links to access each instance. The typo I found earlier was meant to apply a regex to prevent accessing
the machine if you didn't come from the correct origin URL. Implying you would access these machines
from the google cloud console where you are already authenticated.

Initially we tried to report this issue to the team that contacted us about the policy violation.
When they indicated they were unable to act on the information, I decided it was serious enough
to file a bounty for the bug. Eventually it was acknowledged, fixed, and I later confirmed that
this issue was resolved. The deployed fix now removes http/https access for the hosts over the 
external IP address for those instances by default.

At the end of the day I was very happy to receive the reward and get that sense of validation from
my research and efforts with bug bounty programs. On the other hand, I also realized that most
of the skills I had learned while researching vulnerabilities didn't come into play. I felt like
the skills that were most valuable in this situation were all skills I developed from programming.
My knowledge of virtual machines, the ability to quickly read documentation for a service, interpreting
the configuration for the server and basic linux debugging skills won the day here. 

I'm looking forward to sharing more of my adventures in the future, stay tuned!