---
layout: page
title: Commenting
permalink: /commenting
---

**Thank you very much for submitting a comment!**

Since this blog is a personal one, there are no sophisticated spam filters and other checks integrated.
Nevertheless it is important to filter out spam and inappropriate content.
Therefore comments are checked manually and shown once they are approved.

In order to minimize the collection of personal data, there is no notification system in place.
So please have a look at the page from time to time if you expected an answer on your comment.

<ul class="pager blog-pager">
    <li class="previous" id="back-button" />
</ul>

<script>

window.addEventListener("load", function () {
    var urlParams = new URLSearchParams(window.location.search);
    if(urlParams.has('url')) {
        var backButtonContainer = document.getElementById('back-button');
        
        const backButtonLink = document.createElement('a');
        backButtonLink.setAttribute('href', urlParams.get('url'));
        backButtonLink.setAttribute('data-toggle', 'tooltip');
        backButtonLink.setAttribute('data-placement', 'top');
        
        var label = document.createTextNode('Back to post'); 
        backButtonLink.appendChild(label);
        
        backButtonContainer.appendChild(backButtonLink);
    }
});
</script>