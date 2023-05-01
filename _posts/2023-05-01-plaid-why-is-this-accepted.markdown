---
layout: post
title:  "Plaid?" # This is the title that is displayed to users
date:   2023-05-01 00:00:00 -0500 # Remember to set this the same as the filename to avoid confusion
categories: jekyll update # Don't mess with categories unless you know how Jekyll works
author: David Bono
# Uncomment to attach downloads
# downloads:
#     - file_url: /assets/downloads/example_file.pdf
#       name: Example File
---



# Plaid?

I've very recently had the unfortunate displeasure of discovering a financial service I'm using removed the option for people to transfer money directly from a bank account to for interface with the service, and only provides options for Plaid, a Card, or Wire Transfers. The latter two options incur fees to transfer your own money, so they're not really reasonable.



I attempted to use Plaid. 



After clicking the plaid option for bank account linking, I was presented with two fields:


"Online Bank Username:"
"Online Bank Password:"



The alarms started going off in my head, as they would in the mind of any security conscious individual. 

- "This website is not my bank, why are they asking for my login information?"

- "The website I'm attempting to link to my bank account is a (seemingly) legitimate company that many people use daily?"

- "This can't be real, where is the redirect to 'sign in with my bank?'"



I might be late to the collective outrage about this service, but after some digging and rigorous google searching, I discovered that _yes this really is how Plaid works, and nobody seems to care_. 



For most banks, Plaid stores your online bank account username and password, and then runs something probably like headless Chrome to sign in to your online bank account with your credentials, and then programmatically clicks the buttons that you would if you had signed in yourself when there is a call to their "API". Further, there are some financial institutions that have MFA on their online accounts with whom Plaid doesn't play nicely. The Solution? Disable MFA.............



All of these things seem to violate basic security principles, for example, the Principle of Least Privilege. With access to your bank account login information, Plaid could theoretically view _all_ information available on your online bank account. If you're using a service like Venmo for example, the only necessary information to complete a Venmo transaction is your bank account balance and the amount to transfer, but the with the way Plaid works, they now  have the capability to see if you have a credit card with your bank, loan information, investment information, etc. Further, with the provided access, Plaid would have the capability to open accounts or apply for loans in your name with your credentials. 



The other, not necessarily security issue, but more of a UX one, is that a security conscious user would change their password relatively frequently. Changing your online bank password then causes Plaid connections to stop working because they will attempt to sign in with the old password you provided them. The user now has to change the password on all places they connected to plaid. Even moreso, if a password for an online bank account is changed, and a Plaid service continuously requests asynchronous data from the bank with the old credentials, this could trigger security protections to lock the online account after multiple failed login attempts.



Plaid might not be and likely isn't necessarily the "malicious" actor in this conundrum (if they did things like arbitrarily open accounts for you, they'd probably go out of business because of degrading trust amongst the average user), but if they get hacked and there's significant data leakage (some might argue this isn't an if, but a when), a true malicious actor could start taking actions as you on your online bank account. This type of breach would be even worse than some bug or backdoor though, because the security team at your bank would have an exponentially harder time differentiating legitimate requests you made by signing in to your own bank account, from ones an attacker made by signing in the same way you did. 



## What to do from here?

This whole ecosystem is definitely mindblowing, and the easiest answer to avoid this problem is to simply refuse to use Plaid, or any service that requires you connect via plaid. However, I feel this is akin to saying "the most secure computer is the one that's off and in your closet". It might not be a viable option to avoid Plaid, because there are no services you need that allow you to circumvent it. 



There are some banks whose interface with plaid is not a username/password, but rather offer a proper Oauth "sign in with my bank account" option. This list is miniscule compared to the entire list of banks, and in order to event find it you have to create an account with plaid and visit the [place to link Oauth Banks](https://dashboard.plaid.com/team/oauth-institutions) to add api keys for oauth at the various institutions. Secondly, the app you are trying to connect to with Plaid has to configure oauth for these banks. 

At the time of writing this the banks that allow Oauth are; Chase, Capital One, Wells Fargo, Bank of America, USAA, U.S. Bank, and Robinhood. Of these, only Chase, Wells Fargo, Bank of America, and USAA specifically deny access to plaid via account username and password.



If you _really_ need to use Plaid, my best alternative idea is to find a no minimum checking account at one of these institutions, and transfer money from your preffered financial instution to the no minimum checking account, and use the Oauth'd online account as a proxy to minimize your exposure to the risks of Plaid. I'd personally choose U.S. Bank if I could as my "proxy" account but they are not available where I live.





 




