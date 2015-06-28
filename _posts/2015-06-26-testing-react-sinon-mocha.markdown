---
layout: post
title:  "Testing Ajax-y React Components with Sinon"
categories: blog
tags: react testing ajax sinon
---

I started using [React][react] a few months ago, for the large-scale redesign
I'm currently working on at my day job. I am really impressed so far and am
enjoying coding within this paradigm. However, as with any new technology, there
is always a learning curve and changes to workflow to consider. I like the
readability (and writeability) afforded by JSX, but using it does add another step
not only to the build process, but also to the testing process.

Impressed by the ease of use and the automatic Mocking of Everything, I began
writing my tests with [Jest][jest]. Jest was super easy to get started with, but
unfortunately, it was slow. Slow as a dog. I liked
[Mocha][mocha] anyway, so I tried out the guide provided by the good people at
[Hammer Lab][hammerlab]. It required a bit of initial setup, but the rewards of
speed and in-browser debugging of tests have proved invaluable. (You can check
out a simplified version of my setup via [my fork over at GitHub][github].)

This setup has worked well for me with the simplest components, but yesterday I
wanted to test a component with AJAX functionality. [Sinon][sinon] is a mocking library
with an excellent `fakeServer` API that is perfectly suited to this sort of task.
This was where I ran into issues with my testing setup again, however.

I've been using [jQuery][jquery] for the AJAX calls and modeling the setting of
data from the server on the examples given in [the tutorial][tutorial]: setting
an initial state and then retrieving the real state from the server via AJAX in
`componentDidMount`. I needed to test that the re-render after the initial AJAX
call was working as expected.

I added Sinon to my `devDependencies` and updated my test case to create an
instance of `fakeServer` before every test, then do a restore afterwards.
Within the test, I told the fakeServer about the type of request it should queue,
rendered my component, and then let the fakeServer respond to the request before
starting to test things.

{% highlight javascript %}
require('./utils/testdom')('<html><body></body></html>');

var should = require('should'),
    sinon  = require('sinon');

var React     = require('react/addons'),
    TestUtils = React.addons.TestUtils;

var Rating = require('../main/webapp/js/components/Rating.jsx');

describe('Rating', function () {

    var server = null;

    beforeEach(function () {
        server = sinon.fakeServer.create();
    });

    afterEach(function () {
        server.restore();
    });

    it('should initialize itself correctly', function () {

        // Set up the fake response
        server.respondWith('GET', '/initURL',
            [200, {'Content-Type': 'application/json'},
                JSON.stringify({allRating: 2, ratingCount: 1})]);

        // Render the component into the document
        var rating = TestUtils.renderIntoDocument(
            <Rating
                initURL="/initURL"
                updateURL="/updateURL"
                />
        );

        // Respond to the queued requests
        server.respond();

        // Verify things
    });

});
{% endhighlight %}

The first AJAX error I started getting from jQuery was `No transport`. This turns
out to be a pretty common problem and is to do with localhost and the [Same-origin policy][sop].
This was fairly easy to rectify by explicitly telling Sinon that the transport supports
[Cross-Origin Resource Sharing][cors].

{% highlight javascript %}
before(function () {
    sinon.xhr.supportsCORS = true;
});
{% endhighlight %}

Now, the error was gone, but Sinon wasn't actually queuing the requests. The
callback that listens for requests wasn't getting triggered.
This required a little digging into the Sinon source code, but there I found that
it sets its listeners on what it finds at `global.XMLHttpRequest`, not `window.XMLHttpRequest`.
Although I had configured a `window` property on the `global` object via `jsdom-no-contextify`
in my `testdom` util script, I hadn't explicitly configured `global.XMLHttpRequest`.
Once I set that property as `window.XMLHttpRequest`, Sinon was able to queue the
requests correctly.

The final issue I encountered was that although Sinon now recognised the requests,
it wasn't matching the URL correctly and so returned a 404 response. If the URL is
defined as a string, Sinon tries to match it exactly, query strings and all. Since
I am using the `{cache: false}` option in my AJAX calls, this appends a
cache-busting query string to the request, causing Sinon to be unable to match it.
I switched my `respondWith` calls to use a regular expression instead of a string,
and finally all was well.

{% highlight javascript %}
server.respondWith('GET', /\/initURL/,
    [200, {'Content-Type': 'application/json'},
        JSON.stringify({allRating: 2, ratingCount: 1})]);
{% endhighlight %}

Despite these gotchas, I can thoroughly recommend Sinon as a great framework for
mocking AJAX calls. My issues are more to do with the environment in which I'm
working. The API itself is really easy to use and works great. Now that I have
my first AJAX-y test case set up, I can easily extend this to any other component
which requires it.


[react]:     https://facebook.github.io/react/
[jest]:      https://github.com/facebook/jest
[mocha]:     http://mochajs.org/
[hammerlab]: http://www.hammerlab.org/2015/02/14/testing-react-web-apps-with-mocha/
[github]:    https://github.com/heldinz/mocha-react
[sinon]:     http://sinonjs.org/
[jquery]:    https://api.jquery.com/jQuery.ajax/
[tutorial]:  https://facebook.github.io/react/docs/tutorial.html
[sop]:       https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy
[cors]:      https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS
