# Layouts, Partials, and Views

### Objectives
*After this lesson, students will be able to:*

- Describe how templates & views work together
- Use partials for static content and rendering dynamic content

### Preparation
*Before this lesson, students should already be able to:*

- Write HTML
- Describe how to yield a template in a layout
- Use Ruby instance variables in templates
- Use params to store data in instance variables

## Views in Rails - Intro (20 mins)

In this lesson we will cover the details of rendering views and best practices for keeping the views DRY.

### Recall Layout from Hapi.js
In Hapi, we have to set up the layout by defining a `layoutPath` property and set `layout` to `true` as part of configuration of the `Vision` module in the server:
```javascript
  server.views({
    engines: {html: require('ejs')},
    path: Path.join(__dirname, 'views/templates'),
    layout: true,
    layoutPath: Path.join(__dirname, 'views/layouts')
  });
```
Next, we need to define a `layout.html` file which contains just the `<head>` tag which includes all css and js files plus an empty `<body>` tag which includes just a `<%- content %>` statement. Then when you use `reply.view('index')`, Hapi will render `views/layouts/layout.html` which includes the template file `views/templates/index.html`. The content of the template is injected into the `content` of the `<%- content %>` statement in `layout.html`.

## Layout in Rails

How layout works in Ruby on Rails is pretty much the same as in Hapi. The only difference is that in Rails, you get everything pretty much for free.

When the app is created, Rails will automatically add a layout `application.html.erb` in `app/views/layouts/application.html.erb`. This layout already contains a ERB `<%= yield %>` statement (similar to the `<%- content %>` in EJS template) and all the links to css and js files in the head part of the html document. You **do not** need to do any additional setup for the layout in Rails as Rails knows where it is as long as you keep the file in the default location.

In short, if a Rails rendered page were a sandwich, the layout would be the bread, and the views would be the meat. The layout is like a shell our views get piped through to spit out a final HTML document:

   ![erb-rendering](https://cloud.githubusercontent.com/assets/25366/8539041/07a72c44-242b-11e5-95bc-1d1275a1307c.png)


## Using views with Rails - Demo (20 mins)

In Rails, the logic for the rendering a view is quite straightforward. Given that every route in Rails will execute a method inside a controller, when the method is executed, Rails will look for:

1. A folder inside view corresponding to the controller's name (folder `posts` for `PostsController`).
2. A file with the method's name and `.html.erb`.

For example , if we call `http://localhost:3000/posts`, Rails will execute the method `index` in the controller `posts` and then, look for a template located in `app/views/posts/index.html.erb`  This works when the method always renders the same template.

In some cases though, you may want to render a template with a different name than the current method. Lets take a look at this action:

```ruby

def create
    @post = Post.new(post_params)

    respond_to do |format|
      if @post.save
        format.html { redirect_to @post, notice: 'Post was successfully created.' }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end
```

Based on the result of `@post.save`, the method will execute either the code in the `if` or in the `else`.  The code `format.html` or `format.json` means that Rails will understand the format asked by the user, html or JSON.  We will look at JSON later in the course, so for the moment, we will only look at the lines starting with `format.html`.

In the `if` case , we can see that the code executed in the block `redirect_to @post, notice: 'Post was successfully created.'` This code will redirect the request to the show method - redirect to the object @post in rails means "go to the method to show only this object".

In the `else` case, the code executed is `render :new` - this means, Rails will show the template `app/views/post/new.html.erb`.  This template uses an instance variable `@post`. In this case, it will use the instance variable defined at the start of the method create.

To sum it up, Rails will, by default, render the template that has the name of the current method in the controller, unless there is a `render`statement in the method that tells Rails to use a different template.

There are different syntaxes for render, and they all do the same action described above; but the rule of thumb is to use the simplest one that makes sense for the code you are writing.

> Note: Review and explain each of the render functions below.

```ruby

render :edit
render action: :edit
render "edit"
render "edit.html.erb"
render action: "edit"
render action: "edit.html.erb"
render "books/edit"
render "books/edit.html.erb"
render template: "books/edit"
render template: "books/edit.html.erb"
render "/path/to/rails/app/views/books/edit"
render "/path/to/rails/app/views/books/edit.html.erb"
render file: "/path/to/rails/app/views/books/edit"
render file: "/path/to/rails/app/views/books/edit.html.erb"

```

## Integrating Layouts - Codealong (30 mins)

Create a new Rails app "views_and_layouts" and scaffold the resource posts:

```bash

rails new views_and_layouts
cd views_and_layouts
rails g scaffold Post title content:text
```

Open the posts controller and look at how each method renders the templates: some of them, like index and show, are abstract because the name of the template is the name of the method, but for some other methods,like create or update, we need to explicitly tell Rails what to do at the end of the method.

#### Different Layouts

By default, Rails will render the layout `application.html.erb` but sometimes, you want to render a template in a different layout.

For instance, if we want to add a layout called `sidebar.html.erb`: to do so, create the layout in `app/views/layouts/`.  Add a file called `sidebar.html.erb` and copy the content of `application.html.erb` into `sidebar.html.erb`. Then, add a h1 tag at the start of the body tag in `sidebar.html.erb`:

```html

<h1>SIDEBAR LAYOUT</h1>

```

This will help us to differentiate the layouts.

In the controller method `index`, add this to the end of the method:
```ruby
render layout: "sidebar"
```


This line will just tell Rails to use the same logic of template rendering, but instead of using the default `application.html.erb`, it will render the template inside `sidebar.html.erb`.


#### Using partials

A best practice is to always keep every template as small as possible. A rule of thumb would be to keep shorter than 50 lines, for the sake of readability.

So for example, if your website has a layout with a top menu, the content and then the footer, this should all be in the layout, as it is rendered on every page.  But if all this html is in one file, the layout will end up being 200+ lines of code.

When this happens, you should think about splitting the template into partials. Partials are just chunks of a template that will be rendered into another one, like children of the layout.

Take this template:

```html
    <!DOCTYPE html>
    <html>
      <head>
        <title></title>
      </head>
      <body>
        <header>
          <h1>My Website</h1>
          <ul>
            <li>Menu 1</li>
            <li>Menu 2</li>
            <li>Menu 3</li>
            <li>Menu 4</li>
          </ul>
        </header>
        <section>
          <%= yield %>
        </section>
        <footer>
          <ul>
            <li>About us</li>
            <li>Team</li>
            <li>Terms and conditions</li>
          </ul>
        </footer>
      </body>
    </html>
```

This layout will soon be too long and unreadable, thus we need to split into different parts:


In ```app/views/application/_header.html.erb```:


```erb
      <header>
        <h1>My Website</h1>
        <ul>
          <li>Menu 1</li>
          <li>Menu 2</li>
          <li>Menu 3</li>
          <li>Menu 4</li>
        </ul>
      </header>
```

In ```app/views/layouts/application.html.erb```:

```erb
      <!DOCTYPE html>
      <html>
      <head>
        <title></title>
      </head>
      <body>

        <section>
          <%= yield %>
        </section>

      </body>
      </html>
```

In ```app/views/application/_footer.html.erb:```

```erb
      <footer>
        <ul>
          <li>About us</li>
          <li>Team</li>
          <li>Terms and conditions</li>
        </ul>
      </footer>
```


Now the layout is back to a normal size, we just need to include the partials and the final result will then be the same than before.

**Every partial needs to have an underscore as the first character - this way Rails knows that it is not a proper template but only a partial that will be included in a template.

Let's now call the partials in the layout:

```erb
      <!DOCTYPE html>
      <html>
      <head>
        <title></title>
      </head>
      <body>
        <%= render "header" %>
        <section>dd
          <%= yield %>
        </section>
        <%= render "footer" %>
      </body>
      </html>
```

Rails will automatically look in the folder `app/views/application/` for a file that is called by the name given to the method `render` with an underscore before it.


##Independent Practice (You Do) (20 mins)


Create a Rails app called `fanzine`, this app will have a controller called `home` and four routes called:

* Homepage
* Info
* About
* redirect_home

You'll have to match these routes with these templates:

* Homepage renders the template "homepage"
* Info renders the template "information"
* About renders the template "about"
* redirect_home redirect the request to "/homepage"


This app only has one layout, but the layout should have a menu with links to the four routes. These links should be inside a partial called "menu" and the file should be in "app/views/application". The menu should appear on every page.


## Conclusion (10 mins)

#### Questions

* Where do we use the method `render` (2 places)
* What is the easiest way to go to the show page of a restful controller from another method in this controller?
* How to render a different layout only for one method?
