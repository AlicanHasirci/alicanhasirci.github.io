---
author: Alishex
comments: true
date: 2014-07-21 12:55:42+00:00
layout: post
slug: regular-expression
title: Regular Expression
wordpress_id: 87
categories:
- Programming
tags:
- parse
- pattern
- regex
- regular expression
- string
---

Most commonly seen as "Regex", is a forgotten art of pattern searching and the most useful if not one of the useful things that you can use while parsing strings or simply working with strings. Usage of regex is another plus where all common programming languages have a native support for it. 

So far everything seems nice and dandy then why it isn't commonly used? Cause even the look of it is intimidating for a person who never worked with pattern related issues. Using a pattern as **"(\d{1,3}\.){3}\d{1,3}" **only to match seems kind of overkill but when the situation forces your hand to use a swift method with zero false-positives regex is what you need.

A good start for a regex would be looking for a cheatsheet to see what means more then it seems? Here are some frequently used terms in regex to get you started:

**\w** - Matches a single word character (alphanumeric & underscore).

**\d** - Matches a number.

**\s** - Matches a whitespace.

**[A-Z]** -Matches a character between given characters. While giving these letter in lowercase makes the match in lowercase, it is possible to a give  a number range also.

**[abc]** - Such a pattern implies that the match can be either one of the contents inside brackets.

**{1,9}** - This is a quantifier telling us how we should look for for the preceding term. By typing {1,9} we are saying that the preceding pattern can ben between 1 and 9 of quantity.

**+** - Matches 1 or more of the preceding token.

**\*** - Matches 0 or more of the preceding token.

These are the basic character classes that can be used as a match. By putting a quantifier behind such character classes you can tell regex to look for what and for how many.
Also before checking an example a good thing to remember is the use of parentheses. Just like in math, everything between "(" and ")" is considered as a group so putting a quantifier right after a matching group will effect the whole group rather then a single token. Now lets take a look at an example.

    
    (\d{1,3}\.){3}\d{1,3}
    


How does this pattern matches an ip address? Lets take the part inside the capturing group first.
"\d" means a digit and there is a quantifier telling us that it can ben between 1 and 3 in terms of quantity. So this pattern matches everthing between 1-999. Right after that we see a "\." which is a escaped dot, since the dot is a quantifier. This pattern is capsulated in a matching group with a quantifier of "{3}". So now we are looking for three group of numbers which can be between 1 to 999 followed by a dot which are also following one another. Then all you need is one last number group representing the last octet represented by "\d{1,3}". Now this can match to such an IP address also "dummytext123.123.123.123dummytext". So if you want to avoid this you can use "\b" which matches a word boundry.

Lets check another example. Say, we want to match a all the emails in a text file. Now we know that the maximum character length for the part before "@" is 25 and minimum 3, can contain letters and numbers also dots but not underscores. Also remember to cover sites with suffixes such as "co.uk"

    
    
    \b[\w\d\.]{3,25}@[\w\d]+(\.\w+){1,2}\b
    



If you followed this much, this pattern is pretty much straight forward where we try to match for words who have 3 to 25 letters,digits or dots before a @ character also has a word followed by a dot and another word possibly also another dot and a word.
