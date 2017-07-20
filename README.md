![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Web Scraping with Nokogiri

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

Not all data we might want to use from the Internet comes with a nice JSON API. For sites that don't provide an API, web scraping provides an alternative way to automatically pull their data and still be able to use it in our projects.

### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

* Describe the purpose of web scraping.
* Anticipate some of the potential pitfalls in a web scraping project.  
* Describe the purpose of rake tasks and create rake tasks.
* Use Nokogiri to scrape data from a website and put that data into a database.

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

- Create models with Active Record.  
- Use CSS selectors effectively.
- Create a new Rails app from scratch.

## So, what is scraping?
Web scraping is using a script to pull data off of a website automatically.

Remember when we said:
#### "HTML is for humans, JSON is for robots."
While it's true that JSON is generally written for robots, and HTML is generally written for humans, HTML is still a text document that we can parse in code; it just takes more effort than using a JSON API. In general, if a site provides an API, there's no reason to use web scraping unless there's information only available on their website, not over their API.

> Web scraping is NEVER your first choice of how to get data. It's a fallback for when you can't get your data any other way.

For the purposes of today's exploration of scraping, we'll use [Nokogiri](http://www.nokogiri.org/), a web-scraping library in Ruby that will get us up and running relatively quickly.

Let's live-code a quick example of how scraping works by looking at [a website that I like](http://rainbowsandals.com). A couple of the nice features of Nokogiri that you might notice:

* It just takes one line to get data from a site, and we can use that object forever, which means we're only making one web call per URL we want to scrape.
* It lets us do searches using CSS selectors (elements, #ids, .classes), which we're already familiar with and which will usually be all we need to find the specific elements we need.
* It's flexible in how we reference a single element vs. a set of elements. (Much like jQuery, this is really nice, until it means we misunderstand what we're dealing with at some key point.)

**Check for understanding**: What are some of the potential pitfalls in using web scraping instead of using an API?

<!-- ideas for instructors: undocumented, brittle (and they're not likely to announce changes), not formatted correctly (everything is a string) -->

Now, it's your turn: just like you saw, and possibly using this sample code, take some website with data about products and figure out how to scrape that data and print it out.

This is most interesting if you pick a company that you personally like, but if you need some examples of websites that seemed to work, here are a few:
* http://nike.com
* https://www.carteo-handmade.com
* https://www.mybuqu.com/
* https://www.monoprice.com
* http://www.northcoastbrewing.com/beers/year-round-beers/ (note: this one will require following links to get more data)


<details><summary>Example code inside!</summary>

```rb
doc = Nokogiri::HTML(open('https://www.rainbowsandals.com/products-all/womens/sandals'))
doc.css('.itemListing').each do |listing|
  name = listing.at_css('.itemPictureDescription').text.strip
  pid = listing.at_css('.itemNumber').text.strip.split(' ').last
  p name
  p pid
end
```

</details>

## Where do we actually do this?
For these sort of routine, automated tasks, it makes sense to use rake tasks. You know how we can type things like `rake db:migrate` and know that that encapsulates some small task (migration) that we do as a normal part of development? Wouldn't it be nice to encapsulate this the same way, so we could type something like `rake scrape:sandals` and have it work?!?

Here's a [quick tutorial](http://railsguides.net/how-to-generate-rake-task/) on rake tasks. Create a new rails app, and use that to get a task set up; just have it print something first to prove that it's working. Then, try and take the code that you worked in your console, and put it inside the rake task, to get all the data about your site printing.

## What do we do with all this nice data?
Most of the time, we're doing web scraping to get data that we'll be able to use in a future project or app, and we'll want that data to be accessible in a database. Let's update our live-code to also take the data we're interested and put it in a database.

Once we've finished going through an example together, generate a model & run migrations for yourself, then update your rake task to insert into the database.

### Is this actually how we would use this in real life?
Mostly! There are a couple of changes that I would make to make our code production-ready:
* It would be amazing if we could update the relevant rows of our database, rather than creating a new row every time we run this rake task. Because my site gives me access to product IDs, this is relatively straightforward, but it can be much trickier depending on the data your site gives you access to. (Try checking for data-attributes on your HTML elements to find the product ID you're interested in.)
* For many scraping projects, we don't just want to keep track of the current state of the site, but to track things over time. (Maybe I want to track what happens to those sandals' prices every day.) This often means that we keep track of more data than just the current price, or that we create a new row every time that the price changes, or something.
* To set up this rake task to run every so often (every hour/day/week) we could use a cron job (on our own server or computer), or Heroku's scheduler. Or, if we want to just run it once and then periodically to update it, that works too.
* Because scrapers are brittle, for something that's running automatically, I would also add some sort of alert to me when the scraper fails (an email would be a great option).
