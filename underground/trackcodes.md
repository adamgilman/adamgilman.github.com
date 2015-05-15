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

We are focusing on the Victoria line initially since, it's the most straightforward of the London Underground lines, literally. With no branches, no complex arrangements and only a few sidings and turn arounds, it's easy to begin our analysis so we aren't bogged down by the complexity of the layout until after we solve the original problem.  

<img src="/public/images/underground/victoria.gif" width="100%">

These TrackCodes are not documented anywhere on any TfL website and have been subject to [FOI requests](https://www.whatdotheyknow.com/request/trackcode_locations) but, have been denied, so we will have to map them manually.

First, I have to discover all of the TrackCodes in order to map them out in the order in which they actually appear. By querying the TrackNet API at a specific frequency, I can see all of the trains on a line and their location over time.

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

<img class="parafloat" src="/public/images/underground/leadingcar-id.png" width="200" padding="20">
{% highlight python %}
3891671 	 TV6725 	 At Platform
3891671 	 TV6761 	 Between Vauxhall and Stockwell
3891671 	 TV6761 	 Between Vauxhall and Stockwell
3891671 	 TV6769 	 At Stockwell
{% endhighlight %}
With my newly collected individual train data, I can follow a single train up and down the line and three important bits of information.

* The unique set of track codes
* The order of the track codes
* A description for each track code
* Identify sidings and turn arounds
* Full trackcode map of the Victoria Line


## Unique set of track codes
A quick loop through all of the files, and all of the lines of the files we can extract every known track code and use a Python set to quickly find the all of the unique instances for the Victoria line.

{% highlight python %}
import glob, pprint as pp
track_codes = set([])
for file in glob.glob("*.dat"):
    with open(file) as f:
        for line in f:
            track_codes.add( line.split("\t")[1] )
print pp.pprint( track_codes )
{% endhighlight %}

{% highlight python %}
set([' TV6024 ',
     ' TV6025 ',
     ' TV6026 ',
     ' TV6027 ',
     ' TV6028_2 ',
     ' TV6029_1 ',
     ' TV6029_2 ',
     ' TV6029_3 ',
     ' TV6031 ',
     ...
     ' TV6825 '])
{% endhighlight %}

## The order of the track codes
A curousy glance at any of the train specific track codes shows that track codes are orderly arranged in a pretty consistent manner, which makes analyzing the middle sections of the line fairly straightforward

{% highlight python %}
3914411 	 TV6425 	 At Euston
3914411 	 TV6437 	 Between Warren Street and Euston
3914411 	 TV6479 	 Between Oxford Circus and Warren Street
3914411 	 TV6521 	 At Oxford Circus
3914411 	 TV6525 	 At Oxford Circus
3914411 	 TV6533 	 Between Oxford Circus and Green Park
3914411 	 TV6579 	 Departed Green Park
3914411 	 TV6591 	 Between Green Park and Victoria
{% endhighlight %}

TV6425, TV6437, TV6479, TV6521, TV6525, TV6533, TV6579, TV6591 between Euston and Victoria increases quite nicely while going Southbound and interestingly are all odd.

Looking at a Northbound train:
{% highlight python %}
6030561 	 TV6624 	 At Victoria
6030561 	 TV6612 	 Between Victoria and Green Park
6030561 	 TV6586 	 Approaching Green Park
6030561 	 TV6580 	 At Green Park
6030561 	 TV6574 	 At Green Park
6030561 	 TV6572 	 Departed Green Park
6030561 	 TV6564 	 Between Green Park and Oxford Circus
6030561 	 TV6560 	 Approaching Oxford Circus
6030561 	 TV6526 	 At Oxford Circus
6030561 	 TV6524 	 At Oxford Circus
6030561 	 TV6522 	 Departed Oxford Circus
6030561 	 TV6524 	 At Oxford Circus
6030561 	 TV6514 	 Between Warren Street and Oxford Circus
6030561 	 TV6474 	 At Warren Street
6030561 	 TV6462 	 Between Euston and Warren Street
6030561 	 TV6472 	 Between Euston and Warren Street
6030561 	 TV6426 	 At Euston
{% endhighlight %}
TV6624, TV6612, TV6586, TV6580, TV6574: we can seem the same linear progression of track codes decreasing and going Northbound are all even. Implying all Northbound track codes are even and Southbound tracks are odd.

Next, looking at the end of the lines, where trains turn NB -> SB and vica versa we see data matching our previous assumptions
{% highlight python %}
3891671 	 TV6769 	 At Stockwell
3891671 	 TV6775 	 At Platform
3891671 	 TV6815 	 Between Stockwell and Brixton
3891671 	 TV6817 	 Between Stockwell and Brixton
3891671 	 TV6824 	 At Brixton Platform 1
3891671 	 TV6784 	 Approaching Stockwell
{% endhighlight %}
TV6769, TV6775, TV6815, TV6817, TV6824, TV6784: We can see the train going SB to the end of the line at Brixton traveling on numerically increasing odd track codes and then turning NB onto an numerically decreasing even track code.

## A description for each track code
Getting all of the descriptions for track codes is very similar to the process of getting the unique ones. Iterate over all of the files, find the unique track codes and extract the descriptions from them. There's a slight hiccup in the data in that some track codes are listed with differing descriptions for an unknown reason for now.
{% highlight python %}
import glob, pprint as pp
code_description = {}
for file in glob.glob("*.dat"):
    with open(file) as f:
        for line in f:
            line = line.split("\t")
            line = [x.strip() for x in line]
            track_code, description = line[1:]
            if code_description.has_key(track_code):
                code_description[track_code].add(description)
            else:
                code_description[track_code] = set([description])
{% endhighlight %}
{% highlight python %}
{'TV6024': set(['At Walthamstow Central']),
 'TV6025': set(['At Walthamstow Central']),
 'TV6026': set(['Between Walthamstow Central and Blackhorse Road']),
 ....
 'TV6824': set(['At Brixton Platform 1']),
 'TV6825': set(['At Brixton Platform 2', 'At Platform'])}
{% endhighlight %}

We can see that for TV6825 there are two different descriptions: 'At Brixton Platform 2' and 'At Platform'. This will probably require some manual cleaning of the data in the future.

## Identify sidings and turn arounds
In looking through track descriptions, there's some oddities which don't follow our already assumed patterns we have seen. There are some track codes with underscores (_) in their name which occur along the line.
{% highlight python %}
TV6281_2 set(['Departed Finsbury Park'])
TV6823_1 set(['Brixton Area'])
TV6317_2 set(['Between Finsbury Park and Highbury & Islington'])
TV6627_2 set(['At Victoria'])
TV6823_2 set(['Brixton Area'])
TV6314_2 set(['Between Highbury & Islington and Finsbury Park'])
TV6029_2 set(['Between Walthamstow Central and Blackhorse Road'])
TV6029_3 set(['Between Walthamstow Central and Blackhorse Road'])
TV6029_1 set(['Between Walthamstow Central and Blackhorse Road'])
TV6821_1 set(['Brixton Area'])
TV6372_2 set(['Between Kings Cross St. Pancras and Highbury & Isl'])
TV6820_3 set(['0'])
TV6616_2 set(['Between Victoria and Green Park'])
TV6228_2 set(['Between Finsbury Park and Seven Sisters'])
TV6171_1 set(['Between Northumberland Park Depot and Seven Sisters'])
TV6171_2 set(['Approaching Seven Sisters'])
TV6615_2 set(['Between Green Park and Victoria'])
TV6365_2 set(['Between Highbury & Islington and Kings Cross St. P'])
TV6028_2 set(['Between Walthamstow Central and Blackhorse Road'])
TV6822_2 set(['Brixton Area'])
TV6465_2 set(['Between Warren Street and Euston'])
TV6820_2 set(['0'])
TV6282_2 set(['Between Highbury & Islington and Finsbury Park'])
TV6464_2 set(['Between Euston and Warren Street'])
TV6634_2 set(['Approaching Victoria'])
{% endhighlight %}
With the exceptions of the unknown "0" descriptions, the rest of the underscore track codes occur at the end of the lines (Brixton, Walthamstow), major stations (Victoria, Kings Cross, Euston), etc. We can assume these are turn around points in the line and also sidings. Trains can presumably change course (NB -> SB, SB -> NB) and originate from on routes.


## Full trackcode map of the Victoria Line

Given that we know

* Southbound is odd
* Northbound is even
* "_" denotes a siding, station rail
* Track codes are ordered numerically

A little bit of data manipulation gives us this estimated mapping of track codes





<tr> <td>TV6123</td> <td>At Tottenham Hale</td> </tr>

<tr> <td>TV6125</td> <td>At Tottenham Hale</td> </tr>

<tr> <td>TV6127</td> <td>Departed Tottenham Hale</td> </tr>

<tr> <td>TV6129</td> <td>Departed Tottenham Hale</td> </tr>

<tr> <td>TV6133</td> <td>Between Tottenham Hale and Seven Sisters</td> </tr>

<tr> <td>TV6165</td> <td>Between Northumberland Park Depot and Seven Sisters</td> </tr>

<tr> <td>TV6171_1</td> <td>Between Northumberland Park Depot and Seven Sisters</td> </tr>

<tr> <td>TV6171_2</td> <td>Approaching Seven Sisters</td> </tr>

<tr> <td>TV6211</td> <td>Between Tottenham Hale and Seven Sisters</td> </tr>

<tr> <td>TV6213</td> <td>Between Tottenham Hale and Seven Sisters</td> </tr>

<tr> <td>TV6217</td> <td>At Seven Sisters Platform 5</td> </tr>

<tr> <td>TV6219</td> <td>At Seven Sisters Platform 5</td> </tr>

<tr> <td>TV6223</td> <td>At Seven Sisters Platform 5</td> </tr>

<tr> <td>TV6225</td> <td>At Seven Sisters Platform 5</td> </tr>

<tr> <td>TV6227</td> <td>Departed Seven Sisters</td> </tr>

<tr> <td>TV6233</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6235</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6237</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6239</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6241</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6255</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6257</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6259</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6261</td> <td>Between Seven Sisters and Finsbury Park</td> </tr>

<tr> <td>TV6265</td> <td>Approaching Finsbury Park</td> </tr>

<tr> <td>TV6273</td> <td>At Finsbury Park</td> </tr>

<tr> <td>TV6275</td> <td>Departed Finsbury Park</td> </tr>

<tr> <td>TV6279</td> <td>Departed Finsbury Park</td> </tr>

<tr> <td>TV6281_2</td> <td>Departed Finsbury Park</td> </tr>

<tr> <td>TV6283</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6307</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6309</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6311</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6313</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6317_2</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6319</td> <td>Between Finsbury Park and Highbury & Islington</td> </tr>

<tr> <td>TV6321</td> <td>Approaching Highbury & Islington</td> </tr>

<tr> <td>TV6323</td> <td>At Highbury & Islington</td> </tr>

<tr> <td>TV6325</td> <td>At Highbury & Islington</td> </tr>

<tr> <td>TV6327</td> <td>Departed Highbury & Islington</td> </tr>

<tr> <td>TV6329</td> <td>Departed Highbury & Islington</td> </tr>

<tr> <td>TV6331</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6333</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6335</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6337</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6339</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6341</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6357</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6359</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6361</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6363</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6365_2</td> <td>Between Highbury & Islington and Kings Cross St. P</td> </tr>

<tr> <td>TV6369</td> <td>At Kings Cross St. Pancras</td> </tr>

<tr> <td>TV6375</td> <td>At Kings Cross St. Pancras</td> </tr>

<tr> <td>TV6377</td> <td>Departed Kings Cross St. Pancras</td> </tr>

<tr> <td>TV6381</td> <td>Between Kings Cross St. Pancras and Euston</td> </tr>

<tr> <td>TV6383</td> <td>Between Kings Cross St. Pancras and Euston</td> </tr>

<tr> <td>TV6387</td> <td>Between Kings Cross St. Pancras and Euston</td> </tr>

<tr> <td>TV6389</td> <td>Approaching Euston</td> </tr>

<tr> <td>TV6423</td> <td>At Euston</td> </tr>

<tr> <td>TV6425</td> <td>At Platform, At Euston</td> </tr>

<tr> <td>TV6427</td> <td>Between Warren Street and Euston</td> </tr>

<tr> <td>TV6429</td> <td>Between Warren Street and Euston</td> </tr>

<tr> <td>TV6435</td> <td>Between Warren Street and Euston</td> </tr>

<tr> <td>TV6437</td> <td>Between Warren Street and Euston</td> </tr>

<tr> <td>TV6465_2</td> <td>Between Warren Street and Euston</td> </tr>

<tr> <td>TV6475</td> <td>At Warren Street, At Platform</td> </tr>

<tr> <td>TV6479</td> <td>Between Oxford Circus and Warren Street</td> </tr>

<tr> <td>TV6481</td> <td>Between Oxford Circus and Warren Street</td> </tr>

<tr> <td>TV6485</td> <td>Between Oxford Circus and Warren Street</td> </tr>

<tr> <td>TV6513</td> <td>Between Oxford Circus and Warren Street</td> </tr>

<tr> <td>TV6515</td> <td>Between Oxford Circus and Warren Street</td> </tr>

<tr> <td>TV6517</td> <td>At Oxford Circus</td> </tr>

<tr> <td>TV6519</td> <td>At Oxford Circus</td> </tr>

<tr> <td>TV6521</td> <td>At Oxford Circus</td> </tr>

<tr> <td>TV6525</td> <td>At Oxford Circus</td> </tr>

<tr> <td>TV6527</td> <td>Between Oxford Circus and Green Park</td> </tr>

<tr> <td>TV6529</td> <td>Between Oxford Circus and Green Park</td> </tr>

<tr> <td>TV6533</td> <td>Between Oxford Circus and Green Park</td> </tr>

<tr> <td>TV6535</td> <td>Between Oxford Circus and Green Park</td> </tr>

<tr> <td>TV6561</td> <td>Between Oxford Circus and Green Park</td> </tr>

<tr> <td>TV6563</td> <td>Approaching Green Park</td> </tr>

<tr> <td>TV6567</td> <td>At Green Park</td> </tr>

<tr> <td>TV6573</td> <td>At Green Park</td> </tr>

<tr> <td>TV6575</td> <td>At Platform, At Green Park</td> </tr>

<tr> <td>TV6577</td> <td>Departed Green Park</td> </tr>

<tr> <td>TV6579</td> <td>Departed Green Park</td> </tr>

<tr> <td>TV6581</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6589</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6591</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6607</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6613</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6615_2</td> <td>Between Green Park and Victoria</td> </tr>

<tr> <td>TV6617</td> <td>At Victoria</td> </tr>

<tr> <td>TV6621</td> <td>At Victoria</td> </tr>

<tr> <td>TV6623</td> <td>At Victoria</td> </tr>

<tr> <td>TV6625</td> <td>At Victoria</td> </tr>

<tr> <td>TV6627_2</td> <td>At Victoria</td> </tr>

<tr> <td>TV6631</td> <td>Departed Victoria</td> </tr>

<tr> <td>TV6635</td> <td>Between Victoria and Pimlico</td> </tr>

<tr> <td>TV6637</td> <td>Between Victoria and Pimlico</td> </tr>

<tr> <td>TV6639</td> <td>Approaching Pimlico</td> </tr>

<tr> <td>TV6673</td> <td>Approaching Pimlico</td> </tr>

<tr> <td>TV6675</td> <td>At Pimlico, At Platform</td> </tr>

<tr> <td>TV6677</td> <td>Departed Pimlico</td> </tr>

<tr> <td>TV6679</td> <td>Departed Pimlico</td> </tr>

<tr> <td>TV6717</td> <td>Between Pimlico and Vauxhall</td> </tr>

<tr> <td>TV6719</td> <td>Approaching Vauxhall</td> </tr>

<tr> <td>TV6721</td> <td>Approaching Vauxhall</td> </tr>

<tr> <td>TV6723</td> <td>At Platform, At Vauxhall</td> </tr>

<tr> <td>TV6725</td> <td>At Platform, At Vauxhall</td> </tr>

<tr> <td>TV6727</td> <td>Departed Vauxhall</td> </tr>

<tr> <td>TV6729</td> <td>Departed Vauxhall</td> </tr>

<tr> <td>TV6731</td> <td>Between Vauxhall and Stockwell</td> </tr>

<tr> <td>TV6761</td> <td>Between Vauxhall and Stockwell</td> </tr>

<tr> <td>TV6763</td> <td>Between Vauxhall and Stockwell</td> </tr>

<tr> <td>TV6765</td> <td>Approaching Stockwell</td> </tr>

<tr> <td>TV6767</td> <td>Approaching Stockwell</td> </tr>

<tr> <td>TV6769</td> <td>At Platform, At Stockwell</td> </tr>

<tr> <td>TV6771</td> <td>At Platform</td> </tr>

<tr> <td>TV6773</td> <td>At Platform</td> </tr>

<tr> <td>TV6775</td> <td>At Platform, At Stockwell</td> </tr>

<tr> <td>TV6777</td> <td>Departed Stockwell</td> </tr>

<tr> <td>TV6779</td> <td>Departed Stockwell</td> </tr>

<tr> <td>TV6781</td> <td>Between Stockwell and Brixton</td> </tr>

<tr> <td>TV6783</td> <td>Between Stockwell and Brixton</td> </tr>

<tr> <td>TV6789</td> <td>Between Stockwell and Brixton</td> </tr>

<tr> <td>TV6815</td> <td>Between Stockwell and Brixton</td> </tr>

<tr> <td>TV6817</td> <td>Between Stockwell and Brixton</td> </tr>

<tr> <td>TV6819</td> <td>Approaching Brixton</td> </tr>

<tr> <td>TV6821_1</td> <td>Brixton Area</td> </tr>

<tr> <td>TV6823_1</td> <td>Brixton Area</td> </tr>

<tr> <td>TV6823_2</td> <td>Brixton Area</td> </tr>

<tr> <td>TV6825</td> <td>At Brixton Platform 2, At Platform</td> </tr>

## Southbound
<table>
  <thead>
    <tr>
      <th>Station</th>
      <th>Track Code</th>
    </tr>
  </thead>
    <tr>
      <td>Walthamstow Central</td>
      <td>
        <table>
          <tr> <td>TV6025</td> <td>At Walthamstow Central</td> </tr>

          <tr> <td>TV6027</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6029_1</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6029_2</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6029_3</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6031</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6033</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>

          <tr> <td>TV6071</td> <td>Between Walthamstow Central and Blackhorse Road</td> </tr>
        </table>
      </td>
    </tr>
    <tr>
      <td>Blackhorse Road</td>
      <td>
        <table>
          <tr> <td>TV6075</td> <td>At Blackhorse Road</td> </tr>

          <tr> <td>TV6077</td> <td>At Platform, At Blackhorse Road</td> </tr>

          <tr> <td>TV6079</td> <td>Left Blackhorse Road</td> </tr>

          <tr> <td>TV6083</td> <td>Between Tottenham Hale and Blackhorse Road</td> </tr>

          <tr> <td>TV6119</td> <td>Between Tottenham Hale and Blackhorse Road</td> </tr>

          <tr> <td>TV6121</td> <td>Approaching Tottenham Hale</td> </tr>
        </table>
      </td>
    </tr>
    <tr>
      <td><i>Northumberland Park Depot</i></td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Tottenham Hale</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Seven Sisters</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Finsbury Park</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Highbury & Islington</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>King's Cross</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Euston</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Warren Street</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Oxford Circus</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Green Park</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Victoria</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Pimlico</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Vauxhall</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Stockwell</td>
      <td>7</td>
      <td>9</td>
    </tr>
    <tr>
      <td>Brixton</td>
      <td>7</td>
      <td>9</td>
    </tr>

</table>
