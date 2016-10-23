+++
title = "Adding tracking code from Gauges App to Octopress"
date = "2012-08-13"
tags = ["web analytics", "octopress"]
type = "post"
+++

After relocation to a new blog engine and at the same time to a new domain I needed to connect to the blog a web analytics system from gaug.es. I wouldn't like to break Octopress file structure and I would like to use common ways to connection like Google Analytics, for example.

I read some articles from Octopress documentation as well as some blog posts and I decided to move simple way. Below I described my steps.

Google Analytics configuration file is located in `source/_includes/google_analytics.html`. One can use this file and just change javascript code in one. But I did in a different way. 

I created a new file and placed it to `source/_includes/custom/gauges_analytics.html`. I took embedded code from Gaug.es Dashboard and changed in it some lines. Below `gauges_analytics.html` content:  

	{% if site.gauges_analytics_tracking_id %}
		<script type="text/javascript">
		    var _gauges = _gauges || [];
		    (function() {
		      var t   = document.createElement('script');
		      t.type  = 'text/javascript';
		      t.async = true;
		      t.id    = 'gauges-tracker';
		      t.setAttribute('data-site-id', '{{ site.gauges_analytics_tracking_id }}');
		      t.src = '//secure.gaug.es/track.js';
		      var s = document.getElementsByTagName('script')[0];
		      s.parentNode.insertBefore(t, s);
		    })();
		</script>
	{% endif %}



How it's possible to see it's common javascript code but with two specific places. Here is locate Gaug.es app variables that define in `_config.yml` file. If this variable value is defined in `_config.yml`, Gaug.es is enabled. Take a look on `_config.yml`:

	# Google Analytics
	google_analytics_tracking_id:

I just add below some lines about Gauges App:

	# Gaug.es App Analytics
	gauges_analytics_tracking_id: 5028cb21613f5d50de00000a

After that it's necessary change yet another file to enable tracking. Open `source/_includes/head.html` and add 

	{% include custom/gauges_analytics.html %}

after

	{% include google_analytics.html %}

at the end of the file.

And result:

	{% include custom/head.html %}
	{% include google_analytics.html %}
	{% include custom/gauges_analytics.html %}

P.S.
Thanks Juev ([juev.ru](http://www.juev.ru/2012/07/05/jekyll--liquid-code-in-article/)) for the post about using Liquid code in articles.
