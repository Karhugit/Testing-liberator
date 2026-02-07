# Testing-liberator
A download repo for people wanting to test the liberator/orac kodi addon

Overall Idea
============

In the great trakt culling there was a lot of noise about lists. People using trakt lists, tmdb lists, copying trakt to tmdb, using mdb, using personal lists in the addon, etc etc. However this is the wrong was to consider the data. What we actually have is not a large group of lists with duplicates and extraneous data, but a data pool or tvshows and movies that are referenced into subsets by lists. So if you start with the data in the pool, lists just become references. I imagine this as a library room, with shelves containing DVDs. You add DVDs to your library as you see them, like in the old days when you would go to blockbuster and just look for stuff that was interesting. All the DVDs are on the shelves and you have a PC in the centre of the room (which is orac) that is your indexer. It knows all of the DVDs in your library and allows you to search and group them in a way that suits you. As you watch them orac records that, orac gets all the details it needs for all DVDs online and updates regularly. A list is simply a collection of items you add to your library, doesn't matter where from or why.
Why is it called liberator and orac? If you know, you know.

Install
=======

Download the two zip files liberator.zip and orac.zip
Extract the files in orac.zip to a folder of your choice, something like orac_server
In kodi go to addons -> install from zip and install the liberator addon using the zip files
Use a command or terminal window to navigate to the orac server directory, and start the server via python ('python run_server.py' in windows or 'python3 run_server.py' in linux). This can be on any machine visible on your network to the kodi machine, or the same machine
The orac server uses it's own log, created in the orac server directory. Open this with an editor such as notepad++ and check through looking for errors. You can tell when it has finished starting as the last two lines will show the garbage collector (GC) logging
	[Orac] 2026-01-17 10:29:24 [INFO] [Orac] **GC** Starting garbage collection for static databases...
	[Orac] 2026-01-17 10:29:24 [INFO] [Orac] **GC** Garbage collection finished in 0.02 seconds.
	
Setup
=====

First we need to tell liberator where orac is. Go to tools -> settings -> orac (first tab). Here you enter the IP of orac - if it is the same machine as you run kodi then it can be left to the default 127.0.0.1 (local host). If orac is running on a different machine then enter the IP address of that machine here. Both machines must be able to see each other over the network, obviously. Future state the orac can run as a service, but not quite yet. 
Second we authenticate to trakt via the liberator client. Liberator sends the information to orac for use. Open the liberator addon you installed and go to tools -> settings -> accounts -> authorize trakt. Do the usual trakt auth here, nothing looks different. After you have done that you can check the orac log for a message that it has received the trakt credentials and saved them. Liberator does nothing with trakt other than this, there is no sync or trakt account options other than auth and revoke.
Third, optionally, we authorise TMDB. If you do have a TMDB account you want to use go to tools -> settings -> accounts -> authorise TMDB. It will offer a QR code or a web page and option to open that web page, TMDB authorisation is a bit complex. Use any method to get to the web page and authorise liberator (note the name of the app here is actually 'karhu'). Liberator does nothing with TMDB other than this.
Fourth lets get our debrid accounts done. These are in the accounts section and follow standard procedures, so far it supports RD, PM and EN. The others I need to work on a bit.

So now we have liberator on kodi and orac as a server. Liberator and orac run as a one to many relationship, which means you can have liberator running on several machines and a single orac server providing service to all of them, in fact this is the best way. Orac is a multi threaded server and also very quick to work, so you should not have any issues, but doing this means all liberator instances see exactly the same data. If you are watching a film on one kodi box, another kodi box would show that film as part watched and if you were to move from one box to another you could pick up directly. This is true for everything, all indexes, all watched status, every liberator gets the same view. The orac DB size will depend entirely on your library, but it should be around the same size as the caching used by other addons. There are no limits for the DB, the indexes (see below), the tagging, other than your hard drive. SQL is plenty fast enough.


My Library
==========

Orac will sync lists to it's internal DB with full details, this means faster widgets and no reliance on trakt or tmdb. The lists that are synced are separated into 'your lists' (i.e. owned by you and always present in the library on the assumption that as you created them you want to regard whats in the lists as in your library), and 'generic lists' (lists provided by trakt or tmdb such as trending). The 'my lists' section contains all of your lists - trakt (watchlist, collection, personal), tmdb (watchlist, personal) together. This is the heart of this architecture - the lists are just indexes or subsets of your library, nothing more. All of the items (movies/tvshows/episodes) are stored in static and dynamic databases, the lists are stored as indexes to this database but are really just a subset.

At this point the architecture is nothing more than what current addons do with caching, just a slightly different DB structure. So what's the point? Well, for that you need to understand the internal library indexing.

Internal Indexing
=================

Given lists are just indexes of the library, you can technically create new indexes with any filters you like to create a subset of your library that is more suitable. Lets take a couple of well known lists by giladg, 'latest releases' and 'latest 4k releases'. Currently you would have two widgets for these lists, one each, and each containing 100 items. You can flick through them hoping to find what you want - or you could create a new index that is more specific. So lets do that. First we will need to put these lists into our library, as they are not there by default. Go into liberator and find the 'lists manager' section. Open this and you have several sections, open the 'my lists' section. This is all lists associated with you, so either your lists or your liked lists. As I have 'liked' both of these lists they are shown here, but the status shows 'not in library' which is the default state. I'll go to both and for each use the context menu to select 'add to library'. Now the listrs shows both as added, and we will need to tell orac to resync. So back out of this, go back to 'My Library' and select the resync option. Orac will now see the two new lists are missing and add both the lists and the items to your library.Now go into the my library section and select Movies -> Indexes -> Create New Index. It opens a selection of items we can use to create our new index. First thing is the lists, so select this and from the lists selection choose the two lists we have just added - this forms the subset we will work with. Now set the other filters as you like - for example I don't watch horror so I'll exclude the horror genre from index. I'll sort by air date descending so the latest appear first, and then 'Save & Exit'. Let's give it a good name, like 'Latest Flicks' and save that. You will drop back to the index menu and your new index will be there - select it and see if the list looks good. If you decide to make changes to the index you can use the context menu to edit it, maybe you want to exclude another genre. Once happy, you can create a widget that uses this index and you will get a combined widget, no duplicates, filtered and sorted as you want. Important to remember that this is an index not a list, it is dynamic. Orac will resync those lists hourly, so as the lists change so the index items will change. 
You don't need to specify any lists, if you leave this then the whole library is used. For example you might want all comedy films from the 90's, in which case you would set the release date >= 01-01-1990, release date <= 01-01-2000, genre comedy, and you would have an index that showed you those. Again these indexes are dynamically created each time the data is fetched for the widget, they are not static. Note that you can use T relative dates which are also dynamically created each time. 'T' is today, and you can specify a number of days before today (i.e. T-30) or a number of days forward (T+30). So if you wanted all movies in your library that were released last month you would create an index that had release date >= T-30, release date <= T-1, any other filters you like, and this index would recalculate the dates every time it was referenced.
Orac loads the static data from a combination of trakt and tmdb, so you should get the best of both worlds.
Tags are included as a filter (see tagging below), so you could just create an index for a certain tag and see all items with that tag.
All internal indexes are stored in orac's database, and so are available to any instance of liberator using that server.

There are internal indexes for movies, tvshows and epsisodes. You would use this for most of the functions currently provided by addons such as calendar, dropped shows etc. For example a list of dropped shows is just a tvshow index where the status is dropped. A calendar is an episode index where the air date is future dated (T+1) and the TV show status is 'in progress'. 

External Indexing
=================

This functions in a similar way to the internal indexing, but is found at the top level of the liberator options next to 'My Library'. It is basically a TMDB discover - TMDB itself uses the discover function to provide a lot of their own lists. As above, you have lots of filters to play with and T relative dates are supported. I use this for TV show premieres - I set the first air date >= T-30, first air date <= T-1, with original language EN and sort by first air date descending. There is also a setting for UI language, which is not actually a filter but will choose the descriptions in the language you select here if available from tmdb. So if you set this to Spanish the list of items provided by this index will have overviews in spanish (if available from tmdb).  You will use this indexer to provide a lot of your lists, it's well worth spending time here creating indexes for the things you need. As with internal indexes you can edit an index via the context menu, and also delete an index. All external indexes are stored in orac's database, and so are available to any instance of liberator using that server.
External indexes perform a discover request to tmdb each time, and so you get everything from tmdb. However you may well want one or all of your external index items to be loaded to your library, for a few reasons. First the widgets will be quicker, second the data will be enriched from trakt and so will be better fidelity, third these items will appear in your internal indexes. This done via the lists manager (also shown below). For external indexes you go into lists manager, select TMDB, and you will see a group of lists including your external indexes marked with 'external_index' as the owner. By default these are not in the library and so will perform a call to TMDB every time, however you can select the list and via the context menu select 'Add to library'. This means that the items found via this index will be loaded to your library, the items will be synced hourly with TMDB, and if you were to access this index (as a widget for example) the results would be retrieved from your library rather than a discover call. Remember to force a resync (see lists manager) after adding something to your library.
Fun fact - trakt identifies 'anime' as a genre while tmdb does not so adding to your library populates the genres from both allowing you to filter on this. Or......you could simply do what trakt does. Trakt assumes everything that is animation and from Japan is anime, that's all it does. So you can create an external index similar to the premieres one I described above but include only animation as a genre and set origin country to Japan, and you will get an anime premieres list.
This discover call gets 100 items for the index, not 20. 

Tagging
=======

This is my implementation of personal lists, but again everything is stored by orac and so is visible to all instances of liberator using that orac server. Tags are free format text fields, you can enter anything as a tag. You can have as many tags as you want, you can tag movies/tvshows/episodes with one or many tags. So lets say you wanted to make sure you noted sci-fi films worth watching as you go along, for later viewing. Select the item (it doesn't have to be in your library at this point) and via the context menu select 'tag this item'. You will have a choice of all the tags you have currently created or you can create a new one. Create a new tag like 'scifi watchlist' and set this. Once an item is tagged it will be added to the library if not already present. You can use the Tags Manager section to list and delete tags, and can use them in internal indexes.

Lists Manager
=============

This is where you can see all of your 'lists', including indexes, and choose which are added to your library. It is separated into 4 sections - My Lists, TMDB lists, Trakt lists, and FlixPatrol lists. 
My Lists - this is all the lists you own, or have liked in the case of trakt, including watchlists and collections. Each shows how many items there are (if the list is in the library) and the library status. For each list you can use the context menu to add the list to the library (if not there) or remove it from the library (if already added). Don't forget to force a resync from the 'My Library' section if you make library content changes.
TMDB Lists - This section holds the generic TMDB lists and also your defined external indexes. If this list seems very low compared to the huge amount of lists provided by TMDB helper for example that's deliberate. Those lists are simply prebuilt discover lists, external indexes as liberator calls them. If you want a list like that use external indexer to create one that suits you, the idea behind this architecture is to reduce the dependency on 'lists' and allow the user to create an ecosystem that suits them.
Trakt Lists - These are generic trakt lists such as 'trending' and 'most favourited', provided in case you like them. For all lists the items will be either fetched from trakt via an API call (if the list is not in your library) or retreived from your library (if it is). Any lists added to the library will be regularly synced as shown below in the syncing section.
FlixPatrol Lists - these are scraped from the web page and added to your library as that's the best way to do it. Top 10 tvshows and movies for each streaming service they cover, synced regularly. I don't use these, but others might.

Tags Manager
============

This is where you can list all of the tags you have entered. Selecting a tag will show you all of the items with that tag, and you can delete the tag from any item. 
You can also delete a tag completely, which will remove the tag from any item and remove the tag from the database. Items only exist in the library if they are included via a list, external index or tag. Removing a tag from an item could mean that it is considered 'orphaned', i.e. it's in the library but not actually part of any list, external index or tagging. If this happens the item will be removed from the library by the garbage collector (see below) which is how the library gets cleaned up. 

Lists CM
========

Items have an 'Orac list manager' as part of their context menu, which allows you to add the item to a list (if possible) or remove an item from a list it is already on. To orac all lists are the same, so the choice of lists covers trakt and tmdb, watchlists/collections/personal. There is no separation between trakt and tmdb, they are just lists. 

Scraping
========

Orac has it's own scrapers, you can select whether you want liberator to use the scrapers or not from the orac tab in the settings. It's an option in case people only want to check a provider like easynews or internal folders.
Orac handles the scrapers intelligently, so there are few options. In fact right now there is only one - strict deduplication. So orac has 10 scrapers in the list, these scrapers are ranked based on a score. That score is a weighted score based on time taken, number of results and quality of results averaged over the scrapes done. Each scraper is regarded as either 'active' or 'inactive', the inactive state is set if the scraper does not return any results (when others do) and handles those times when scrapers go down for a while. When you scrape for sources orac will use the top two ranked providers as primary, scrape those and return the results to liberator. The scores fo those two scrapers are then updated. It then runs the other scrapers in the background to get a score for each and updates the score averaged across the scrapes performed. The idea is that orac uses only the two best scrapers unless they fail and then it has to fall back to trying all of the others. If a scraper is marked inactive then it will not be considered for one of the two primary, even if it has the highest score, until it is marked active by providing results. This will stop that 'pause' in scraping when one scraper is offline for a while, orac tends to reply between 1.5 to 4 seconds every time no matter what happens with scrapers.
By using your scraping to rank the scrapers they will be biased towards what you as a user scrape, rather than a generic idea of health.  
The strict deduplication reduces the sources down to the unique entries. For example, if orac used comet to scrape it might get the same torrent provided by several providers, simply with a suffix indicating the source. So you might have 'GoT S01E01 Edith' and 'GoT S01E01 Edith [tgx]' and 'GoT S01E01 Edith [torr]'. Each are the same torrent from different providers, and you can see in the results that they are identical. If you turn on strict deduplication orac will remove all of the duplicates and return only one (the matching includes quality). This makes it easier to see the choices and select the one you want, however if you treasure a lot of results then turn this off. Remember to watch something you just need one good source, everything else is a waste of your time.
Orac does not cache results ever, each scrape is new just in case somethign better has shown up since last time. It usually takes under 4 seconds so this is better, and I dislike cached results so this is the way it is!

I would turn both on, and select a good source to enjoy your watching. Any more results than you can see on the screen is just taking time away from the point of the exercise, which is to watch something. Liberator will show you how long it took to find your sources at the top of the screen.

Garbage Collection
==================

The garbage collector runs regularly, looking for orphaned items. An item is orphaned if it is not part of a list, external index, or have any tags at which point it will be removed from the library. This is how orac keeps things cleaned up, it doesn't delete as you remove a tag for example, but the GC cleans up regularly.

Syncing
=======

Orac syncs on an hourly basis. The syncing is performed by :
- Trakt lists are checked for updated times. If the list has been updated since the last time then any additions are added to the library, anything removed is deleted if required. 
- TMDB lists are done in the same way, synced to the library
- Shows/Movies updates at TMDB are retreived from TMDB, only the last 100, and if any of the updates are to items in the library then those updates are applied. This mostly affects things like ratings or new episodes.
- Any external indexes or other lists are also parsed and updated if required.


API Orac Calls
==============

Orac runs as a multi-threaded server, listening on port 5555 (ha ha ha ha). Liberator calls it directly via an interprocess comms method, but the call can just as easily be made using a terminal or a simple python script. I'm working on a full apiary, and can add stuff, but for the moment if you know what you are doing (or can use AI) then you could easily create a script that, for example, adds a tmdb list and loads it with a full set of movies in two calls. Will be an ongoing project.

Trakt/TMDB updates
==================

Because orac has a full DB, it does not need either trakt or tmdb to be available. If trakt was gone for a week it's unlikely the user would notice, similar with tmdb, unless you were trying to use generic lists not in your library. Orac uses a method called store-and-forward for it's updates to both systems, this means it places the update in the update queue for processing rather than actually doing anything when getting a message from liberator. Every 5 minutes the queue manager runs and checks the queue, if there are updates to be sent it will attempt to send them. Should the update fail, for example trakt is down, the update is marked as failed and will be tried again later until it goes through. This should shield the system from the regular trakt outages and as the DB is full liberator will always be able to get next episodes etc.

