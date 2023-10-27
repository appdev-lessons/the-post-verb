# The POST verb

The purpose of most [HTTP requests](https://learn.firstdraft.com/lessons/98) — i.e., when we click on a link or tap on something — is to _retrieve_ information. E.g. this request retrieves HTML source code that describes Chicago:

```http
GET /wiki/Chicago HTTP/1.1
Host: en.wikipedia.org
```

You can place the above `GET` request in Ruby:

```ruby
require "http"

x = HTTP.get("https://en.wikipedia.org/wiki/Chicago")

puts x.to_s
```
{: .repl #get_wikipedia title="GET wikipedia" points="1"}

Or you can place the above `GET` request by pasting this URL into the address bar of a browser tab:

```
https://en.wikipedia.org/wiki/Chicago
```

Or you can place the above `GET` request by [clicking on this link](https://en.wikipedia.org/wiki/Chicago).

---

But what if the user should be able to send any information they want as part of a request? For example, if we want to perform a Google search for the term `ruby language`?

It turns out that Google will accept any search term in a [query string](https://learn.firstdraft.com/lessons/102-query-strings-and-forms) at the end of the request path `/search`:

```http
GET /search?q=ruby+language HTTP/1.1
Host: www.google.com
```

Notice that the value of the `q` parameter in the query string is what Google uses to perform the search. Since spaces aren't allowed in URLs, Google expects us to use pluses (`+`) in place of spaces.

You can place the above `GET` request by pasting this URL into the address bar of a browser tab:

```
https://www.google.com/search?q=ruby+language
```

Or you can place the above `GET` request by typing `ruby language` into a form that looks like this:

```html
<form action="https://www.google.com/search">
  <label for="search_term">Search term:</label>
  <input id="search_term" type="text" name="q">

  <button>Let me Google that for you</button>	
</form>
```

The above HTML code would produce the following form. Try typing any search term, submitting the form, and look at the address bar to see the `GET` request that was placed:

---

<form action="https://www.google.com/search" target="_blank">
  <label for="search_term">Search term:</label>
  <input id="search_term" type="text" name="q">

  <button>Let me Google that for you</button>	
</form>

### Limitations of query strings

Query strings are a straightforward way to send information right inside the URL, but they have some limitations:

- **Length:** URLs have a maximum length. It varies by client and server, but it's approximately 2,000 characters.

  That means if you, for example, tried to send a new blog post longer than 2,000 characters in a query string, part of it would get cut off (or it might error out altogether).
- **File uploads:** Values in query strings can only be text. This means we can't send a file upload as part of a query string.
- **Security:** Imagine we are writing a form that lets users sign up for an account. Consider the following code and try to guess what will happen when we fill it out and submit it:

```html
<form action="https://learn.firstdraft.com/lessons/115-the-post-verb">

  <label for="email_field">My email:</label>
  <input id="email_field" type="text" name="my_email">
	
  <label for="password_field">My password:</label>
  <input id="password_field" type="password" name="my_password">

  <button>Pretend to sign up</button>	
</form>
```

Here is the rendered form. Try filling it out and submitting it — **but don't enter a real password, because it will be leaked!** How will the password be leaked?

---

<form action="https://learn.firstdraft.com/lessons/115-the-post-verb">
  
  <label for="email_field">My email:</label>
  <input id="email_field" type="text" name="my_email">
	
  <label for="password_field">My password:</label>
  <input id="password_field" type="password" name="my_password">

  <button>Pretend to sign up</button>	
</form>

---

Since the `action` is `https://learn.firstdraft.com/lessons/115-the-post-verb`, you end up right back here on the URL for this lesson. No sign up actually happens; this is just a pretend form.

But you can see that even though we did the right thing and used an `input` of `type` `password` which masks the characters that the user types, after the form is submitted the password shows up in clear text in the address bar, in the query string. Oops!

### Request bodies

Due to these limitations, when the purpose of a request is to _create_ rather than _read_ information, we shouldn't use query strings. Instead, the information should be located in a different part of the request: the **request body**.

- The body of a request is located after the headers, after a blank line.
- There is no length limit to the body of a request.
- We can include files in a request body.
- The information will not be visible in the URL of the request.

However, `GET` requests cannot include a body. Instead, if we need to include a body in the request, then we should use the `POST` verb (HTTP verbs are also called "methods" — unrelated to the Ruby term "method"). An HTTP request with a body looks like this:

```http
POST /users/sign_up HTTP/1.1
Host: www.example.com
Content-Type: application/x-www-form-urlencoded

my_email=alice@example.com&my_password=supersecretpassword
```

- The verb/method in the request line is `POST` rather than `GET`.
- There is an additional header, `Content-Type`, that specifies the format of the body. `application/x-www-form-urlencoded` is the format that HTML forms send.
-  The forms encode the values just like they did for the query string (key-value pairs separated by ampersands), but now the information is located in the body of the request rather than in the URL itself.

A browser will place a `POST` request like the above when a form like the following is submitted:

```html{1:(54-66)}
<form action="https://www.example.com/users/sign_up" method="post">
  <label for="email_field">My email:</label>
  <input id="email_field" type="text" name="my_email">
	
  <label for="password_field">My password:</label>
  <input id="password_field" type="password" name="my_password">

  <button>Pretend to sign up</button>	
</form>
```

There is one key difference between this `<form>` and the earlier one we tried: the `method` attribute.

If we omit the `method` attribute from  a `<form>` element, by default the browser will use `method="get"` (just as it uses `type="text"` by default for `<input>`s).

However, for most forms, specifically ones whose purpose is to _create_ information, it's better to switch the verb to `POST` by explicitly including `method="post"`.

That's it! Now we can build forms that allow users to send us entire blog posts, file uploads, and sensitive uploads in the body of the request.

### Responding to POST requests

In Sinatra, to respond to a `POST` request, we use the method `post()` rather than `get()` in our `app.rb`.

In other words, if we have a form that looks like this on one of our pages:

```html{1:(29-41)}
<form action="/add_results" method="post">
  <label for="first_num_field">Add this:</label>
  <input id="first_num_field" name="first_number">
	
  <label for="second_num_field">to this:</label>
  <input id="second_num_field" name="second_number">

  <button>Add!</button>	
</form>
```

When submitted, the browser would send a request that looks something like this:

```http
POST /add_results HTTP/1.1
Host: ourdomain.com
Content-Type: application/x-www-form-urlencoded

first_number=10&second_number=5
```

In order to match this request, we must use the method `post()` rather than `get()` in our route:

```ruby{1:(1-4)}
post("/add_results") do
  # The code for the action goes here
end
```

The key-value pairs in the body of the request are parsed and placed into the `params` hash under keys determined by the `name`s of the `<input>`s, just like before. So the `params` hash will look like this:

```ruby
{ "first_number" => "10", "second_number" => "5" }
```

And our action would `fetch` them , just like before:

```ruby
# /app.rb

post("/add_results") do
  @first = params.fetch("first_number").to_f
  @second = params.fetch("second_number").to_f

  @result = @first + @second

  erb(:addition_results)
end
```

---

So, this was a long-winded way to say:

- Most HTML `<form>`s should use `method="post"` rather than `method="get"` (the default).
- In our Ruby apps, the routes that receive and process these requests should use `post()` rather than `get()`.
- Everything else stays the same as when we were using `GET`s and query strings.

- Select all that are true:
- We should use a POST on the form and route when we are _reading_ from our database.
  - Not quite, re-read the previous section.
- By default, forms make a GET request.
  - Yes!
- We should use a POST on the form and route when we are _creating_ in our database.
  - Yes!
- GET requests hide the query string from the URL bar.
  - Not quite.
- POST requests hide the query string from the URL bar.
  - Yes! But even though it's hidden, we still access the `params` in exactly the same way as before in our backend.
- GET request query strings have an unlimited length.
  - Not quite, re-read the previous section.
- Making a form submit a POST request is done by adding `method="post"` after the `action` attribute.
  - Yes!
- This is a valid form and route pair and will not cause an error: `<form action="/result" method="post">` and `get("/result" ...)`.
  - This is _not_ valid. POST forms should be paired with POST routes: `post("/result" ...)`, or you will get a "route not found" error.
{: .choose_all #post_quiz title="POST Quiz" points="4" answer="[2,3,5,7]" }

---
