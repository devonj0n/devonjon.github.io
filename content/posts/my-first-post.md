---
title: "Rust: Serializing Strings With Serde"
date: 2020-09-15T11:30:03+00:00
# weight: 1
aliases: ["/latest"]
tags: ["rust", "serde"]
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "More specifically: is serializing &str faster than serializing String"
canonicalURL: "https://devonjon.co/1"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "/sped.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "sanic & knuckles know about sped" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

String slices are primitives in rust. However, Strings are put on the heap.
At what point do we start benefiting from serializing string slices instead of strings?

It makes the code more ugle and there's more to manage (life times)

but if you have a huge Payload, is the trade off worth it? 

probably not

```Rust
#[derive(Serialize, Deserialize, Debug)]
struct Information<'a> {
    #[serde(borrow)]
    results: Res<'a>,
    status: &'a str,
}

#[derive(Serialize, Deserialize, Debug)]
struct Res<'a> {
    sunrise: &'a str,
    sunset: &'a str,
    solar_noon: &'a str,
    day_length: &'a str,
    civil_twilight_begin: &'a str,
    civil_twilight_end: &'a str,
    nautical_twilight_begin: &'a str,
    nautical_twilight_end: &'a str,
    astronomical_twilight_begin: &'a str,
    astronomical_twilight_end: &'a str,
}
#[derive(Serialize, Deserialize, Debug)]
struct Information2 {
    results: Res2,
    status: String,
}
fn main() {
    let info = Information {
        results: Res {
            sunrise: "7:07:42 AM",
            sunset: "5:56:02 PM",
            solar_noon: "12:31:52 PM",
            day_length: "10:48:20",
            civil_twilight_begin: "6:42:28 AM",
            civil_twilight_end: "6:21:16 PM",
            nautical_twilight_begin: "6:12:02 AM",
            nautical_twilight_end: "6:51:42 PM",
            astronomical_twilight_begin: "5:41:55 AM",
            astronomical_twilight_end: "7:21:49 PM",
        },
        status: "OK",
    };
    let now = SystemTime::now();
    for _ in 1..1000000{
        let json_s = serde_json::to_string(&info).unwrap();
        let _json_i = serde_json::from_str::<Information2>(&json_s).unwrap();
    }
    let end = now.elapsed().unwrap().as_millis();
    dbg!(end);
}
```