---
layout: page
title: london underground performance
---

<p class="message">
  A long running personal research study into the <a href="http://www.tfl.gov.uk">TfL's</a> <a href="http://www.tfl.gov.uk/info-for/open-data-users/">TracketNet API</a> using my Python package (<a href="https://github.com/adamgilman/tube-python">tube-python</a>) in determining a better method of the performance of the LU (London Underground).
</p>

<img class="parafloat" src="/public/images/underground/tfl-status-small.png" width="200">

The <a href="http://www.tfl.gov.uk">TfL's</a> only provides basic information regarding the performance of it's train network on each line: e.g.: Good Service, Minor Delays, Part Suspended, etc. These details are sufficient to make basic judgements on transport options but, really provides very little transparency in how performance varies from day to day or even hour by hour. <br>

This project's aim is to get an accurate representation of the tph (trains per hour) each line actually achieves on a daily basis and provide real time information related to the current performance of not just each line but, each segment of each line between parts of the networks which can run independently of each other.

The high level plan breaks down into the following components

* Start with a singular line for an MVP (Victoria due to simplicity)
* [Identify all of the measurement points (TrackCodes) along a line](/underground/trackcodes/)
* Map all measurement points for each line, on each branch, in every direction
* Measure train counts along each point
* Build basic UI for displaying train performance
* Work back across other lines and branches
