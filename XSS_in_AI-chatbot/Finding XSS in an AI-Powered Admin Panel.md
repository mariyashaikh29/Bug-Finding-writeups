# When a Chatbot Talks Back: Finding XSS in an AI-Powered Admin Panel

A few weeks into my role as a cybersecurity analyst, I was given scope to run an authorized penetration test on an EdTech platform. Standard stuff on paper: reconnaissance, access control checks, input validation testing. But one feature on the admin side stood out immediately: an AI-integrated chatbot built into the admin panel.

The idea was simple and genuinely useful. Instead of digging through dashboards and reports manually, an admin could just ask the chatbot a question and get the information pulled together automatically. It was meant to save time for whoever was managing the platform day to day.

As a tester, that's exactly the kind of feature that catches my attention. Any time user input gets passed to something that generates a response and that response gets rendered back on a page, I want to know what happens at the edges.

## The First Test

I started simple. I typed a basic script-based payload into the chatbot's input field, the kind of thing you'd use to check if a browser alert pops up when the input isn't sanitized properly.

The response came back blank.

At first glance, that looks like the application did its job. No popup, no visible reaction, nothing broken. If I had stopped there, I would have logged it as "not vulnerable" and moved on to the next test case.

But a blank response isn't the same as a blocked request. If the input had actually been rejected, I'd expect either an error message, a filtered/sanitized echo of what I typed, or no response at all because the request never went through. A genuinely blank response from something that's supposed to answer you is its own kind of signal. It means the input reached the backend and something happened with it, just not something visible yet.

![Payload entered into the admin chatbot, initial blank response](XSS_in_AI-chatbot/ss1.png)

## Reading Between the Lines

So I opened the page source. Sure enough, there it was: my payload had been encoded in the response. Encoding isn't the same as safe handling. It depends entirely on where and how that encoded value gets used later. If it's only ever displayed as encoded text, you're fine. If it gets decoded and rendered anywhere else in the application without re-sanitizing it, you have a problem.

That observation was the turning point. I stopped assuming the chatbot had "handled" my input correctly and started treating the blank response as incomplete information rather than a negative result.

## Making It Talk

Next, I tried a different angle. Instead of a one-off payload, I asked the chatbot to repeat back what I had just told it, essentially prompting it to echo my input rather than process it silently.

That's when the alert executed.

The chatbot repeating my own input back to me was enough to trigger the script. What looked like a dead end on the first attempt turned out to be a textbook case of reflected (and potentially stored, depending on how the conversation history was handled) cross-site scripting, sitting inside an AI feature that admins were expected to trust completely.

![Alert box firing after the chatbot echoed the payload back](XSS_in_AI-chatbot/ss2.png)

## Why This Matters

A few things made this finding particularly worth flagging:

**It lived in a trusted context.** This wasn't a public comment field or a contact form. It was inside the admin panel, a place built for people with elevated privileges. An XSS vulnerability there has a much higher blast radius than the same bug on a public page.

**AI features change the testing surface.** The chatbot wasn't just rendering static input and output. It was processing, interpreting, and sometimes repeating content back. That extra layer of "intelligence" doesn't make an application more secure by default. If anything, it adds more places where untrusted input can end up somewhere unexpected.

**A blank response is not a conclusion.** This is probably the biggest lesson from the whole exercise. Early on, it would have been easy to read "no visible reaction" as "no vulnerability" and move on. The only reason this finding surfaced at all was the habit of checking the source, not just the rendered output.

## The Bigger Takeaway

Testing AI-integrated features is still a fairly new muscle for a lot of security teams, including mine. The traditional checklist for XSS testing (try a payload, see if it fires, move to the next field) doesn't fully account for components that process input conversationally and generate dynamic output based on context.

What worked here wasn't a fancy tool or an advanced technique. It was slowing down after an inconclusive result, reading the actual source instead of trusting the visible behavior, and trying one more angle before writing it off.

If there's one habit I'd recommend carrying into any pentest involving AI-powered features, it's this: don't let "it looks fine" be the end of the investigation. Look at what's actually being sent back, not just what's being shown to you.
