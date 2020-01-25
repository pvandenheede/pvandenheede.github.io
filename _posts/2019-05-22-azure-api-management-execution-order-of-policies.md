---
layout: post
title: Execution order of policies
summary: Here's a blog post which explains you in which order Azure API Management runs its policies.
featured-img: 20190522-policyexecutionorder
comments: true
categories: Azure-API-Management
---

<!-- wp:paragraph -->
<p>I wrote some posts about Azure API management earlier last year, but last week I was discussing some things with a colleague of mine and he asked a question neither of us could answer out of our heads:</p>
<!-- /wp:paragraph -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>When you send a request to Azure API Management, what order is maintained to execute the policies?</p></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>After some searching neither of us could find it, so here's a blog post to save you the trouble after some testing :-)</p>
<!-- /wp:paragraph -->

<!-- wp:more -->
<!--more-->
<!-- /wp:more -->

<!-- wp:heading -->
<h2>Scenario</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>We want to determine the execution order in which Azure API Management executes policies for a request. We consider the following policies are considered here:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Operation policy</li><li>API policy</li><li>All APIs policy</li><li>Product policy</li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2>Set-up</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>To test this specific case, I provisioned an Azure API Management instance within the relatively new <a rel="noreferrer noopener" aria-label="Consumption  (opens in a new tab)" href="https://www.codit.eu/blog/azure-api-management-consumption-tier/" target="_blank">Consumption </a>tier. Using this tier I can just set this instance up in my MSDN subscription with limited Azure credit and still only get billed when I use it... Downside is that I don't have any developer portal, but for testing this is not needed.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I setup a new API "Test" with a free and easy <a href="https://requestbin.fullcontact.com" target="_blank" rel="noreferrer noopener" aria-label="RequestBin (opens in a new tab)">RequestBin</a> back-end:</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":2626} -->
<figure class="wp-block-image"><img src="https://pvandenheede.files.wordpress.com/2019/05/apim-policy-api2.png" alt="" class="wp-image-2626" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I created one operation there called "post" (yes, I sometimes struggle with creativity):</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":2625,"width":538,"height":278} -->
<figure class="wp-block-image is-resized"><img src="https://pvandenheede.files.wordpress.com/2019/05/apim-policy-apibackend.png" alt="" class="wp-image-2625" width="538" height="278" /></figure>
<!-- /wp:image -->

<!-- wp:image {"id":2623} -->
<figure class="wp-block-image"><img src="https://pvandenheede.files.wordpress.com/2019/05/apim-policy-api.png" alt="" class="wp-image-2623" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>I also setup a new Product called "genericProduct":</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":2624} -->
<figure class="wp-block-image"><img src="https://pvandenheede.files.wordpress.com/2019/05/apim-policy-product.png" alt="" class="wp-image-2624" /></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Now we just need to come up with some policies...<br>We know that policies often have a &lt;base /&gt; policy, so we need to take that into account as well.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I decided to setup the following policies:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Our All APIs policy:<br>Here we don't have a &lt;base /&gt; policy, so I just add a single new query parameter.</li><li>Our Test API 'All Operations' policy:<br>We add separate query parameters before and after the base policy.</li><li>Our Test API operation policy:<br> We add separate query parameters before and after the base policy. </li><li>Our Product policy<br>We add separate query parameters before and after the base policy. </li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>These are the policies in detail:</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>All APIS policy</h3>
<!-- /wp:heading -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">&lt;policies&gt;
    &lt;inbound&gt;
        &lt;set-query-parameter name="policy-allapis" exists-action="append"&gt;
            &lt;value&gt;allapis&lt;/value&gt;
        &lt;/set-query-parameter&gt;
    &lt;/inbound&gt;
    &lt;backend&gt;
        &lt;forward-request /&gt;
    &lt;/backend&gt;
    &lt;outbound /&gt;
    &lt;on-error /&gt;
&lt;/policies&gt; </pre>
<!-- /wp:preformatted -->

<!-- wp:heading {"level":3} -->
<h3>Test API - All Operations policy </h3>
<!-- /wp:heading -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">&lt;policies&gt;
    &lt;inbound&gt;
        &lt;set-query-parameter name="policy-allops-before" exists-action="append"&gt;
            &lt;value&gt;allops-before&lt;/value&gt;
        &lt;/set-query-parameter&gt;
        &lt;base /&gt;
        &lt;set-query-parameter name="policy-allops-after" exists-action="append"&gt;
            &lt;value&gt;allops-after&lt;/value&gt;
        &lt;/set-query-parameter&gt;
    &lt;/inbound&gt;
    &lt;backend&gt;
        &lt;base /&gt;
    &lt;/backend&gt;
    &lt;outbound&gt;
        &lt;base /&gt;
    &lt;/outbound&gt;
    &lt;on-error&gt;
        &lt;base /&gt;
    &lt;/on-error&gt;
&lt;/policies&gt; </pre>
<!-- /wp:preformatted -->

<!-- wp:heading {"level":3} -->
<h3>Test API - post operation policy </h3>
<!-- /wp:heading -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">&lt;policies&gt;
    &lt;inbound&gt;
        &lt;set-query-parameter name="policy-operation-before" exists-action="append"&gt;
            &lt;value&gt;operation-before&lt;/value&gt;
        &lt;/set-query-parameter&gt;
        &lt;base /&gt;
        &lt;set-query-parameter name="policy-operation-after" exists-action="append"&gt;
            &lt;value&gt;operation-after&lt;/value&gt;
        &lt;/set-query-parameter&gt;
    &lt;/inbound&gt;
    &lt;backend&gt;
        &lt;base /&gt;
    &lt;/backend&gt;
    &lt;outbound&gt;
        &lt;base /&gt;
    &lt;/outbound&gt;
    &lt;on-error&gt;
        &lt;base /&gt;
    &lt;/on-error&gt;
&lt;/policies&gt;  </pre>
<!-- /wp:preformatted -->

<!-- wp:heading {"level":3} -->
<h3>Product policy </h3>
<!-- /wp:heading -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">&lt;policies&gt;
    &lt;inbound&gt;
        &lt;set-query-parameter name="policy-product-before" exists-action="append"&gt;
            &lt;value&gt;product-before&lt;/value&gt;
        &lt;/set-query-parameter&gt;
        &lt;base /&gt;
        &lt;set-query-parameter name="policy-product-after" exists-action="append"&gt;
            &lt;value&gt;product-after&lt;/value&gt;
        &lt;/set-query-parameter&gt;
    &lt;/inbound&gt;
    &lt;backend&gt;
        &lt;base /&gt;
    &lt;/backend&gt;
    &lt;outbound&gt;
        &lt;base /&gt;
    &lt;/outbound&gt;
    &lt;on-error&gt;
        &lt;base /&gt;
    &lt;/on-error&gt;
&lt;/policies&gt; </pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->
<p>So what do we see now in RequestBin?</p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Conclusion</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>We consistently see the following appearing in our traces:</p>
<!-- /wp:paragraph -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">3

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175110"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before</a>"
4

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175248"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before</a>"
5

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175298"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before&amp;policy-product-before=product-before</a>"
6

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175350"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before&amp;policy-product-before=product-before&amp;policy-allapis=allapis</a>"
7

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175417"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before&amp;policy-product-before=product-before&amp;policy-allapis=allapis&amp;policy-product-after=product-after</a>"
8

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175458"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before&amp;policy-product-before=product-before&amp;policy-allapis=allapis&amp;policy-product-after=product-after&amp;policy-allops-after=allops-after</a>"
9

source
"set-query-parameter"
timestamp
"2019-05-23T09:25:03.0219982Z"
elapsed
"00:00:00.1175504"
data

message
"Request URL was updated with the query value."
request

url
"<a>http://requestbin.fullcontact.com/1bipq9x1/post?policy-operation-before=operation-before&amp;policy-allops-before=allops-before&amp;policy-product-before=product-before&amp;policy-allapis=allapis&amp;policy-product-after=product-after&amp;policy-allops-after=allops-after&amp;policy-operation-after=operation-after</a>" </pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->
<p>I tried several dozens of requests and different requestbins, the order always stays the same.<br><br>So, it seems we can <strong>conclude</strong> this is the order in which APIM executes policies:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li><strong>Operation </strong>policy <strong>before </strong>&lt;base /&gt;</li><li><strong>All Operations </strong>policy <strong>before </strong>&lt;base /&gt;</li><li><strong>Product </strong>policy <strong>before </strong>&lt;base /&gt;</li><li><strong>All APIs</strong> policy</li><li><strong>Product </strong>policy <strong>after </strong>&lt;base /&gt;</li><li> <strong>All Operations</strong> policy <strong>after </strong>&lt;base /&gt; </li><li> <strong>Operation </strong>policy <strong>after </strong>&lt;base /&gt; </li></ol>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>As usual, finding the solution proved to be much quicker than writing the blog post :-)</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Please place any feedback in the comments below. I hope you have a great day!<br>Cheers, Pieter</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><strong>Noteworthy: </strong>initially I was looking at RequestBin itself to see the order of the query parameters. However, this doesn't match with the query parameters I saw in the APIM traces. They are in a different order or even alphabetically. Don't depend on it too much!</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":2628} -->
<figure class="wp-block-image"><img src="https://pvandenheede.files.wordpress.com/2019/05/apim-policy-result.png" alt="" class="wp-image-2628" /></figure>
<!-- /wp:image -->
