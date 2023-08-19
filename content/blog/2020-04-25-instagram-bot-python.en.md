---
template: post
title:  "How to download images from an Instagram profile with Python and web scrapping"
date: 2020-04-25
tags: ["bot", "python", "instagram", "web-scrapping"]
author: Ander Granado
---
Instagram is the social network to showing off. It's probably the social network I use the most. I'm too young to actively use Facebook and too old to even consider creating TikTok (if you see it happening, I give you permission to kill me). I also use twitter, but more as a "news" provider, but I don't post anything on that one.

So all my posts tend to go to Instagram. Over the years I have found that it has become a kind of personal milestone diary. Above the photos from my travels you might see photos from when I got my degree, from when I lived in Madrid and was in a videogame studio as a game programmer, or from good times with friends. Although the quality of the images may not be the best, I like to go there from time to time and see all those trips and moments.

Still, I'm a techie and I know that the only way to keep your information safe is to have a copy of it safe by yourself. So I've been thinking for a while about making a script that would allow me to download all my profile pictures. I could also use this to refresh a bit my Python knowledge and practice some web scraping.

# Web Scrapping with Selenium

Some time ago I tried to implement i, and failed, mainly because I tried to download, parse and search the pages manually. The problem is that a web page does not consist only of a single HTML resource, and doing it by hand implies getting all the resources that make up a page. So I put it aside. That was until the other day, when by chance I found a [video][video-bot] in which someone programmed a bot for Instagram to see the people who had unfollowed him. I didn't care about the followers, but I wanted to see how he did it.

When I saw how he did it I saw that he was using a library called [Selenium][selenium], which is a tool that allows you to automate actions inside a browser, mainly to automate functional test and things like that. For me, libraries like that are basically the holy grail of Web Scrapping, it saves you all the hassle of making the requests, filtering the pages, searching for tags, etc. So I downloaded the WebDriver for Firefox (it's the bridge between the library and the browser), installed the Python library and based on the video I started programming.

# Log in on Instagram

![Instagram login website](/static/images/ig-login.png)

The first thing to do is to log in to the page. To do this, we need to do the following:

1. Get the page. As we are not logged in, the page that will appear will be the login page.
2. Obtain the user and password fields and fill them.
3. Click on the _Log in_ button.

All this, done programmatically in Python looks as follows:

```python
class InstaBot:
    def __init__(self, username, pw):
        self.driver = webdriver.Firefox(executable_path="./geckodriver.exe")
        self.username = username
        self.driver.get("https://instagram.com")
        sleep(2)    
        self.driver.find_element_by_xpath("//input[@name=\"username\"]")\
            .send_keys(username)
        self.driver.find_element_by_xpath("//input[@name=\"password\"]")\
            .send_keys(pw)
        self.driver.find_element_by_xpath('//button[@type="submit"]')\
            .click()
        sleep(4)
        self.driver.find_element_by_xpath("//button[contains(text(), 'Ahora no')]")\
            .click()
        sleep(2)
```

As can be seen, I've put the code in the constructor of a class, which is the class where I'm going to implement everything. It is in the constructor because, whatever you want to do, you always need to login. The `sleep()` is necessary because the page needs time to load.

If you run that, you will see that it is the same process that anyone would do to log into Instagram, but automated. You will even be able to see it live in the isolated browser that opens the WebDriver. I'm not going to dig into each function, as I think the names are pretty self-explanatory and it's easy to understand what each one is doing.

# Get all Instagram posts

![Instagram posts](/static/images/ig-posts.png)

After that, my idea was to go to the profile page, grab each link to each post and from each one grab the URL where the image is located. I've already done it manually, so I know that Instagram images can be grabbed by inspecting the HTML for the URL of the image, the URL of a post is usually something like this, which is in the `<img>` tag nested by several `<div>`:

![URL of the image of an Instagram post](/static/images/g-image-post-url.png)

But before we get to that, we need to grab the links to all the posts, so we can find the image link for each of them. The links of an Instagram post have the format `https://www.instagram.com/p/B--N-oBKdPL/`. To get them, once you get to the profile page, we have to scroll down and look for these links.

My initial idea was to scroll all the way down and then look for all the links. The problem is that the profile page only keeps a certain number of posts loaded in the page and, as you scroll down and load new ones, the previous ones disappear. In my experience, it usually keeps around 30 posts loaded. Therefore, what should be done is to scroll down and get the links. To do this I have created the function `get_pictures_links()`, which contains the following:

```python
def get_pictures_links(self):
    self.driver.find_element_by_xpath("//a[contains(@href,'/{}')]".format(self.username))\
        .click()

    sleep(2)

    links = []

    last_height = self.driver.execute_script("return document.body.scrollHeight")
    while True:
        self.driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        sleep(2)

        links_elements = self.driver.find_elements_by_xpath('//a[contains(@href,"p/")]')

        for elem in links_elements:
            links.append(elem.get_attribute('href'))

        new_height = self.driver.execute_script("return document.body.scrollHeight")
        if new_height == last_height:
            break
        last_height = new_height

    links = list(set(links))
    return links
```

There are several things to note in the above function:
- Scrolling is done using JavaScript. To execute JavaScript code on the web, the `execute_script()` function is used.
- The height counter is used to know when to stop scrolling down. If it remains in the same position when scrolling as in the previous scroll, it stops.
- To search for links to posts, look for the corresponding `<a>` tags. In this case we look for a link tag containing `/p` in the `href` attribute.
  - Once we have obtained the tag or, as in this case, the list of tags that meet the criteria, we obtain the attributes themselves, which are the URLs we want. These are saved in the list.
- Because of the way the grid works, it is normal to end up picking up repeated values, as the tags are unloaded over time, but are kept for several scrolls. Therefore, at the end of the scroll, you do a `list(set(links))` to remove duplicates (go to a set, which cannot contain repeats, and then to a list to leave it as before). Doing that leaves the elements out of order, but in this case it doesn't matter. Of the solutions I found on [StackOverflow][stack-overflow-delete-duplicates], that seemed to me to be the cleanest and most adequate for this case.

# Get the _permalinks_ to the images

With all the posts, the only thing left to do is to open them one by one and look for the images they contain. To do this, I have created the function `get_picture()`, to which I pass each of the links I have previously obtained and look for the image to download it.

```python
def get_picture(self, link):
    self.driver.get(link)
    sleep(2)
    try:
        img_element = self.driver.find_element_by_xpath('//img[contains(@class,"FFVAD")]')
        url = img_element.get_attribute('src')
        time_element = self.driver.find_elements_by_tag_name('time')
        timestamp = time_element[0].get_attribute('datetime')
        if url is not None and timestamp is not None:
            timestamp = timestamp.replace(':', '-')
            timestamp = timestamp.replace('.', '-')
            print(url)
            urllib.request.urlretrieve(url, timestamp + '.jpg')
    except:
        pass
```

Ignoring that I don't deal with the exception (and I should), this is where the magic really happens and where there are the most problems. At the moment the easiest way I've found to get the tag that contains the actual image is to search for it via its class. The names of all the classes used in the HTML of the Instagram website are minimized or obfuscated. But, based on the testing I've been doing, they don't change over time, so I'm using that class name. Once you know how to do it, it's very easy to find the tag with Seleniun and get the URL.

Also, to save the images I also get the timestamp of when the post was published. From there, you just need to download the image with `urllib`.

# Summary

The bot has many parts with room for improvement. So far I haven't talked about posts that contain a video or posts that contain several images. I will improve it over time, but I think that as it stands it is a good way to understand how to do basic web scraping. I'm going to leave the code in a [GitHub repository][github-repo], where you will be able to see these improvements.

# What about the Instagram API?

At this point, perhaps someone is wondering why do all this and not use Instagram's API directly. On the one hand, I know there is a new API, but I don't know it. My goal is to try to do the same thing that this tool does but with this API. The one I know is the previous API, which I think is limited and will soon be obsolete.

Still, the goal of this bot is not to depend on whether Instagram is going to allow you to get those images or not. After all, once you're logged into the app, you can technically grab as many images as you want, so you should be able to do it programmatically as well.

In the end, it's not about whether Instagram will let you do it or not, it's about: if you can, why not?


[video-bot]: https://www.youtube.com/watch?v=d2GBO_QjRlo
[selenium]: https://www.selenium.dev/
[stack-overflow-delete-duplicates]: https://stackoverflow.com/a/7961393
[github-repo]: https://github.com/ander94lakx/InstaBot