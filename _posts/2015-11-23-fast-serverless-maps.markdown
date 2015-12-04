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

The Daily Californian’s new crime map might not look complex, but there’s a lot going on behind the scenes. Here’s how we built fast filtering in a web map — with only static files — and why we think it’s an interesting direction for our future visualizations.

The data we received from the campus police department didn't provide a precise latitude and longitude, only an address, so we ended up with many sets of overlapping points — all the incidents on the corner of Bancroft Way and Telegraph Avenue, for example, got mapped on top of each other. One way to solve this is to aggregate incidents in a particular area, a process known as binning — it's the same concept behind histograms, which aggregate numerical data in a particular range.

We wanted to make a browser-based map of our bins, and we had a couple options. We could pre-compute the number of incidents in each bin and send those counts to the browser in the "property" field of a GeoJSON file. Or we could send the bins and the incidents separately, match each incident to its bin in the browser, count the number of incidents in each bin, and draw the map appropriately.

The advantage of the second approach is the ability to filter what you're looking at without requesting new data, which significantly improves the responsiveness of the map. For example, if you're interested in property crime, you could filter down the list of incidents (browser-side) to include just that category, then count the number of incidents in each bin and redraw the map. This process can be much faster than sending a request to a server to send over another GeoJSON with the new counts, and snappy maps are important because they make users more willing to spend time exploring the data. Much of the value of crime maps in particular is allowing users to examine trends near their home or work.

Wait — if we're only looking at violent, property and quality-of-life crime, doesn't it make sense to pre-compute the number of incidents in each bin for each of those categories and then send the data over once? Yes it does. But what if you're interested in, say, violent crime that happens at a specific time on a specific day of the week? Now you've got to pre-compute and send over an increasing number of counts, and this system quickly becomes unwieldy. We could have, conceivably, set up a “real” API, but we wanted the final product to be entirely static files (for simplicity and sanity).

Sending over tens of thousands of incidents seems like a bad idea — the file is going to be huge! This problem is mitigated somewhat if you don't send over the latitude and longitude of each incident, just the ID of the bin in which it's located. Still, though, scaling this approach up to bigger datasets seems bound to fail.

Enter Tamper, a New York Times project for serializing categorical data: variables that can take one of several values, unlike numerical data or arbitrary text. By assigning each incident to a type (violent, property, quality-of-life) and a bin, we transformed our source records to a large categorical dataset, one that can be really efficiently compressed.

Our incident data, a JSON file, ended up being 41 KB. We included the category, bin, day of the week, hour of the day, month and year, though we didn’t end up providing controls for most of those filters — they just weren’t interesting for this particular dataset.

Note: Our implementation was somewhat clumsy. Tamper doesn't have a native Python encoder, so we pulled an ugly hack to interface with the JavaScript encoder and store the resulting JSON as a static file.

Now you've got the data — all of it — in the browser. But if you want to, say, look at violent crimes at 1 p.m., you're going to have to loop through all the incidents to filter out the ones you don't want, right?

Nope. PourOver, another New York Times project, handles this sort of filtering efficiently. If you know what filters you're interested in (day of week, category, etc.), it will pre-compute enough queries to make arbitrarily chaining filters very fast. So, we can quickly filter down to the incidents we're interested in, count the number in each bin and color the bins appropriately — all without touching a server.

This is probably overkill for a map showing fewer than 10 thousand incidents, but we've started experimenting with scaling this approach up to data with hundreds of thousands of points. This map doesn’t even come close to the limits of file size or filtering speed; with this approach, we have the ability to sort through and visualize, say, all the crime in the city of Berkeley for the last decade.

To do that, we want to extract the UCPD-specific parts of this project and refine our loading, binning and packing process. If that's something you're interested in, drop us a line.
