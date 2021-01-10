---
layout: post
title: "Migrate from Tumblr to Github Pages"
date: 2021-01-09 23:28:49 GMT
tags: technology blog go golang
description: "Using Golang to migrate content from Tumblr to Github Pages"
---

Recently, I've decided to move away from Tumblr. I choose Github Pages, mostly because [Jekyll](https://jekyllrb.com). 

Although Jekyll provides a tool to [import](https://import.jekyllrb.com/docs/tumblr/) content from Tumblr, I decided to implement one on my own using Go. 

For starters, Tumblr expose a simple API endpoint to extract content for a given user, supporting pagination and filtering. In its most simple use, you just need to `GET`  from `https://[username].tumblr.com/api/read/` to get an XML with the aggregated posts. In go, we can retrieve a resource with the `http` library

```
response, err := http.Get(url)
```

Where `url` is a type `string`. We can hardcore the url, and it would be fine, but we can also leverage the `net/url` framework to build our request.

```
func produceURL() string {
	params := map[string]string{
		"num":    "50",
		"type":   "text",
		"filter": "none",
	}
	return getURL("https://iamvolonbolon.tumblr.com/", "api/read/", params)
}

func getURL(base string, path string, params map[string]string) string {
	b, e := url.Parse(base) // Let's insatiate the URL
	if e != nil {
		return ""
	}
	b.Path += path // Add the corresponding path
	p := url.Values{} // And transform our slice of strings into an encoded set of params
	for k, v := range params {
		p.Add(k, v)
	}
	b.RawQuery = p.Encode()

	return b.String()
}
```

Now, that we have our url appropriately compose, we can call the API. 

```
func getXMLFrom(url string) ([]byte, error) {
	r, e := http.Get(url)

	if e != nil {
		return []byte{}, fmt.Errorf("Error retrieving %v", url)
	}

	defer r.Body.Close()

	if r.StatusCode != http.StatusOK { // Checking if the response code is 200
		return []byte{}, fmt.Errorf("Status error: %v", r.StatusCode)
	}

	d, pe := ioutil.ReadAll(r.Body)
	if pe != nil {
		return []byte{}, fmt.Errorf("Error processing response: %v", pe)
	}

	return d, nil // Returns a byte slice with the response body
}
```

In order to parse the response xml into native data types, we need to define a set of structs. The xml structure is [documented](https://www.tumblr.com/docs/en/api/v1) 

```
<tumblr version="1.0">
    <tumblelog ... >
        ...
    </tumblelog>
    <posts>
        <post type="regular" ... >
            <regular-title>...</regular-title>
            <regular-body>...</regular-body>
        </post>
        ...
    </posts>
</tumblr>
```

We will need three `structs`

```
type post struct {
	DateGmt      string   `xml:"date-gmt,attr"`
	Date         string   `xml:"date,attr"`
	Slug         string   `xml:"slug,attr"`
	RegularTitle string   `xml:"regular-title"`
	RegularBody  string   `xml:"regular-body"`
	Tag          []string `xml:"tag"`
}

type posts struct {
	Post []post `xml:"post"`
}

type tumblr struct {
	Posts posts `xml:"posts"`
}
```

Once defined the structs, `encoding/xml` can handle the parsing: 

```
var d tumblr
xml.Unmarshal(xmlBytes, &d)
``` 

I've defined a couple of methods to format the `post` structure as required by Jekyll. 

```
func (p post) formatHeader() string {
	str := "---\n"
	str += "layout: post\n"
	str += "title: \"" + p.RegularTitle + "\"\n"
	str += "date: " + p.DateGmt + "\n"
	str += "tags: " + strings.Join(p.Tag, " ") + "\n"
	str += "---\n"

	return str
}

func (p post) formatPost() string {
	header := p.formatHeader()
	body := p.RegularBody

	str := header + "\n" + body

	return str
}
```

Now, it is just a matter of looping through the `[]post` slice. 

```
	for i := 0; i < len(d.Posts.Post); i++ {
		post := d.Posts.Post[i]

		date := strings.Split(post.DateGmt, " ")[0]
		fn := date + "-" + post.Slug + ".markdown"

		p := post.formatPost()
		err := ioutil.WriteFile(fn, []byte(p), 0644)
		if err != nil {
			os.Exit(2)
		}
	}
```