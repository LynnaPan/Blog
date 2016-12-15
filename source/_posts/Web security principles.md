---
title:  Web security principles
date:   2016-07-07 19:10:53 +0800
categories: Web
---
## Web security principles

Total security is unachievable.

	1. Least privilege 最少的权限
	2. Simple is more secure 越简单越安全
		a. Legacy code
		b. Built-in functions are often better your own versions
		c. Disable or remove unused features when possible
	3. Never trust users 永远不要相信用户
		a. Well-meaning users can cause problems
		b. Be paranoid
		c. Don’t even trust admin users completely
		d. Use caution with contractors
		e. Even offline
			i. Phone
			ii. Email
			iii. Printing
	4. Expect the unexpected 
		a. Prevent the crime before it happens
		b. Consider "edge cases"
		c. Security is not reactive
	5. Defense in depth
		a. Layer defense
		b. Redundant security
	6. Security through obscurity
		a. More information benefits hackers
		b. Limit exposed information
		c. Limit feedback
		d. Obscurity does not mean misdirection
	7. Blacklisting and whitelisting
	8. Map exposure points and data passageways
		a. Input
			i. Urls
			ii. Forms
			iii. Cookies/sessions
			iv. Database reads
			v. Your public api
		b. Outgoing exposure
			i. HTML
			ii. Javascript/json/xml/rss
			iii. Cookies/sessions
			iv. Database write
			v. Third-party APIs
