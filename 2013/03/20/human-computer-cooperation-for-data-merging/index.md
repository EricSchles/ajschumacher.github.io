# Human-Computer Cooperation for Data Merging

This problem comes up all the time, but the instance that got me thinking about it most recently was this: The NYC <a href="http://mta.info/">MTA</a> provides subway station geo-coordinates like <a href="http://www.mta.info/developers/data/nyct/subway/google_transit.zip">this</a>:

```html
127, ,Times Sq - 42 St, , 40.75529,  -73.987495, , , 1,
624, ,103 St,           , 40.7906,   -73.947478, , , 1,
119, ,103 St,           , 40.799446, -73.968379, , , 1,
```

But their subway turnstile data is linked with a table like <a href="http://www.mta.info/developers/resources/nyct/turnstile/Remote-Booth-Station.xls">this</a>:

```html
R032    R145    42 ST-TIMES SQ   1237ACENQRS    IRT
R180    R252    103 ST           6              IRT
R191    R170    103 ST           1              IRT
```

Notice a couple things. There's only one subway stop called 'Times Square', but the name is different in the two files. That's one thing, but then there are actually three different stations called '103 Street'. (Only two are shown here.) To tell the difference between them you have to look at subway lines (1, 6, etc.) in one file, and at longitudes in the other. It's a huge pain even identifying the contenders to choose between, and certainly no generic matching algorithm will do it correctly for you. It's such a specialized case that it probably isn't worth developing an algorithm just for this data - or perhaps it's that an 'algorithm' which would solve this problem would be essentially identical to just doing the matching by hand anyway.

This kind of merge difficulty comes up all the time while getting data munged into a usable state. In this example, I just want to attach the turnstile data to the geo data, and most of the matches aren't terribly hard, but it's hard enough that it won't happen automatically, and it would be a super big hassle to do by hand, grepping or control-f'ing around the files to build a merge table. More than once I've wished there was a tool that would help me do this kind of thing quickly.

One way to improve the situation going forward is to have everyone in the world implement unique IDs and/or controlled vocabularies for absolutely everything, and enforce their usage through input validation and strong checking of everything at every point in data processes. That would be nice. I think some messy data situations may persist.

There are a couple special cases of this matching problem that have received a lot of attention:

Geocoding is essentially doing messy matching on strings that represent addresses in order to figure out a unique match with associated geo-coordinates.  These can be found in various GIS software packages, and New York City makes available <a href="http://www.nyc.gov/html/dcp/html/bytes/applbyte.shtml#geocoding_application">such a tool</a> specifically for NYC addresses.

Record linkage in health care and education seeks to match up humans records using name, date of birth, and so on. My colleagues at DOE and I worked (and they continue to work) on the education case, for example matching college outcome data to DOE internal student ID numbers that have associated test scores. <a href="http://the-link-king.com/">The Link King</a> is a pretty complete human-matching tool birthed in health care but possibly more broadly applicable. (It is itself free but runs on top of SAS, which isn't.)

I don't particularly care about address matching or human matching, for two reasons. First, they won't help me with the subway stations. I want a general-purpose tool. Domain-specific rules are certainly useful in specific domains, but I don't like them much in general. Second, in both cases the tools achieve some match percentage which I don't find acceptable. I want to achieve 100% matched data. I want a tool that will make it easy for me to personally review any case that isn't 100% certain. (The Link King has some functionality for this, but it's super specific to human-identifying data. Also, it runs on SAS, which is gross.)

There are more general tools which compare strings for similarity. They all seem to be based on <a href="http://en.wikipedia.org/wiki/Soundex">Soundex</a> or <a href="http://en.wikipedia.org/wiki/Levenshtein_distance">Levenshtein edit distance</a> or something like these. Python has <a href="http://docs.python.org/2/library/difflib.html">difflib</a>, there's <a href="https://github.com/joshaven/string_score">string_score</a> for JavaScript, etc. These are a good start, I think. SeatGeek has their <a href="http://seatgeek.com/blog/dev/fuzzywuzzy-fuzzy-string-matching-in-python">FuzzyWuzzy</a>, which extends basic string comparisons to work better for common cases while still remaining fairly general.

I want to be super OCD with these merges though - I don't want to just hope that the computer is doing a good job matching. So what I really want is not just the computer to find a likely match, but to let me confirm or correct the matches as it goes. I want not just an algorithm but an interface. In the case of the 103rd St subway stop, it should show me that there are three good possible matches and let me work it out - and it should make the whole operation as frictionless as possible.

I'm imagining this as a JavaScript/web app, probably running entirely client-side. You give it two lists of values that you'd like to be able to merge on. For example,

```html
mergevals1
cow
Ducks
```

and

```html
mergevals2
duckies
Cow
```

For each value in the first list, the interface shows you a list of options from the second list, ordered by apparent likeliness of being a match, based on string comparisons of some kind. So you would quickly click on "Cow" to match with "cow" and perhaps agree more slowly that "duckies" is a match for "Ducks" and perhaps specify new common names for both matches (it could just use the first list's version by default as well). The interface would then produce these:

```html
mergevals1  common
cow         cow
Ducks       duck
```

and

```html
mergevals2  common
duckies     duck
Cow         cow
```

Then you can merge `common` onto your first data set, and also onto your second data set, and then you can merge both your data sets by `common`.

(In the subway case, you would pass in a combined column with station name and lines on the one side, and station name and geo-coordinates on the other.)

The process would still have a lot of human review, but the annoying parts are sped up by the computer so that the human user is just doing the decision-making. I think this would make a lot of important and useful data merges way more feasible and therefore lead to more cool results.

Maybe I'll make this.


*This post was originally hosted [elsewhere](https://planspacedotorg.wordpress.com/2013/03/20/human-computer-cooperation-for-data-merging/).*
