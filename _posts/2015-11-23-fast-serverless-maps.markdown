---
layout: post
project_title:  "Crime on Campus"
post_title: "Fast, server-less maps"
date:   2015-11-23 00:00:00 -0800
project_url: http://projects.dailycal.org/crime/
short_description: We analyzed five years of police reports to see where and when crime occurs most often.
image: crime.jpg
author: Sahil Chinoy
author_link: http://dailycal.org/author/schinoy
---

The Daily Californian’s new crime map might not look complex, but there’s a lot going on behind the scenes. In creating it, we’ve built the beginnings of a framework for fast filtering in web maps using only static files — a framework we want to refine for our future geospatial visualizations.

We filed a public records request with the University of California Police Department for incident-level crime reports, but the data we received didn't provide a precise latitude and longitude. With only an address, we ended up with many sets of overlapping points, so we aggregated incidents in a particular area: a process known as binning.

To make a browser-based map of our bins, we had a couple of options. We could pre-compute the number of incidents in each bin and send those counts to the browser. Or we could send the bins and incidents separately, match each incident to its bin in the browser, count the number of incidents in each bin and draw the map appropriately.

The advantage of the second approach is the ability to filter what you're looking at without requesting new data. For example, if you're interested in property crime, you could filter down the list of incidents browser-side, then count the incidents and redraw the map. This process can be much faster than sending a request to a server to send over another file with the new counts, and snappy maps are important because they make users more willing to spend time exploring the data.

If we're only looking at violent, property and quality-of-life crime, though, it makes sense to pre-compute the number of incidents in each bin for each of those categories and then send the data over once. But what if we’re interested in violent crime that happens at a specific time on a specific day of the week? Now we have to pre-compute and send over an increasing number of counts, and this system quickly becomes unwieldy. We could have set up a “real” API, but we wanted the final product to be entirely static files (for simplicity and sanity).

However, sending over tens of thousands of incidents seems like a bad idea — the file is going to be huge! Enter Tamper, a New York Times project for serializing categorical data: variables that can take one of several values, unlike numerical data or arbitrary text. By assigning each incident to a type (violent, property, quality-of-life) and a bin, we transformed our source records to a large categorical dataset, one that can be really efficiently compressed.

Our incident data, a JSON file, ended up being 41 KB. We included the category, bin, day of the week, hour of the day, month and year, though we didn’t end up providing controls for most of those filters — they just weren’t interesting for this particular dataset.

Note: Our implementation was somewhat clumsy. Tamper doesn't have a native Python encoder, so we pulled an ugly hack to interface with the JavaScript encoder and store the resulting JSON as a static file.

Now we have the data — all of it — in the browser. But if we want to look at violent crimes at 1 p.m., it seems as though we’d have to check each incident to find the ones we want, a slow process as the dataset gets larger.

PourOver, another New York Times project, handles this sort of filtering efficiently. If you know which filters you're interested in (day of week, category, etc.), it will pre-compute enough queries to make arbitrarily chaining filters very fast. Using PourOver, we can quickly filter incidents, count the number in each bin and color the bins appropriately, without ever touching a server.

This is probably overkill for a map showing fewer than 10 thousand incidents, but we've started experimenting with scaling this approach up to data with hundreds of thousands of points. This map doesn’t even come close to the limits of file size or filtering speed; with this approach, we have the ability to sort through and visualize all the crime in the city of Berkeley for the last decade.

To do that, we want to extract the UCPD-specific parts of this project and refine our loading, binning and packing process. If that's something you're interested in, drop us a line.
