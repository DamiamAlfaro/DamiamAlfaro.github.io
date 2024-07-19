---
layout: post
title: "Pinpoint"
date: 2024-07-17 10:00:00 -0000
categories: blog
---

In the field of computer science and mathematics, and perhaps any expertise, can be mastered by a human with one single action: pinpointing.

The action of pinpointing can be described using as an adjective, and a verb:
![pinpoint_definition](https://github.com/user-attachments/assets/6e2032d2-9d79-4a44-91b4-d3d6fe53e7c9)

The action of pinpointing in life is crucial. Such action has the power to direct an impetus of ambition, distress, inspiration,
desire, revenge, among others. Conceiving the next approach is all one requires in order to make progress and materialize creativity and
intellect. A great example of such paradigm is language: language is primary motor of devising and articulating the abstraction that forms in our mind
when an instance of creativity or "connecting the dots" appears, aka the [Eureka Effect](https://en.wikipedia.org/wiki/Eureka_effect). Language pinpoints
words that by matter of grammar and [semantics](https://en.wikipedia.org/wiki/Semantics) which aids us to diminish the level of abstraction of an idea
and articulate it until its fruition. A great example of language as a tool to pinpoint an useful approach is the following fictional story: imagine a person that is leaking information to the press
regarding a financial scam that is ocurring within a corporate company back in the early 90s, this scam is affecting the lives of thousands of people by extracting small amounts of money from
their paycheck. The person who leaked the scam to the press is not aware of the word "track", "traceable", or "source". Why does this matter? Well, although the knowledge of this words might be insignificant for most
cases, that person being unaware of such words resulted in the locationg of the fax machine "traceable" number that directed to the office of the person. Would have he known the meaning/existence of such words he might've avoided such evidence from leaking by pinpointing the possible avenues to avoid the "track" of the fax machine, which ended up being the "source" of the scam scheme to the press. You see how knowing words alter and perhaps enhances our understanding and description of events? Expanding and enriching your lexicon might be seen as a boring activity nowadays since the best way to do so is by quietly reading, but it nevertheless expands your cognitive versatility, especially your [cognitive flexibility](https://en.wikipedia.org/wiki/Cognitive_flexibility) to adapt to almost any expertise easily. How? By describing it. Being able to describe something gives you the willingness to understand it, hence control it. In order to describe something acutely, one ought to expands the lexicon.
I recommend an excellent book on language [Meaning and Mind: a study in the psychology of language](https://www.amazon.com/Meaning-Mind-Study-Psychology-Language/dp/B00113B8FQ).

In this however, we are not talking about linguistic language per se, but about programming and computer science language. The importance of pinpointing something
in the field of computer science is a paradigm that has aid programmers throughout decades. The way we call the action of pinpointing an object in computer science is
[Indexing](https://pandas.pydata.org/docs/user_guide/indexing.html). This paradigm is vital for the life of a programmer. 

The beauty about programming, and specifically about data science is that with the right tools, anything at all can be extracted as a datum. 
If it exist as a visual data point, one can access it with programming. As long as it exist in the display of a computer, it can be accessible. With the paradigm of [Web Scraping](https://en.wikipedia.org/wiki/Web_scraping), the the world wide web, and anything within it becomes an extractable datum. The world wide web thereby becomes the biggest data base known to man.

I show the benefit of pinpointing in computer science in the [extraction of strings from planetbids prospective bidders](https://github.com/DamiamAlfaro/Earth-Prototypes/blob/main/Europe/Text_Strings/A.ipynb) by finding the attributes that I am looking for within an ocean of strings only by using indexing, or in other words, **pinpointing** in the field of computer science. See example below:

```python
# The address is only composed of two lines, either 1) Regular street address (e.g. 123 StreetName) and 2) the remaining address line (Municipality, State Zip Code)
    if bool(re.match(r'^\d', refined_text_lines[iteration_list_contact[attribute_iteration]-2])) == True or refined_text_lines[iteration_list_contact[attribute_iteration]-2].startswith(("PO","P O","P.O","P.O.")):
      street_address_part_2 = refined_text_lines[iteration_list_contact[attribute_iteration]-2]
      subcontractor_name = refined_text_lines[iteration_list_contact[attribute_iteration]-3]
      street_address_part_complete = street_address_part_2 + ' ' + street_address_part_final
      subcontractor_addresses.append(street_address_part_complete.upper())
      subcontractor_names.append(subcontractor_name.upper())

    # Or, the address is composed of three lines, either 1) Regular street address (e.g. 123 StreetName), 2) a Suite or Apartment number (e.g. STE 2) and 3) the remaining address line (i.e. Municipality, State Zip Code)
    else:
      street_address_part_2 = refined_text_lines[iteration_list_contact[attribute_iteration]-2]
      street_address_part_1 = refined_text_lines[iteration_list_contact[attribute_iteration]-3]
      subcontractor_name = refined_text_lines[iteration_list_contact[attribute_iteration]-4]
      street_address_part_complete = street_address_part_1 + ' ' + street_address_part_2 + ' ' + street_address_part_final
      subcontractor_addresses.append(street_address_part_complete.upper())
      subcontractor_names.append(subcontractor_name.upper())
```

If you want more context on what the loop and variables mean and do please visit [A.ipynb](https://github.com/DamiamAlfaro/Earth-Prototypes/blob/main/Europe/Text_Strings/A.ipynb), but briefly: my goal with this program was to allocate a text document containing prospective bidders' information (address, phone number, email, etc.) and allocate that information into an excel to both: 1) keep a record of it and 2) create a [macro that sends emails to each subcontractor found within the text document](https://github.com/DamiamAlfaro/Earth-Prototypes/blob/main/Asia/AutomationMacros/A_A.bas). As you can see, all I do is find the index of a word or set of words that meets two criteria: 1) repeats itself only once, and 2) repeats itself in all subcontractors. The word in question is "Contact", which is always found in the same position within each subcontractor; after the address data and before the phone number data. With those three facts and the help of our human ability to pinpoint I was able to find the rest of the data that I needed from each subcontractor. Why? Because by knowing the position of one datum I was able to find the rest, all I needed was the index, also known as **pinpointing**.
