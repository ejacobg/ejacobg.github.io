---
title: "Hacker News Feed"
date: 2023-08-07T16:27:02-07:00
draft: false
---

I've been working with Go for the past ~1.5 years now, and while I would still like to continue working with it in the future, I realize that it isn't exactly the most employable language in my area. Therefore, I've switched gears and started to learn C#. I did have some prior experience with the language and its related technologies through my [senior capstone project](https://unr-cs426-team-07.github.io/) and an elective class I took at UNR. This project was an opportunity to experiment with and refine my learning process. Below I'll discuss mostly everything I learned during the development of this project. You can find the repository for the project on [GitHub](https://github.com/ejacobg/HackerNewsFeed).

## Background

My typical usage of Hacker News looks like this:

1. Go through the first two pages and the "Ask HN" section and open any interesting articles and discussions in a new tab.
2. Read through all the open tabs. Upvote and favorite them as needed.
3. Close any articles/discussions that weren't as interesting as originally expected.
4. Come back later to the tabs that are still open, and see if any discussion has progressed.
5. Repeat from (1).

This workflow works fine for the most part, but I've found it annoying to have to wade through many of the same submissions every time I visit the front page. I know that the "hide" feature exists, but clicking that button adds more work for submissions I never even wanted to interact with in the first place. I'm only interested in at most ~20% of the front page.

The Hacker News Feed acts as an alternative HN frontend specifically tailored to my use case. Specifically, it...

* Merges the first two pages and the "Ask HN" section into one single page.
* Keeps track of the submissions that I've opened.
* Allows me to mark submissions as uninteresting (similar to the "hide" feature).
* Automatically marks unopened submissions as uninteresting (no need to manually click "hide" for each one).
* Displays just the new front page submissions instead of re-showing the ones I'm not interested in.

## Approach

This technically wasn't my first time working with C#, but I haven't used it in a while, so I decided to relearn it from scratch. I did skip over the language basics though since most of the language features were familiar to me. As for learning material, I decided upon Andrew Lock's [_ASP.NET Core in Action, Second Edition_](https://www.manning.com/books/asp-net-core-in-action-second-edition) and read through the first 10 chapters (ASP.NET Core basics), chapter 12 (CRUD operations using EF Core), and chapter 21 (using `IHttpClientFactory`) before I actually started coding. This reading actually took up a decent amount of my time (~15.5 hours over 15 days), but on the whole I think that this was a good decision since my notes were a huge help throughout the project. I just need to learn to read a little faster.

## System Design

Before coding, I identified all the core operations needed for the program and grouped them into several components. I wanted to experiment using Logseq's [whiteboard](https://docs.logseq.com/#/page/whiteboard) feature, and I think this definitely helped.

[![Component diagram](/hacker-news-feed-components.png)](/hacker-news-feed-components.png)

Feed items are pulled from an `IFeedProvider`. The diagram shows two implementations of this interface, but I only implemented the `ApiProvider`. The `FixedProvider` functionality could be provided internally inside an `IFeedService` implementation.

`IFeedService` encapsulates all the operations needed to implement the functionality described above. Like with `IFeedProvider`, I planned out two implementations for a feed service: one that generates and returns fake feed items and stores them in-memory, and another one that pulls data from the Hacker News' [Firebase](https://github.com/HackerNews/API) and [Algolia](https://hn.algolia.com/api) APIs and saves data to a LocalDB instance.

Two controllers were needed to manage the front page of the application and to provide endpoints for viewing and "subscribing" to submitted items.

## Implementation

The core of the project is built around the `Item` class, which looks like this:

```c#
// Item reflects that of a Hacker News item, but typically represents an story, Ask HN, or Show HN post.
[Index(nameof(ItemId), IsUnique = true)]
public class Item
{
    [Key]
    public int Id { get; set; }
    
    // The item's ID as given by Hacker News.
    public int ItemId { get; set; }
    
    public string Title { get; set; }

    // If this item contains a URL, then it will be added here.
    public string Url { get; set; }

    public int Points { get; set; }

    public int Comments { get; set; }

    // Created represents the time this item was submitted to Hacker News, not when it was created in our local database.
    [DataType(DataType.Date)] public DateTime Created { get; set; }

    // Updated represents the time this item was last updated from the API.
    // Changing the subscription status shouldn't affect this timestamp.
    [DataType(DataType.Date)] public DateTime Updated { get; set; }

    // Subscribed is true if the user opened the item or explicitly subscribed to it, false if the user explicitly unsubscribed from it, and null if the user has yet to interact with this item.
    public bool? Subscribed { get; set; }
}
```

The `IFeedProvider` pulls data from the APIs, then converts it into `Item` objects. The `IFeedService` takes these items and saves them to the database. The feed service is also in charge of managing the subscription status of all the feed items - subscribe, unsubscribe, and "clear", which marks all `null`-subscribed items as unsubscribed. A basic frontend was made to show all the feed items in a table:

[![Frontend](https://raw.githubusercontent.com/ejacobg/HackerNewsFeed/main/HackerNewsFeed/Screenshots/Update.png)](https://raw.githubusercontent.com/ejacobg/HackerNewsFeed/main/HackerNewsFeed/Screenshots/Update.png)

## Reflection

According to my logs, this project took around 21 hours to complete, spread across 12 days (not all consecutive). This figure includes both the time spent planning the project and the time spent coding. In hindsight, this project seemed conceptually pretty simple, but there were a lot of details I was missing when trying to implement it, and that took me down several rabbit holes I could have done without. I will admit that I did use ChatGPT (3.5) to help fill in my knowledge gaps, for example how to structure my services and what directories make sense to place them in, providing thread safety using `ReaderWriterLockSlim`, calling JSON APIs, and some frontend code. You can find a copy of my chat here: https://chat.openai.com/share/8230c51f-54c5-4090-880a-1765d5d148b2. Despite this, I still needed to turn to Google and StackOverflow for other issues, mainly related to working with EF Core. That was the only part that wasn't covered very thoroughly in the book.

I've always been the type to do a lot of reading and planning before jumping into a task. Reading and taking notes on everything has given me a lot of value over the years. However, I will admit that this is a time-consuming process, and I tend to only revisit a small portion of the notes that I've taken. This project was an experiment in project-based learning/learning on-demand, and while I wouldn't call it a huge success, I've realized that there's a lot more room to improve. My key takeaways were that my book notes helped, I just needed to find the right sections to take notes on. ChatGPT is good at identifying keywords to look up, but I'm probably better served by reading a book chapter than throwing those keywords into Google and StackOverflow.
