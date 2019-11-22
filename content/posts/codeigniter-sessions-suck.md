---
title: "Codeigniter Sessions Suck"
date: 2010-11-21T12:20:24+05:30
draft: true
toc: false
images:
tags: 
  - codeigniter
  - php
  - sessions
---

> I have restored these posts from wayback machine. I was young and stupid then, so I would take my words with a hint of caution too

I have been developing on Code Igniter for some time now. Its a good, light framework for developers who mostly deploy websites on a shared server. There are no server side changes to be made (like in Smarty templates) and it is not as bulky as Symfony OR Cake.

Recently, while working on a project, I ran into some serious issues with the sessions Library. This is the first time I Code Igniter has given me issues and how.  I usually set the session using `$this->session` and it always worked. However, for this website, if I browsed to another page, my custom userdata that I set using `$this->session->set_userdata()` was somehow not accessible. I tried passing it using a single array also, as suggested on a website, but nothing seemed to work. First, I thought this is an issue with the way the controller was written, so I made a few changes with the logic but faced the same  problem again. Determined, I started reading a few forums and found others facing the same problem. The problem was similar to the one mentioned here: http://stackoverflow.com/questions/1703770/code-igniter-login-session-and-redirect-problem-in-internet-explorer only that I was facing this in every browser and not only IE.

My session variables were getting nuked by some stealth submarine I couldn’t see. Somewhere, on the page, someone mentioned an alternate Session Library called Native Sessions (http://codeigniter.com/wiki/Native_session/ ). This is called native sessions because it uses the PHP sessions class to create and destroy  sessions. Plus, this was compatible with CI_Session; except for a few changes, everything had to be the same.

This class was like a super high tech destroyer fitted with sonar detection, which would save me from this stealth sub. But it failed. I included the new Sessions.php class file in my /application/libraries folder and called it a night. The next morning, when I started work again, the code still had the same issues.

My piece of code looked like this:

```
$this->session->destroy();
//Create a fresh, brand new session
$this->session->CI_Session();
//Set session data
$this->session->set_userdata(array(‘id’ => $q['id'],’username’ => $q['user']));
//Set logged_in to true
$this->session->set_userdata(array(‘logged_in’ => true));
```

If I comment the line where I create a new session, `$this->session->CI_Session`, the problem seems to disappear. I understand this is because the constructor function for this class initializes the session, so there is no need to do it explicitly, but if this is the case, then why are my session variables disappearing. Ideally, if I start with destroying any old sessions and then start a new session, my custom variables should appear. I did not have the time to figure out what was wrong, so I left it that way. It still is the same way and has been working seamlessly. If anyone knows what the problem is, do let me know. Please remember, I am not using the Code Igniter sessions library included in /system/libraries. I am using a different library which can be found on http://codeigniter.com/wiki/Native_session/

Looking forward to an explanation. Hope this post helps out other tired souls who have sore eyes trying to figure out what went wrong with their websites session handling.
