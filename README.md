# Safari Content Blocking - Cocoaheads August 20, 2015

## Why Safari Content Blocking?

All about User Experience. Users want privacy and a better mobile web experience. Current 'Extension' solution used on OS X is resource intensive, relies on Javascript's ```onBeforeLoad``` event, and can do all sorts of tracking and other potentially nefarious work. 

How Safari Content Blocking addresses those potential issues:

* 'Precompiles' rules into Safari via app extensions, avoiding large processing and memory requirements of giant rules and stylesheets used by common extensions, improving battery life and resource allocation
* Rules exist in a simple JSON file that can either block or hide elements
* No executable or scripts run, eliminating tracking or other actions

## Implementation

In Xcode, create a new Application Extension target using the Content Blocker Extension option. Open the blockerList.json file, let's examine the included sample:

```
[
    {
        "action": {
            "type": "block"
        },
        "trigger": {
            "url-filter": "webkit.org/images/icon-gold.png"
        }
    }
]
```

Each dictionary requires an action and a trigger. In this example, we use a url-filter trigger with a block action. This would block Safari from requesting the resource at the given URL. Each rule we set up must have an action and at least one trigger. Here are the available options:

#### Actions

* ```block``` (blocks the request to the asset)
* ```block-cookies``` (blocks cookies at the asset)
* ```css-display-none``` (requires a CSS selector, loads the asset, but hides it using CSS)
* ```ignore-previous-rules``` (overrides rules preceding the current)

#### Triggers

* ```url-filter``` (required, regular expression against full url)
* ```load-type``` (first-party or third-party)
* ```resource-type``` (filters specific types of media, scripts, images, stylesheets, etc)
* ```url-filter-is-case-sensitive``` (true or false)

## A Case Study

Dean Murphy wrote a content blocker and tested it against iMore.com. His before and after:

Before | After
-------|-------
![Before](http://static1.squarespace.com/static/50562b2ee4b02b42cb307607/t/558b4b0ce4b037cdf6064f7b/1435192078901/Space+Grey+%403x+%2B+IMG_8111.png?format=250w) | ![After](http://static1.squarespace.com/static/50562b2ee4b02b42cb307607/t/558b4b00e4b037cdf6064f51/1435192066977/Space+Grey+%403x+%2B+IMG_8111+%2B+IMG_8114.png?format=250w)
With no content blocked, there are 38 3rd party scripts  (scripts not hosted on the host domain) running when the homepage is opened, which takes a total of 11 seconds... network activity was still active after a minute of leaving the page dormant. I decided to turn them all off all 3rd party scripts and see what would happen. | After turning off all 3rd party scripts, the homepage took 2 seconds to load, down from 11 seconds. Also, the network activity stopped as soon as the page loaded so it should be less strain on the battery. 

[Read more](http://murphyapps.co/blog/2015/6/24/an-hour-with-safari-content-blocker-in-ios-9) about Dean's experiment or find his JSON rules on [Github](https://gist.github.com/CraftyDeano/777579c628b3d8d50f25).


## Let's Build It

### Block Third Party Scripts for Better Mobile Web Experience

There's controversy around ad or tracker blocking. Whatever side of the fence you stand on, the implementation for blocking assets is super simple, and the details apply to other content filtering as well.

Let's clean up [The Verge](http://www.theverge.com/2015/7/20/9002721/the-mobile-web-sucks). (No joke, this post on The Verge crashed Safari while I was preparing this README.) We could spend a lot of time refining this, but I bet we get a faster loading experience by simply blocking all third-party scripts.

```
[  
   {  
      "action":{  
         "type":"block"
      },
      "trigger":{  
         "url-filter":".*",
         "resource-type":[  
            "script"
         ],
         "load-type":[  
            "third-party"
         ]
      }
   }
]
```

### Hide the Price on Amazon While Asking Permission

I've been craving the Dell U3415W 34-inch monitor for a few months now. Maybe if I could hide the price so I can give my full pitch when asking my boss (or wife) permission to buy it.

```
[  
   {
       "action": {
           "type": "css-display-none",
           "selector": "div[id*='price']"
       },
       "trigger": {
       "url-filter": ".*"
       }
   }
]
```

### Block Sites by URL Keyword

Primitive, sure. But it can help block anything particularly heinous. Like anything related to the Los Angeles Lakers.

```
[
    {
        "action": {
            "type": "block"
        },
        "trigger": {
        "url-filter": ".*lakers.*"
        }
    }
]
```


## Final Tips

* Your rules are compiled into Safari's master rule set when you activate the extension. If you are working in the code, you either need to deactivate and reactivate the extension, or use ```SFContentBlockerManager.reloadContentBlockerWithIdentifier``` to reload it programmatically.
* The documentation says that this works with the new CSS Level 4 selectors. Just be aware of which of those new selectors Safari supports. I shot 3.5 hours tinkering with the .has() selector before realizing it wasn't actually supported in any browser yet.
* Be aware that this only works in Safari and the new ```SFSafariViewController```.
* And, because this is Apple, be aware that if the system deems your rules as potentially negatively impacting user experience, they won't load.

#### Sources:

* [WWDC Session 511](https://developer.apple.com/videos/wwdc/2015/?id=511)
* [Hacking With Swift](https://www.hackingwithswift.com/safari-content-blocking-ios9)
