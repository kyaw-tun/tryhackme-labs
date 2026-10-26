# Searchlight - IMINT

> **TryHackMe Room:** [Searchlight OSINT](https://tryhackme.com/room/searchlightosint)

This write-up covers how I approached the room, step by step. People phone numbers, emails, etc are not included.

## Introduction

A TryHackMe OSINT room focused on IMINT/GEOINT, image and geospatial intelligence. The goal across nine tasks is to geolocate photographs (and one video) and answer context questions about them, using nothing but what is visible in the image plus open sources.

One thing shapes the whole room: the images are deliberately low quality and stripped of useful EXIF data. I checked with `exiftool` and apart from one small hint in Task 2, there was nothing to geolocate from. So this is not a metadata exercise, it is visual analysis first, then reverse image search and Google dorking to confirm. The room's own first lesson puts it well: your first tool is your eyes.

Answers are withheld. This write-up covers how I approached each task, not the solutions, and I have deliberately left out the names of the private individuals the room asks you to identify.

## Toolkit

- Close visual analysis (signage, storefronts, road markings, architecture, fashion, vehicles)
- Google search and dorking
- Reverse image search
- Google Maps, including Satellite, Street View, and Street View historical imagery
- `exiftool` (checked, but the images carried nothing useful)
- `FFmpeg` for extracting frames from the video task (optional)

## The straightforward ones

Task 2 was a warm-up: a street sign named the location outright, and a quick search confirmed the sign is exactly where it appears to be. Interestingly, `exiftool` here returned no GPS data, but a Photoshop "slice group" name hinted at the location, a reminder that metadata can leak context even when it carries no coordinates.

Task 3 had no usable metadata, so I worked from the image: a subway entrance, a recognizable store in the background, and a phone ad that roughly dated the photo, though oddly the pedestrians' fashion read a couple of decades earlier. A reverse image search identified the station (a partial sign down the stairs confirmed it), and Wikipedia answered the context questions about its opening year and platform count.

Task 4 showed a food court with a billboard carrying a short code and a `.ca` web address, enough to narrow down the country before searching. A search confirmed the exact airport. Two small traps: the question asked for the building, and I over-thought it looking for terminal names (both times I did this room), until the answer's length matched the airport's name. And the "which city" question is a catch, the airport is not in the city it is named after, so I checked Google Maps to find the municipality it actually sits in.

## Task 5 — a multi-source investigation

This was the most satisfying. The image showed only a shop sign, photographed from a cafe across the street, and every question was about that cafe, not the shop.

The visual clues did the narrowing: the shop sits on a corner where the road slopes uphill, with a one-way sign directly in front. Searching the shop name returned franchise locations worldwide, so I needed the specific one. Two sources matched the exact scene, the same slope, the same one-way sign: a listing photo on one site and a news photo from another angle on a second. Both named the town in their captions, which gave me the location.

From there it became a cross-referencing exercise on Google Maps. The shop fronts several streets at a corner, but the cafe opposite is on a different street, so I had to read the map carefully rather than assume. Clicking through the cafe's Maps entry gave its contact details and a link to its social page, which carried the email. The owners' names were the hardest part and took me a while both times, mostly because I had to work around the many spoiler write-ups now flooding the results. I eventually found them by following a social post that thanked the operators by first name, then confirmed their full identities through a mapping site's "operator" field. (I am leaving the names out here for the same reason, they are private individuals.) Worth noting: an OpenStreetMap entry held the street, contact details, and operator all in one place, so a cleaner route existed than the one I took.

## Task 6 — reverse image search

A crowded restaurant interior with almost no signage, so reverse image search was the only way in. One search named the restaurant. The follow-up, identifying a magazine editor who had done a feature there, was where I had used a hint the first time. This time I got it quickly with a plain search; I suspect result ranking had shifted a popular article to the top. I avoided the AI-generated answer summaries throughout, since they were full of spoilers.

## Task 7 — the hardest, and a lesson in looking

An unusual sculpture, a reindeer fused with a motorcycle. Reverse image search named it and placed it in a city, and matching the answer length confirmed the name.

The second question, who took the photo, was the real challenge, because it required identifying the exact source of this specific image. I spent a long time on it, gave up, and checked the hint, which pointed to a city tourism site. The frustrating part: I had already visited that site this time around without realizing the answer was on it. Even knowing it was there, searches within the page returned nothing. I eventually found it by opening an "outdoor sculptures" article that looked almost empty, just a background image and a map widget, and activating the map, scrolling to the sculpture, and reading the attribution there. The lesson stuck: the answer had been in front of me, and I had walked past it because I had not interacted with the one element that mattered.

## Task 8 — straightforward geolocation

A well-known allegorical statue in front of a building. I recognized the figure immediately, so the first answer was quick. A search identified the building as a courthouse and the city from a photo caption. For the final question, the building opposite, I used Google Maps: Satellite view was not enough, so I switched to Street View, checked the wrong side of the building first, then found the statue on the other side and simply read off the building facing it.

## Task 9 — geolocating a video, and time-traveling in Street View

The last task was a video shot from a hotel balcony. `FFmpeg` is suggested for pulling frames, but I worked from the footage directly in Google Maps both times.

The first time I did the room, I fixated on a zoomed sign at the end of the clip and nearly missed the obvious: an early frame shows an instantly recognizable landmark that pins the city immediately. (I knew that skyline from childhood, I just was not looking at it.) Lesson repeated from Task 7, look at the whole frame, not the one detail you have latched onto.

Placing the hotel itself was harder this time than last, for a reason I did not expect: Street View showed the area years later, after the building had been demolished and redeveloped, and nearby buildings I had used as color landmarks had been repainted. The scene in the video no longer existed. Once I realized that, I used Street View's historical imagery to roll the date back, navigated until the scene matched, and found the hotel's brand on the building. A further wrinkle: at an intermediate date the site had gone by a different name entirely. The first time, searching that older name was enough to find the hotel. This time I stayed in Street View, and the brand on the facade plus the district name, which appeared all over the map, gave me the full name.

## Reflection

This kind of work has been a pull for me since I first started using the internet. Connecting small clues into a picture is something I used to do just reading people's social media, you can reconstruct a surprising amount of someone's story from what they post without thinking about it. The WYWM OSINT introduction drove the same point home with a case of a journalist retracing a person's movements entirely from their own posts.

These skills matter, finding a missing person, verifying where and when an image was really taken. But the same techniques retrace the steps of someone who never consented to being found, which is why they have to be used carefully. That dual-use reality is worth keeping in mind every time.
