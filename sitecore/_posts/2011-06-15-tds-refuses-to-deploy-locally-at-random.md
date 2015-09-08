---
layout: post
title: TDS refuses to deploy locally at random?
redirect_from: /blog/2011/06/tds-refuses-to-deploy-locally-at-random/
---

It turns out that the project build order can get jumbled somehow. Luckily the fix was quite simple:

1.  Right click on the the Solution or any Project in the Solution Explorer and choose "Build order..."
2.  Verify that your TDS project is at the bottom of the list.
3.  If not:
	-  Choose the Dependencies tab
	-  Select the TDS project in the projects drop down
	-  Tick the web project as a dependency
4.  If the TDS project is at the bottom of the list this fix probably will not help you.

That should do it. I was pointed in this direction by Hedgehog Development's excellent support staff -thanks guys!