---
title: "Microservices in .NET"
date: 2023-08-30T11:19:30-07:00
draft: false
---

## Review

A week after finishing the [Hacker News Feed](https://ejacobg.com/projects/hacker-news-feed/) project, I had found another book to work through: Christian Horsdal's [
_Microservices in .NET, Second
Edition_](https://www.manning.com/books/microservices-in-net-second-edition) (2021). I was (and still am) interested in learning more about both microservices and the .NET stack, so this book couldn't have come at a better time. The difficulty seemed reasonable, it had some projects I could take a look at, and it even covered deploying to Azure, which I did manage to get working. Overall, the book covered mostly what it said it would, but it felt like it could have done so in a much shorter page count. This book does not cover everything very deeply, and in hindsight, I really should have seen this coming and [heeded my advice from before](https://ejacobg.com/notes/hands-on-software-engineering-with-golang/#:~:text=If%20your%20goal%20was%20to%20learn%20each%20of%20the%20aforementioned%20topics%2C%20you%20may%20be%20better%20served%20reading%20a%20dedicated%20book%20on%20each%20one).

I don't have as many written notes as the [Software Engineering with Golang](https://ejacobg.com/notes/hands-on-software-engineering-with-golang/) book. The main value add for me was the code examples rather than the written content. Honestly, if you just read and understand the README and code from the [repository that I made](https://github.com/ejacobg/microservices-in-dotnet/), then you'll probably absorb like 70% of the book. But this is the notes section of the blog, so here are my main takeaways:

* [Some](https://github.com/ejacobg/microservices-in-dotnet/blob/main/LoyaltyProgram/SpecialOffers/Dockerfile) [example](https://github.com/ejacobg/microservices-in-dotnet/blob/main/LoyaltyProgram/Dockerfile) [Dockerfiles](https://github.com/ejacobg/microservices-in-dotnet/blob/main/ShoppingCart/ShoppingCart/Dockerfile) for building .NET applications.
* [Some](https://github.com/ejacobg/microservices-in-dotnet/blob/main/LoyaltyProgram/loyalty-program.yaml) [Kubernetes](https://github.com/ejacobg/microservices-in-dotnet/blob/main/LoyaltyProgram/SpecialOffers/special-offers.yaml) [manifests](https://github.com/ejacobg/microservices-in-dotnet/blob/main/ShoppingCart/ShoppingCart/shopping-cart.yaml) to consider reusing.
* The idea of using an event feed for publishing microservice actions seems useful.
* Working with [unit](https://github.com/ejacobg/microservices-in-dotnet/tree/main/LoyaltyProgram/UnitTests) and [service](https://github.com/ejacobg/microservices-in-dotnet/tree/main/LoyaltyProgram/ServiceTests) tests.
* I didn't understand most of it, but I did get my Azure Kubernetes Service cluster working without too many issues. I did notice it ate up almost $20 of credit only being deployed for a couple of days, so some optimizations may be needed.

## Reflection/Future Work

In total, it took me ~33 hours spread over 18 (consecutive) days to work through this book cover-to-cover (300 pages) and get most of the code working. This actually represents a faster pace than usual, but that may be because I was already familiar with most of the concepts discussed in the book. If I'm being honest with myself, I think I definitely could have spent these 33 hours in a much more efficient manner.

The words "tutorial hell" spring to mind when I think back on this experience. For the past ~2 years, my learning strategy has more or less revolved around finding a book to read that ideally includes some sort of project that gets built as you go along. I'd take notes and add comments to the code so that I could look back and reference it later if I need to. This strategy has worked well back when I didn't exactly know what I was doing and needed the hand-holding. I've grown a lot since then and have made several projects on my own, yet this need for hand-holding still persists. I recognize that there are times when books like this are useful, but I may have been long past that point for a while. I can't expect my beginner strategy to work anywhere past the beginner stage.

Looking forward, I'll try to come up with more of my own projects rather than follow along with a textbook. If I really need a project to reference, there's a million .NET repositories on GitHub if I need them. If I need something to read, I'll stick to the more popular recommendations like Building Microservices and DDIA, or a reference book if I really need it.
