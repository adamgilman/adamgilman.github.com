---
layout: page
title: trackcodes of the underground
---


<div class="message">
  This is a post in a <a href="/underground/">series</a> determining a better method of the performance of the London Underground.
</div>

As a part of my methodology in determinig the frequency of trains (tph: trains per hour) on the underground, I want to determine the position of trains at any given time in order to establish a frequency count at specific measurement points. I know from experience using the <a href="http://www.tfl.gov.uk">TfL's</a> <a href="http://www.tfl.gov.uk/info-for/open-data-users/">TracketNet API</a> deliver a "TrackCode" reference with every train which is easily accessible with my <a href="https://github.com/adamgilman/tube-python">tube-python</a> package.

{% highlight python %}
>>> from tube import Tube
>>> t = Tube()
#get all trains currently on the Victoria Line
>>> trains = t.getAllTrainsForLine("V")
#get the track code location of the first train
>>> trains[0].track_code
u'TV6818'
{% endhighlight %}

These TrackCodes are not documented anywhere on any TfL website and have been subject to [FOI requests](https://www.whatdotheyknow.com/request/trackcode_locations) but, have been denied, so we will have to map them manually.

First, I have to discover all of the TrackCodes in order to map them out in the order in which they actually appear. By querying the TrackNet API at a specific frequency, I can see the train and it's location and it's motion across the line I'm working on (starting with the Victoria Line)

{% highlight python %}
from tube import Tube
from time import time, sleep
tube = Tube()

while True:
    trains = tube.getAllTrainsForLine("V")
    for t in trains:
        print "%s: %s \t %s \t %s" % (time(), t.leadingcar_id, t.track_code, t.current_location)
    sleep(10)
{% endhighlight %}

{% highlight python %}
1431451332.11: 4174551 	 TV6584 	 Approaching Green Park
1431451332.11: 4138341 	 TV6324 	 At Highbury & Islington
1431451332.11: 4136951 	 TV6789 	 Between Stockwell and Brixton
1431451332.11: 4173121 	 TV6165 	 Between Northumberland Park Depot and Seven Sisters
1431451332.11: 4169901 	 TV6274 	 At Finsbury Park
1431451332.11: 4175091 	 TV6820_1 	 0
1431451332.11: 4171731 	 TV6481 	 Between Oxford Circus and Warren Street
{% endhighlight %}

The first column is the current unix epoch time, then the leading car id, current track code and the TfL description of it's location. To follow a single train, I can modify the code a bit to write out a file for each train (leading car id) and track it's track code location over time.

{% highlight python %}
while True:
    trains = tube.getAllTrainsForLine("V")
    for t in trains:
        #track train
        fn = "%s.dat" % t.leadingcar_id
        f = open(fn, 'a')
        data = "%s \t %s \t %s \n" % (t.leadingcar_id, t.track_code, t.current_location)
        f.write(data)
        f.close()
    sleep(10)
{% endhighlight %}

<img style="float: left" src="/public/images/underground/leadingcar-id.png" width="200">
With my newly collected individual train data
{% highlight python %}
3891671 	 TV6725 	 At Platform
3891671 	 TV6761 	 Between Vauxhall and Stockwell
3891671 	 TV6761 	 Between Vauxhall and Stockwell
3891671 	 TV6769 	 At Stockwell
{% endhighlight %}

* find all unique track codes (set)
* find the order of track codes for each nb/sb  

> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/poole/poole/issues/new">Open an issue.</a>
