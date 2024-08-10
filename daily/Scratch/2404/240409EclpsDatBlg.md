Blog on open data set for eclipse chasers.
# Motivation for open data
At the risk of stating the obvious, the biggest weakness of a data scientist is that they can't practice their craft without high quality data. And creating a high quality dataset isn't exactly trivial. This becomes the most obvious blocker to adding any kind of value via this discipline. Unlike engineering where you can roll up your sleeves and start building on day one, a data scientist can't do much without data.

In a big to medium sized organization, this problem is typically addressed by investing in data engineering first, getting the data flowing so that data scientists can then work on top of it and bring their skills to bear. An important feature of these data sets is that they are not static, but animate. As the business churns, data keeps flowing into the datasets, making them animate and evolving. The data science products built on top of them can then also evolve. This then becomes a positive feedback loop, where once people see the value the data science products bring, it drives further investment in data engineering and collecting even richer data which in turn enables more powerful data science applications and so on.

While this story repeats many times over behind the closed doors of various organizations, I haven't seen it unfold in the realm of open source. On the other hand, many excellent and widely used open source software projects exist. In a sense, the world of open source is lagging behind the corporate world.

I'm not saying that no open source data sets exist. There are many like MNIST (for handwriting recognition). But these were always meant to be static, to be used for benchmarking machine learning models. They are like statues, frozen in time. Beautiful statues but still statues.

What I have in mind are animate, living and breathing open data sets. As a hypothetical example, imagine there was an open database where every time anyone went grocery shopping, an entry was logged with each item they purchased, its price, the grocery outlet and its location, the date of purchase, etc. A data science application on top of this could be a recommender system that tells people where to shop given their grocery list under certain constraints (like how far the person is willing to drive) so that they get what they need with the least spending.

I could spend a lot more time talking about this vision for open data sets but its much better to show via a working example. We could use the grocery example above, but grocery's are far too boring for a proof of concept demonstration. We need something with more buzz. At the time of writing this in April 2024, nothing has more buzz than the total solar eclipse that just hit North America. And for the most part, the event delivered, with weather cooperating in some of the major locations like Dallas.

And it so happens that the eclipse is perfect for demonstrating how this idea will work. It is very amenable to structured data, it has an amazing community behind it that are willing to share information (read data) about their unique experiences and would watch such a data set like hawks and find creative ways to use it.

If you're not an eclipse enthusiast, please bear with me as I lay out some context that is going to be helpful when I introduce my first open, breathing database.

# The Eclipse
Roughly once a month, we get a new moon. This means the moon is on the same side of the Earth as the Sun. And since the moon doesn't emit any light of its own, only reflects what the Sun emits, you can't see a new moon. All the light it reflects travels away from us.

There is one very spectacular exception to this rule. When the moon is on the same side, and also on the same line as that between the Earth and the Sun. Now, it starts actively blocking some of the light of the Sun. While this might make the Sun feel noticeably cooler, it'll still be too bright to see the Moon with the naked eye. Unless the perfect conditions arise for the Moon to totally cover the Sun and you happen to be in a specific patch of circular land at the right time. Then, you'll see with your naked eye the pitch black outline of the Moon, surrounded by a thin, shimmering ring of light. This is the only kind of new Moon you'll comfortably see without any special aid for your eye (all other times, don't try to see it without eye protection; it could damage your eye). But you'll see it by the light it blocks, not the light it reflects. And this is an awe inspiring, impressive sight, an experience that simply cannot be replicated.

Every few years, a total solar eclipse touches some of our major cities and some percentage of the people who witness it for the first time develop a resolve to witness it again and again. This is how "eclipse chasers" are born. This might sound like some cult of fanatics, but its really anyone who is interested in seeing another totality. This could range from "I'll only see it if the totality shadow comes within a 7 minute drive of me" to "I'll take a week off and travel anywhere in the world to see it". On the one hand, total eclipses are highly predictable. We know the precise times and locations of everyone of them for the next tens of thousands of years (at least). On the other hand, variables like the weather (you won't see the totality if clouds are covering it), economics (prices of everything tend to rise during an eclipse), different kinds of totalities (totalities near the horizon present a different experience from the more common ones high up in the sky) make planning a trip challenging and an endeavor that'll certainly benefit from data. Now that we've set all the context, we can finally dig into the data I've been hyping.

# The data so far
## Totality data
The first table in the database has one row per city where a totality is going to happen. Naturally, some rows will pertain to past totalities as time moves beyond them. For example, a few cities that experienced the 2024 totality are also present. A screenshot of the data in its current form (22 rows) is shown below.
![[Screenshot 2024-04-10 at 7.40.27 PM.png#invert]]
Let me describe the columns:
1) Date: The date on which the totality will happen. People planning a trip during totality can know one what day they need to be in that city. Note that this column can be used as a unique identifier of the eclipse since two eclipses can never happen on the same day. Type: datetime.
2) City: The city where the totality is happening. Even if only a part of the city is going to be inside the totality shadow, it qualifies. Once in the city, its relatively easy to move around and get to the right spot. The date and city together uniquely identify a record. Type: string.
3) State: The state in which the city lies (example: Dallas lies in Texas). Type: string.
4) Country: Self explanatory. Type: string.
5) Comments: Some subjective comments about the event. For example, some cities are listed as having the location of the longest totality in that specific eclipse month. Type: string.
6) Reference: The source for the data. For instance, I found an article detailing the 2026 and 2027 eclipses in Spain and schematized a bunch of the information there. I'm hoping others can do the same as they find articles online since most data on the different totalities tends to appear in this unstructured form online. Type: string.
7) TotalitySecs: The duration of the totality in seconds. Type: integer.
8) TotalityStart: The start time of the totality. Type: datetime.
9) TotalityEnd: The end time of the totality. Type: datetime.
10) ClearProbability: The subjective probability the sky will be clear at the time of the totality (at the time of data entry). This is the subjective opinion of the person entering the data, supposed to be grounded in various weather models. For totalities in the past, we already know if it was clear enough or not. And this number should ideally be updated once that happens. Type: integer.
11) WeatherScore: Another subjective score. On a scale of 1 to 10, how cooperative the weather is. Note that this is distinct from weather or not the totality will be visible. A great example is Dallas 2024. Most weather models were showing a mostly cloudy sky at the time of the totality. And indeed, on the day of the eclipse, it was overcast all morning. And then, around the time of the totality (about 2:50 PM), the clouds magically cleared and people were able to see it. So even though it worked out in the end and the probability of seeing the totality is now 100%, it could have played out differently far too easily. For that reason, I gave it a weather score of 6 out of 10. Type: integer.
12) DateLogged: The date on which the entry was logged. Type: datetime
13) AddedBy: The Github username of the person logging the entry. We'll get into why Github later. Type: string.
## Expense data
Another factor eclipse chasers have to contend with is demand inflation. On the one hand, we want to share this unique experience with others and spread awareness about it. On the other, demand for everything from flights to hotels to cars increases considerably during such events (and so do the prices) and this demand will get worse if more people know about it. In extreme cases, the thing you want to book might even become completely unavailable and this could make the whole trip infeasible. The thing I learnt is that rental cars are the most susceptible to this kind of inflation and hotel stays are the least.

I decided to share data on all of my bookings for the two totalities I've witnessed (2017 and 2024) in this second table, Eclipse_expenses. Its rows are all the bookings I made that were necessary for the trips, when I made the bookings, the prices I paid, etc.

![[Screenshot 2024-04-10 at 10.16.58 PM.png#invert]]

The columns are described below:
1) EclipseDate: The date of the total solar eclipse the booking was made for. This uniquely identifies the eclipse since two can never happen on the same day. Type: datetime.
2) Location: The location where the booking originated. Type: string.
3) Item: Description of the booking. Type: string.
4) Price: Price paid for the booking. Type: float.
5) Currency: The currency in which the price was paid. Type: string.
6) Vendor: Name of the vendor who provided the service. Type: string.
7) Comment: Any comments on the booking. Type: string.
8) BookingDate: When the booking was made. This will give an idea of how far in advance bookings are typically made and how that affects prices. Type: datetime.
9) StartTime: When the booking started. Type: datetime.
10) EndTime: When the booking ended. Type: datetime.
11) DataLogged: When the data was logged. Type: datetime.
12) AddedBy: The Github username of the person adding the entries. Type: string.

While this data has the potential to help eclipse chasers a lot, one risk with making this kind of data openly accessible is that vendors can also look at this data and use it to strategically inflate prices, making the life of eclipse chasers harder.
## Call to action
The data in both of these tables is currently just a handful of records each. It'll be very useful if people provide suggestions on what other columns can be added to the schema, what other schematized tables will be useful for eclipse chasers, etc. And this is a good segue into the data collection process.

# Data collection process
We need to make the data collection process easy to get started with and work with for anyone, even if they're not technical. This will help cast the net wider and collect more high quality data. Let's describe a few features of the data collection process to enable this.
## Format
We could have chosen from a plethora of formats to log the data. CSV, or flat, comma separated files is the most logical for the following reasons:

1. Ease of use: we'd like just about anyone to be able to quickly log data with a minimal learning curve. The CSV format is understood by all spreadsheet applications (Microsoft Excel, Google and Apple sheets, etc.), is pretty straightforward and can be edited even as text files.
2. Universality: Most data processing libraries (like Pandas in Python), applications like SQL server, etc. have provisions to read CSV data. It is easy to quickly write custom applications that process CSV data as well.

Since our files are supposed to be comma separated, the content of the data itself shouldn't have any commas. I mention this because Microsoft Excel does allow commas in CSVs.

## Curation
All data isn't created equal. There is always a subjective trade-off between volume and quality. So, we need some curation to ensure we're being deliberate and consistent in this subjective trade-off. 

There is already a tool that helps curate large groups of humans entering text. That tool is called Git. Its best known for curating code, but no reason it can't also be used for curating manually entered data like this.

To add new data, modify existing data or change the schema of the tables in any of the databases, a contributor would need to cut a new branch from the main branch, make the additions or modifications and then start a pull request for merging the changes into the main branch. The owners of the repository will review the pull request, make suggestions as needed and finally merge the changes in when all comments are resolved.

# Querying and applications
In the near future, I'll add some scripts and instructions that take these CSV's and upload them to a structured SQL database. Various queries can then be run using SQL on the data and data science applications built on top. For instance, a table around flight and car prices people have been seeing on various websites could be combined with weather forecast models to create an application that recommends the best cities to fly to and still have a good chance of seeing the totality.

# Conclusion
The eclipse database was just an example. We can imagine all kinds of living, breathing datasets to which people around the world contribute on a regular basis. This will unlock use cases, and get a positive feedback loop going.

________________
# Linked pages
[diary page](obsidian://open?vault=diary&file=daily%2F2404%2F240409)
