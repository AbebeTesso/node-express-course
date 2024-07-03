You continue to work in the `jobs-ejs` repository. Create a branch called `lesson14` for this week’s work.

### Fixing the Security

Passport is using the session cookie to determine if the user is logged in. This creates a security vulnerability called “cross site request forgery” (CSRF). We will demonstrate this.

To see this, clone **[this repository](https://github.com/Code-the-Dream-School/csrf-attack)** into a separate directory, outside of the current `jobs-ejs` folder. Then, within the directory you cloned, install packages with `npm install` and run the app with `node app`. This will start another express application listening on port **4000** of your local machine. This is the attacking code. It could be running anywhere on the Internet — that has nothing to do with the attack.

You should have two browser tabs open, one for localhost:3000, and one for localhost:4000\. The one at localhost:4000 just shows a button that says Click Me! **Don’t click it yet**. Use the `jobs-ejs` application in the 3000 tab to set the secret string to some value. Then close the tab for localhost:3000\. Then open a new tab for localhost:3000\. Then check the value of the secret string. So far so good — it still has the value you set. If you log off, your session is discarded. Try this: Log off. Then click the button in the localhost:4000 tab. Then log back on and view the secret string. It is back to syzygy. Set it to a custom value.

Now, without logging off of jobs-ejs , click the button in the 4000 tab. Then refresh the /secretWord page in `jobs-ejs`. Hey, what happened! (By the way, this attack would succeed even if you closed the 3000 tab entirely.)

You see, the other application sends a request to your application in the context of your browser — and that browser request automatically includes the cookie. So, the application thinks the request comes from a logged on user, and honors it. If the application, as a result of a form post, makes database changes, or even transfers money, the attacker could do that as well.

So, how to fix this? This is the purpose of the host-csrf package you installed at the start of the project. Follow the instructions **[here](https://www.npmjs.com/package/host-csrf#:~:text=The%20csrf%20middleware,Example%3A)** to integrate the package with your application. You will need to change app.js as well as **each of the forms** in your EJS files. You can use `process.env.SESSION_SECRET` as your `cookie-parser` secret. Note that the `app.use` for the CSRF middleware must come _after_ the cookie parser middleware and _after_ the body parser middleware, but _before_ any of the routes. You will see a message logged to the console that the CSRF protection is not secure. That is because you are using HTTP, not HTTPS, so the package is less secure in this case, but you would be using HTTPS in production. As you will see, it stops the attack.

Re-test, first to see that your application still works, and second, to see that the attack no longer works. (A moral: Always log off of sensitive applications before you surf, in case the sensitive application is vulnerable in this way. Also note that it does not help to close the application, as the cookie is still present in the browser. You have to log off to clear the cookie. Even restarting the browser does not suffice.)

Enabling CSRF protection in the project is an _important_ part of this lesson — don’t omit it! By the way, the CSRF attack only works when the credential is in a cookie. It doesn’t work if you use JWTs in the authorization header.  However, as we've seen, to send JWTs in an authorization header, you have to store sensitive data in browser local storage, which is always a bad idea.

### A Couple of Tips

The rest of this lesson shows how to build a dynamic database application with **no client-side JavaScript.** Of course, in real-world applications, you’ll often have client side JavaScript, but this lesson shows that you can do a lot of things without it.

However, it does necessitate some differences in approach. If all you have on the client side is HTML, the client can only send `GET` requests (for links) or `POST` requests (for submitting a form). You can’t send `PUT`, `PATCH`, or `DELETE` operations from HTML — unless you add in some client-side JavaScript. So, in this lesson, all routes are GET and POST routes.

You are going to get a list of job listings and display them in a table. You are also going to enable the user to create a new job listing, edit an existing one, or delete one from the list. As always, a given user can only access the entries they own, and not other people’s. Because you can’t do `PUT`, `PATCH`, or `DELETE`, you’ll do `POST` operations for each of these, giving a different URL for each so that the server knows what to do. **Never add, update, or delete data using a `GET`.** That would introduce security vulnerabilities.

Your table should have columns for each the attributes (company, position, status) of each job listing. In addition, it should have buttons on each row to edit or delete an entry. Editing an entry starts with a `GET` to display a form. Deleting an entry just sends a `POST`. So, you should have routes something like this:

```
GET /jobs (display all the job listings belonging to this user)
POST /jobs (Add a new job listing)
GET /jobs/new (Put up the form to create a new entry)
GET /jobs/edit/:id (Get a particular entry and show it in the edit box)
POST /jobs/update/:id (Update a particular entry)
POST /jobs/delete/:id (Delete an entry)
```

In your table, you’ll have a button for edit and a button for delete. The button for edit should do a GET, so that’s a link. A good way to make a link look like a button is to put the button inside the link, as follows:

```
<a href="/jobs/edit/2093410392"><button type="button>edit</button></a>
```

Of course, the URL in the href should have the actual ID of the entry. The button for delete should do a POST. How do you make a button do a POST? As follows:

```
<form method="POST" action="/jobs/delete/0qw9a09as0d9f" style="display: inline">
  <button>delete</button>
  <input type="hidden" name="_csrf" value="<%= _csrf %>" />
</form>
```

This is really a form masquerading as a button. And, because it’s a form, you have to add the `_csrf` token, or your CSRF protection won’t let the operation through. The `display: inline` allows this to line up on the table row.

Ok, so how to build the table? The `GET` for `"/jobs"` comes in, and your router calls a function in your controller to pull all the job listings for that user from the database into a jobs array (which might be empty). Then the controller function makes the following call:

```javascript
res.render("jobs", { jobs });
```

This render call is going to load and parse` /views/jobs.ejs`, passing the array as a local variable to that template. Now you need to construct the table, using EJS code. It will look something like this:

```
    <h2>Jobs List</h2>
    <table id="jobs-table">
      <tr id="jobs-table-header">
        <th>Company</th>
        <th>Position</th>
        <th>Status</th>
        <th colspan="2"></th>
      </tr>
      <% if (jobs && jobs.length) { %>
        <% jobs.forEach((job) => { %>
          <tr>
            <td><%= job.company %></td>
            <td><%= job.position %></td>
            <td><%= job.status %></td>
            <td><button type="button">edit</button></td>
            <td>button type="button">delete</button></td>
          </tr>
        <% }) %>
      <% } %>
    </table>
```

Of course, you also have `include` statements for the header and footer in this ejs file. You see the conditional JavaScript logic in the EJS brackets `<% %>.` But, the buttons aren’t going to do anything yet. So you need to substitute this for the edit button:

```
<a href="/jobs/edit/<%= job.id %>">
  <button type="button">edit</button>
</a>
```

That puts the actual id of the job listing into the URL. Similarly, for the delete button, you have to build one of those button-only forms described above, and it should have the following as its action attribute:

```
action="/jobs/delete/<%= job.id %>"
```

So that the actual id of the entry to delete is included in the URL on the `POST`. Enough on the tips. Here are the steps to complete the project.

### Steps

1. Create `routes/jobs.js` and `controllers/jobs.js`. The router should have each of the routes previously described, and the controller should have functions to call when each route is invoked. Remember that `req.params` will have the id of the entry to be edited, updated, or deleted. You might want to start with simple `res.send()` operations to make sure each of the routes and controller functions are getting called as expected.
2. In `app.js`, `require` the jobs router, and add an `app.use` statement for it, at an appropriate place in the code. The `app.use` statement might look like:

```javascript
app.use("/jobs", auth, jobs);
```

You need to include the auth middleware in the `app.use`, because these are protected routes and the requester must be a logged on user.

1. Test your routes. You can test the `GET` routes from the browser. For the `POST` routes, you’ll need to use Postman.
2. Create `views/jobs.ejs`. That should have the table described above, plus a button to add a new entry.
3. Create `views/job.ejs` (note the singular form here rather than plural like in step 4). That should have the fields so that you can create an entry. You’ll want to use the same form for adding and editing. When adding, you’ll do `res.render("job", { job: null })`. That will tell `job.ejs` that it is doing an add because there’s no value in the `job` local variable. When editing, you’ll do `res.render("job", { job })`. When a non-null entry is passed to `job.ejs`, then the form knows it is doing an edit, so the fields are populated and the button says update. Note that the action for the form is different for each case. If job is null, then `action="/job"`. But if job is not null, then `action="/job/update/<%= job.id %>` so that the update route is called.
4. Add the necessary Mongo calls to `controllers/jobs.js`. You first require the `models/Job` model, so that you can do `Job.create`, `Job.findOne`, etc. As always, have appropriate error handling. You can use `util/parseValidationErrs.js` to handle validation errors. You can use `flash` to pass error and information messages to the user. Be sure that if you do an edit or update or delete for an entry, that that entry belongs to the active user. Here’s a hint: Passport stores the active user in `req.user`. So you can use `req.user._id` for your `createdBy` value.
5. Add a link to `index.html` for the `/jobs` URL.
6. Try it out!
7. There is one more step. You need to make your application more secure! You should configure the helmet, xss-clean, and express-rate-limit packages, just as you did for Lesson 10\. Then try the application out one more time. CORS is not needed in this case.

### Submitting Your Work

The usual steps apply.