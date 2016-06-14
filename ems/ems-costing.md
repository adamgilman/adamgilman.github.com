---
layout: page
title: ems cost analysis
---

<p class="message">
  Note: There are a lot of things to consider outside of price when choosing a transactional email provider. This is only considering cost for informational purposes. Choosing the lowest cost provider is not always the best choice, as is the same for choosing the expensive provider. Do your own research and determine which provider gives you the best functional fit for your application.
</p>
In a new project I was starting I need to do some financial modeling to determine pricing and transactional emails were a large component of the project (mailing lists). Every company I researched had different pricing tiers and bands which made a apples to apples comparison quite difficult. I figured as a nice warm up to the new project, I would write some quick cost calculators and do some basic graphs for some expected volumes to see if I find any surprises.

I did an analysis of the following EMS/TEP products from the [Tools Of The Trade](https://github.com/cjbarber/ToolsOfTheTrade#transactional-email) collection

* [PostMark](https://postmarkapp.com/)
* [Mandrill](http://mandrill.com/)
* [MailGun](http://www.mailgun.com/)
* [AWS SES](https://aws.amazon.com/ses/)
* [SendGrid](http://sendgrid.com/)
* [CritSend](http://www.critsend.com/)
* [Postage](http://postageapp.com/)
* [Sendwithus](https://www.sendwithus.com/)

For each service I want to write a basic interface in Python which I give a number of emails and it returns a cost per month. This will allow me to keep the data gathering, analysis and graphing portion extermely simple.

For example:
{% highlight python %}
>>> awsses.getPrice(numberofemails=23430)
23.87 #made up response
{% endhighlight %}


<img class="parafloat" src="/public/images/ems/cost-calculator.png" width="200" padding="20">

I can get my test cases directly from the webpages of the providers themselves, which makes TDD easier but, finding the strange edge cases will be the fun/difficult part. [Do you want to skip all of the code and go straight to the results and charts?](#results)

# Vendors

## AWS SES

### Tests
<script src="https://gist-it.appspot.com/github/adamgilman/ems-costing/blob/master/tests/tests_awsses.py"></script>
