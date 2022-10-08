---
title: jquery temlate else 태그
categories:
  - Dev
tags:
  - jquery
  - template
last_modified_at: 2017-08-19T09:00:00-00:00
---

#### Using the {{else}} Template Tag without a parameter

###### Template:

```
<li>     Title: ${Name}.     {{if Languages}}         (Alternative languages: ${Languages}).     {{else}}         (Available only in the original version).     {{/if}} </li> 
```

###### Data:

```
var movies = [     { Name: "Meet Joe Black", Languages: "French" },     { Name: "The Mighty" },     { Name: "City Hunter", Languages: "Mandarin and Cantonese" } ]; 
```

#### Using the {{else}} Template Tag with a parameter

###### Template:

```
<li>     Title: ${Name}.     {{if Languages}}         (Alternative languages: ${Languages}).     {{else Subtitles}}          (Original language only. Subtitles in ${Subtitles}).     {{else}}          (Original version only, without subtitles).     {{/if}} </li> 
```

###### Data:

```
var movies = [     { Name: "Meet Joe Black", Languages: "French", Subtitles: "English" },     { Name: "The Mighty", Subtitles: "French and Spanish" },     { Name: "The Mighty" },     { Name: "City Hunter", Languages: "Mandarin and Cantonese" } ];
```

참고 사이트

http://web.archive.org/web/20120921044917/http://api.jquery.com/template-tag-else/